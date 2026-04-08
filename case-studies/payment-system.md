# Case Study: Payment System Design (Stripe-Style)

## What Makes This Problem Hard

A payment system looks like "an API over a database" — until you stare at the failure modes. Then it becomes one of the hardest distributed systems problems you can be asked in an interview:

- **Money cannot be lost** under any failure condition. Not under crashes, not under retries, not under partial writes.
- **External systems are unreliable.** Banks, card networks, and PSPs (Payment Service Providers) time out, return ambiguous responses, and send delayed webhooks.
- **Duplicate charges are catastrophic.** A double-charge isn't a bug — it's a legal problem, a chargeback, and a trust failure.
- **The flow is inherently async.** The bank responds when it responds. Sometimes 200ms, sometimes 30 seconds, sometimes never.
- **Consistency requirements are stricter than almost any other system.** Financial integrity must hold even when half your services are on fire.

**One-line framing:** *This is not just an API-over-a-database problem. It's a distributed systems problem where every assumption you'd normally make about consistency, retries, and failure handling has to be re-examined under the lens of "what if money is involved."*

---

## Functional Requirements

### Above the Line (Core Scope)

1. **Merchants initiate payment requests** — a merchant backend creates a charge for a customer.
2. **Users pay with credit/debit cards** — the customer submits card details and the system charges them.
3. **Merchants view payment status updates** — pending, succeeded, or failed, delivered via webhooks.

### Below the Line (Out of Scope)

- Saved payment methods / customer vault
- Refunds and partial refunds
- Transaction history and reporting dashboards
- Alternative payment methods (ACH, SEPA, wallets)
- Recurring payments / subscriptions
- Merchant payouts and balance management

---

## Non-Functional Requirements

**Scale target:** 10,000 TPS at peak load.

State these proactively before the interviewer asks — they signal staff+ thinking:

| NFR | Why It Matters |
|-----|----------------|
| **Exactly-once processing** | A duplicate charge is not "just a bug" — it's a chargeback, a customer support escalation, and potentially a legal liability. |
| **Durability** | Once a payment is acknowledged, no failure (process crash, DB failover, AZ outage) may lose it. The transaction record must survive. |
| **Financial integrity with async networks** | The bank/PSP runs on its own clock. Your system must reconcile eventually-consistent external state without losing money or double-counting. |
| **Security (defense in depth)** | Card data is one of the most valuable PII categories. Tokenization alone is not enough; every layer must assume the previous one can fail. |

---

## Core Entities

- **Merchant** — the business accepting payments.
- **Customer** — the end user paying.
- **PaymentIntent** — the merchant's *intent* to charge a customer for a specific amount. Lives independent of execution.
- **Payment** — a single *attempt* to execute a PaymentIntent. Has a lifecycle status (PENDING → PROCESSING → SUCCESS/FAILED).
- **PaymentMethod** — a tokenized reference to a card. Never the raw card data.
- **LedgerEntry** — an immutable, append-only financial record. Double-entry bookkeeping.

### Why PaymentIntent and Payment Are Separate

This is how Stripe actually models it, and it matters:

- A single PaymentIntent can have **multiple Payment attempts**. If the first attempt fails (insufficient funds, network timeout), a retry is a new Payment record under the same intent.
- The PaymentIntent represents the merchant's business goal ("charge $100 for order #123"). The Payment represents the technical execution against the PSP.
- This separation gives you a clean retry boundary, clean idempotency boundary, and clean audit trail.

---

## API Design

```
POST   /v1/payment-intents            # Create intent
GET    /v1/payment-intents/:id        # Read status
POST   /v1/payment-intents/:id/confirm # Execute (creates Payment, calls PSP)
POST   {merchant_webhook_url}         # Outbound — status updates to merchant
```

**Security rule, stated upfront:** Raw card data never touches your servers. The client submits card details directly to a PCI-compliant tokenization vault (Stripe.js style). Your API only ever receives a token. This keeps your services entirely outside PCI DSS scope.

---

## The Payment Flow (How It Actually Works)

The full async flow, step by step:

```
1.  Customer enters card details in browser
2.  Browser submits card details DIRECTLY to Tokenization Vault
        (skipping your servers entirely — PCI scope isolation)
3.  Vault returns payment_method_token
4.  Merchant backend calls POST /payment-intents/:id/confirm with token
5.  Payment Service creates Payment record (status = PENDING)
6.  Payment Service calls PSP / Acquirer with the token
7.  PSP submits to Card Network (Visa / Mastercard)
8.  Card Network contacts Issuing Bank
9.  Bank returns approved / declined (ms to seconds — sometimes longer)
10. PSP sends webhook back to Payment Service
11. Payment Service updates Payment status (PROCESSING → SUCCESS/FAILED)
12. Payment Service sends webhook to Merchant
```

> **Key insight:** Steps 6–10 are **outside your control**. The bank can be slow, the PSP can time out, the webhook can fail to deliver, and any of those can happen *after* you've already retried. Your system must handle all of these gracefully — and the rest of this document is largely about how.

---

## High-Level Architecture

```
   Customer Browser/App
         │
         │ (card details, direct)
         ▼
   ┌──────────────────────┐
   │  Tokenization Vault  │   ← PCI scope lives here
   │  (Stripe.js etc.)    │
   └──────────┬───────────┘
              │ token only
              ▼
       Merchant Backend
              │
              ▼
   ┌──────────────────────┐
   │     API Gateway      │   TLS, auth, rate limiting
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐      ┌─────────────────────────┐
   │   Payment Service    │─────▶│  PostgreSQL              │
   │                      │      │  (payment_intents,       │
   │                      │      │   payments)              │
   │                      │      └─────────────────────────┘
   │                      │      ┌─────────────────────────┐
   │                      │─────▶│  Ledger DB               │
   │                      │      │  (append-only, immutable)│
   └──────────┬───────────┘      └─────────────────────────┘
              │
              ▼
   ┌──────────────────────┐
   │   PSP Adapter Layer  │
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │  External PSP        │   Stripe / Adyen / Braintree
   │  (Stripe / Adyen)    │
   └──────────┬───────────┘
              │
              ▼
   Card Networks → Issuing Banks
              │
              │ (webhook back, async)
              ▼
   ┌──────────────────────┐
   │  Webhook Handler     │
   │     Service          │
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │  Kafka               │   payment.events topic
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │  Payment Service     │   updates status
   └──────────┬───────────┘
              │
              ▼
   ┌──────────────────────┐
   │ Merchant Webhook     │
   │ Dispatcher           │
   └──────────────────────┘
```

---

# The Four Hard Problems

## Hard Problem 1 — Exactly-Once Processing (Idempotency)

**The problem:** Network timeout after PSP submission. You retry. The PSP processes the request twice. The customer is charged twice.

This is the canonical distributed systems failure mode — and in payments, it's catastrophic.

**The solution:** Idempotency keys at every layer of the system.

### Code Example 1 — PSP submission with idempotency key (Python)

```python
def submit_to_psp(payment: Payment) -> PSPResponse:
    # Deterministic key — same payment_id always produces the same key
    idempotency_key = f"payment-{payment.id}"

    return psp_client.charges.create(
        amount=payment.amount_cents,
        currency=payment.currency,
        source=payment.payment_method_token,
        idempotency_key=idempotency_key,  # PSP deduplicates on this
    )
```

The PSP (Stripe, Adyen) guarantees that the same idempotency key returns the **same result** for any duplicate request, even days later. Retrying is safe by construction.

### Code Example 2 — API-level idempotency

```python
def create_payment_intent(merchant_id: str, body: dict, idempotency_key: str):
    existing = db.payment_intents.find_one({
        "merchant_id": merchant_id,
        "idempotency_key": idempotency_key,
    })
    if existing:
        return existing  # Return prior result, do nothing else

    intent = PaymentIntent(
        merchant_id=merchant_id,
        idempotency_key=idempotency_key,
        amount=body["amount"],
        currency=body["currency"],
        status="REQUIRES_CONFIRMATION",
    )
    db.payment_intents.insert(intent)
    return intent
```

The unique constraint on `(merchant_id, idempotency_key)` is the database-level guarantee. Two concurrent inserts with the same key — one succeeds, one fails with a constraint violation, and you fall back to the read path.

### Code Example 3 — Redis idempotency store (fast path)

```python
def check_and_set_idempotency(key: str, ttl_seconds: int = 86400):
    """Returns (is_duplicate, prior_result)."""
    sentinel = "PROCESSING"
    # Atomic SET-if-not-exists
    was_set = redis.set(key, sentinel, nx=True, ex=ttl_seconds)
    if was_set:
        return (False, None)  # First time — caller proceeds

    prior = redis.get(key)
    if prior == sentinel:
        raise IdempotencyInProgressError("Concurrent request still processing")
    return (True, json.loads(prior))

def store_idempotency_result(key: str, result: dict, ttl_seconds: int = 86400):
    redis.set(key, json.dumps(result), ex=ttl_seconds)
```

This is the fast pre-check before hitting the DB. The DB constraint is the source of truth; Redis is the latency optimization.

---

## Hard Problem 2 — Durability and Auditability (The Ledger)

**Core principle:** Double-entry bookkeeping. Every transaction has two sides. Money never appears or disappears — it moves between accounts.

### Example

```
Customer pays $100, with a 3% platform fee:

  DEBIT   customer_account     $100
  CREDIT  merchant_account     $97
  CREDIT  platform_revenue     $3

  Sum of debits   = Sum of credits   ✓
```

If your ledger entries don't balance, you have a bug. This invariant is checkable in production at any moment — and that's the entire point.

### Ledger Design — Append-Only and Immutable (Go)

```go
type EntryType string

const (
    Debit  EntryType = "DEBIT"
    Credit EntryType = "CREDIT"
)

type LedgerEntry struct {
    ID             string    // UUID
    PaymentID      string    // Foreign key to Payment
    AccountID      string    // Which account is moving
    EntryType      EntryType // DEBIT or CREDIT
    Amount         int64     // ALWAYS in cents — no floats, ever
    Currency       string    // ISO 4217
    CreatedAt      time.Time
    IdempotencyKey string    // Prevents duplicate ledger writes
}

func RecordPayment(ctx context.Context, db *sql.DB, p Payment) error {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer tx.Rollback()

    debit := LedgerEntry{
        ID:             uuid.NewString(),
        PaymentID:      p.ID,
        AccountID:      p.CustomerAccountID,
        EntryType:      Debit,
        Amount:         p.AmountCents,
        Currency:       p.Currency,
        CreatedAt:      time.Now().UTC(),
        IdempotencyKey: fmt.Sprintf("ledger-%s-debit", p.ID),
    }
    credit := LedgerEntry{
        ID:             uuid.NewString(),
        PaymentID:      p.ID,
        AccountID:      p.MerchantAccountID,
        EntryType:      Credit,
        Amount:         p.AmountCents - p.FeeCents,
        Currency:       p.Currency,
        CreatedAt:      time.Now().UTC(),
        IdempotencyKey: fmt.Sprintf("ledger-%s-credit", p.ID),
    }

    if err := insertEntry(ctx, tx, debit); err != nil {
        return err
    }
    if err := insertEntry(ctx, tx, credit); err != nil {
        return err
    }
    return tx.Commit()
}
```

Both entries commit atomically in a single DB transaction. Either both land or neither does. **Never UPDATE. Never DELETE.** Corrections are *new* compensating entries.

### Why Append-Only Matters

- Every historical state of the system is recoverable — by replaying entries up to any point in time.
- The audit trail is complete *by definition*, not as an afterthought feature.
- Bugs are fixed by adding **correcting entries**, not by mutating history. The wrong entry stays, and a reversal entry sits next to it. Both are visible.
- This is **event sourcing applied to finance** — the ledger is the source of truth, and balances are projections.

### Account Balance as a Derived Value

Never store balance as a mutable column. The ledger is the source of truth.

```sql
SELECT SUM(
    CASE WHEN entry_type = 'CREDIT' THEN amount
         ELSE -amount
    END
) AS balance
FROM ledger_entries
WHERE account_id = $1
  AND currency   = $2;
```

- No hot row contention under burst (no row to lock).
- Source of truth is always the ledger.
- For dashboard reads, use a **materialized view** that refreshes periodically. See [../databases/materialized-views.md](../databases/materialized-views.md).

---

## Hard Problem 3 — Financial Integrity with Async Networks

### The Race Condition

```
T = 0s    Payment Service submits charge to PSP
T = 5s    PSP times out — no response received
T = 30s   Retry submitted (different request, same idempotency key)
T = 10m   Original webhook arrives: SUCCESS
T = 10m+ε Retry's webhook arrives: SUCCESS (or worse, DUPLICATE)

Question: did we double-charge?
```

The answer must be **no, never, under any timing.**

### Three-Layer Solution

#### Layer 1 — Idempotency keys

Already covered in Hard Problem 1. The PSP guarantees that the second submission with the same key returns the *same* result as the first. So even if both webhooks arrive, they describe the *same* charge.

#### Layer 2 — Payment State Machine with Valid Transitions (Python)

```python
VALID_TRANSITIONS = {
    "PENDING":    {"PROCESSING", "FAILED", "CANCELLED"},
    "PROCESSING": {"SUCCESS", "FAILED"},
    "SUCCESS":    set(),    # Terminal
    "FAILED":     {"PENDING"},  # Retry creates a new attempt
    "CANCELLED":  set(),    # Terminal
}

def transition_payment(payment_id: str, new_status: str):
    with db.transaction():
        # Pessimistic lock — no two workers can transition the same row
        payment = db.execute(
            "SELECT * FROM payments WHERE id = %s FOR UPDATE",
            (payment_id,),
        ).fetchone()

        if new_status not in VALID_TRANSITIONS[payment.status]:
            raise InvalidTransitionError(
                f"{payment.status} → {new_status} not allowed"
            )

        db.execute(
            "UPDATE payments SET status = %s, updated_at = NOW() WHERE id = %s",
            (new_status, payment_id),
        )
```

Two webhooks arriving for the same payment can both try to transition `PROCESSING → SUCCESS`. The `FOR UPDATE` lock serializes them. The second one sees `status = SUCCESS`, finds `SUCCESS → SUCCESS` is invalid, and aborts safely.

#### Layer 3 — Reconciliation Job (runs hourly)

This is the ultimate safety net — comparing your internal state against the PSP's truth.

```python
def reconcile_window(start: datetime, end: datetime):
    psp_records = psp_client.list_charges(created_gte=start, created_lt=end)
    psp_by_id = {r.idempotency_key: r for r in psp_records}

    db_payments = db.payments.find({"created_at": {"$gte": start, "$lt": end}})
    db_by_key = {p.idempotency_key: p for p in db_payments}

    # Case 1: PSP has it, we don't → missed webhook → catch up now
    for key, psp_rec in psp_by_id.items():
        if key not in db_by_key:
            recover_missed_payment(psp_rec)

    # Case 2: We have SUCCESS, PSP doesn't → flag for manual review
    for key, db_pay in db_by_key.items():
        if db_pay.status == "SUCCESS" and key not in psp_by_id:
            alert_finance("ghost_success", db_pay.id)

    # Case 3: Amount mismatch → page finance team immediately
    for key, db_pay in db_by_key.items():
        if key in psp_by_id and db_pay.amount != psp_by_id[key].amount:
            page_finance("amount_mismatch", db_pay.id, psp_by_id[key].amount)
```

> **This is what staff+ means by "resilient reconciliation processes that guarantee eventual consistency with external payment networks."** You're not just retrying on failure — you're closing the loop with the external system's source of truth on a regular cadence.

---

## Hard Problem 4 — Security (Defense in Depth)

Each layer assumes the previous one can fail.

### Layer 1 — Client-Side Tokenization

```
   Browser ──card number──▶ Tokenization Vault
   Browser ◀────token──── Tokenization Vault
   Browser ────token────▶ Your API
```

Your servers never see the card number. Your services run **outside PCI DSS scope** entirely. This single decision saves enormous compliance burden.

### Layer 2 — Encryption at Rest (Envelope Encryption)

```python
def encrypt_metadata(plaintext: bytes, kms) -> tuple[bytes, bytes]:
    # Generate a fresh Data Encryption Key per record
    dek = os.urandom(32)

    # Encrypt the data with the DEK
    nonce = os.urandom(12)
    ciphertext = AESGCM(dek).encrypt(nonce, plaintext, None)

    # Encrypt the DEK with the KEK in KMS — KEK never leaves KMS
    encrypted_dek = kms.encrypt(KeyId="alias/payments-kek", Plaintext=dek)["CiphertextBlob"]

    return (nonce + ciphertext, encrypted_dek)
```

Store `encrypted_metadata` and `encrypted_dek` together in the DB. The KEK lives in AWS KMS / HashiCorp Vault and never appears in any database. Stealing a DB dump is useless without KMS access.

### Layer 3 — TLS Everywhere + PCI Scope Minimization

- TLS 1.2+ on every internal hop, mTLS where possible.
- Raw card data lives in a **separate PCI-scoped environment** (the vault provider). Your main services have no path to it.
- Network segmentation: PCI-scoped services live in their own VPC/subnet with no egress to your main app subnets.

### Layer 4 — Fraud Detection via Velocity Checks (Python + Redis)

```python
def velocity_check(customer_id: str, amount_cents: int) -> str:
    hour = int(time.time() // 3600)
    key = f"velocity:{customer_id}:{hour}"

    pipe = redis.pipeline()
    pipe.incr(key)
    pipe.expire(key, 3600)
    attempts, _ = pipe.execute()

    if attempts > 10:
        return "BLOCK"
    if amount_cents > 10_000_00:  # $10k
        return "MANUAL_REVIEW"
    return "ALLOW"
```

Cheap, in-line, and catches the most obvious abuse patterns before they hit the PSP.

---

# DLQ Design

Three failure points need DLQ coverage:

1. **PSP submission failure** — your service can't reach the PSP, or the PSP errors out.
2. **Internal webhook processing failure** — the inbound webhook from the PSP can't be applied to your DB.
3. **Merchant webhook delivery failure** — your outbound webhook to the merchant fails.

### Scenario 1 — PSP Submission Failure (Python)

```python
def submit_with_dlq(payment: Payment):
    backoff = [1, 2, 4]  # seconds
    for attempt, delay in enumerate(backoff, start=1):
        try:
            return submit_to_psp(payment)
        except PSPNetworkOutageError:
            # Skip retries — go straight to DLQ
            break
        except PSPTransientError:
            time.sleep(delay)

    kafka.produce("payment.submission.dlq", {
        "payment_id": payment.id,
        "failed_at": datetime.utcnow().isoformat(),
        "reason": "psp_unreachable",
        "attempts": attempt,
    })
    transition_payment(payment.id, "PENDING_RECOVERY")
```

### Scenario 2 — Webhook Processing Failure (Python)

```python
def process_psp_webhook(event: dict):
    for attempt in range(3):
        try:
            apply_webhook_to_db(event)
            return
        except DatabaseUnavailableError:
            time.sleep(2 ** attempt)

    kafka.produce("payment.webhook.dlq", {
        "event": event,
        "failed_at": datetime.utcnow().isoformat(),
        "reason": "db_unavailable",
    })
```

### Scenario 3 — Merchant Webhook Delivery Failure (Python)

```python
def dispatch_merchant_webhook(merchant: Merchant, payload: dict):
    body = json.dumps(payload)
    signature = hmac.new(merchant.webhook_secret, body.encode(), "sha256").hexdigest()

    backoff = [1, 2, 4, 8, 16]
    for attempt, delay in enumerate(backoff, start=1):
        try:
            r = requests.post(
                merchant.webhook_url,
                data=body,
                headers={
                    "Content-Type": "application/json",
                    "X-Attempt": str(attempt),
                    "X-Signature": signature,
                },
                timeout=10,
            )
            if r.status_code < 500:
                return
        except requests.RequestException:
            pass
        time.sleep(delay)

    kafka.produce("merchant.webhook.dlq", {
        "merchant_id": merchant.id,
        "payload": payload,
        "failed_at": datetime.utcnow().isoformat(),
    })
```

### DLQ Processor Design (Go)

A single worker drains all DLQ topics with age-based routing.

```go
func ProcessDLQ(ctx context.Context, msg DLQMessage) error {
    age := time.Since(msg.FailedAt)

    switch {
    case age < 1*time.Hour:
        // Likely transient — retry immediately
        return retryOriginalOperation(msg)

    case age < 24*time.Hour:
        // Schedule a delayed retry in 30 minutes
        return scheduleDelayedRetry(msg, 30*time.Minute)

    default:
        // Something is genuinely wrong — escalate
        alertOperationsTeam(msg)
        return moveToManualReview(msg)
    }
}
```

### DLQ Architecture

```
   Payment Service ──fail──▶ payment.submission.dlq ──┐
                                                      │
   Webhook Handler ──fail──▶ payment.webhook.dlq ─────┼──▶ DLQ Processor
                                                      │       │
   Webhook Dispatcher ─fail─▶ merchant.webhook.dlq ──┘       │
                                                              ▼
                                                  retry / delayed retry
                                                  / manual review
```

### DLQ + Reconciliation — Complementary Layers

| | DLQ | Reconciliation |
|-|-----|----------------|
| **Granularity** | Individual failed event | All payments in a time window |
| **Latency** | Minutes | Hours |
| **Trigger** | A specific failure happened | Scheduled, regardless of failures |
| **Recovery** | Automatic retry | Automatic + manual review |
| **Catches** | Known transient failures | *Unknown* failures (silent drops, missed webhooks) |

DLQ catches the failures you saw. Reconciliation catches the failures you didn't.

### Idempotency Keys Make DLQ Safe

DLQ retries reuse the **original** idempotency key. The PSP deduplicates. No double charge.

> **Without idempotency keys, a DLQ retry is a loaded gun. With them, retries are safe by design.**

---

# High-Concurrency Burst Handling

### Where Bursts Occur (Not Just Ticketing)

| Scenario | Trigger | Constrained Resource |
|----------|---------|---------------------|
| Concert ticket sales | On-sale moment | Seats |
| Black Friday / 11.11 flash sale | Time-based promotion | Inventory + payment throughput |
| Uber post-event payment burst | Concert/game ends | Driver supply + payment processors |
| Stock market open | 9:30 AM ET | Order book matching |
| Food delivery lunch hour | 12:00 PM local | Restaurant capacity + payments |
| Sports betting pre-game | Match start | Bet placement throughput |
| Subscription renewal | Midnight batch | Payment processor rate limits |
| NFT / limited drops | Drop time | Mint slots + payment throughput |

**Pattern:** time-based trigger + constrained resource. Always.

### What Breaks Under Burst — Failure Sequence

100,000 simultaneous requests hitting the system in order of collapse:

1. **API Gateway** — rate limiter overwhelmed, latency spikes.
2. **Payment Service** — thread pool exhausted holding open PSP calls.
3. **Database** — connection pool saturated; new requests block.
4. **PSP** — external rate limit hit (Stripe default: 100 req/s).
5. **Ledger** — write contention on hot accounts (e.g., the `nike` merchant account).
6. **Kafka** — producer backpressure as consumers fall behind.

Each layer needs a different mitigation.

### Layer 1 — API Gateway: Rate Limiting + Virtual Waiting Room

**Token bucket per merchant (Python):**

```python
def allow_request(merchant_id: str, limit_per_sec: int) -> bool:
    second = int(time.time())
    key = f"rate:{merchant_id}:{second}"
    pipe = redis.pipeline()
    pipe.incr(key)
    pipe.expire(key, 2)
    count, _ = pipe.execute()
    return count <= limit_per_sec
```

**Virtual waiting room (Python):**

```python
CAPACITY_THRESHOLD = 50_000

def admit_or_queue(request_id: str) -> dict:
    in_flight = redis.scard("payment:active")
    if in_flight < CAPACITY_THRESHOLD:
        redis.sadd("payment:active", request_id)
        return {"status": "ADMITTED"}

    position = redis.rpush("payment:queue", request_id)
    eta_seconds = position * 0.5  # rough estimate
    return {"status": "QUEUED", "position": position, "eta": eta_seconds}
```

This is what Ticketmaster does. Excess load gets a "you're #4,231 in line, ETA 30 minutes" page instead of a 500.

### Layer 2 — Payment Service: Async via Kafka

**Before — synchronous:**

```python
def confirm_payment(intent_id: str, token: str):
    payment = create_payment(intent_id, token, status="PROCESSING")
    result = submit_to_psp(payment)        # Blocks 200–2000ms
    update_payment(payment.id, result)
    return payment   # Holds the connection the whole time
```

**After — async:**

```python
def confirm_payment(intent_id: str, token: str):
    payment = create_payment(intent_id, token, status="PENDING")
    kafka.produce("payment.submitted", {
        "payment_id": payment.id,
        "token": token,
    })
    return Response(payment, status=202)  # < 50ms, connection released
```

The user polls for status, or receives a webhook when the PSP responds. **The thundering herd becomes a controlled stream consumed at the rate the PSP can handle.**

### Layer 3 — Database: Connection Pooling + Sharding

**PgBouncer**

```
  100,000 app connections ──▶ PgBouncer ──▶ 100 real PostgreSQL connections
                              (transaction
                               pooling)
```

Without PgBouncer, 100k connections crash PostgreSQL. With it, requests queue at PgBouncer (cheap) instead.

**Sharding by merchant_id (Python):**

```python
NUM_SHARDS = 16

def get_shard(merchant_id: str):
    shard_index = hash(merchant_id) % NUM_SHARDS
    return shard_pool[shard_index]
```

No single DB receives all writes. Each shard handles a subset of merchants — natural horizontal scaling.

### Layer 4 — PSP Rate Limits (The Hardest Problem)

Stripe's default is 100 req/s. Your burst is 10,000 req/s. You will hit the limit *instantly*.

#### Strategy 1 — Multiple PSP Accounts (Python)

```python
PSP_ACCOUNTS = [
    {"name": "stripe-acct-1", "weight": 1},
    {"name": "stripe-acct-2", "weight": 1},
    {"name": "stripe-acct-3", "weight": 1},
]

def select_account(payment_id: str):
    # Consistent hashing — same payment always routes to same account
    # so retries land on the same idempotency-key namespace
    idx = hash(payment_id) % len(PSP_ACCOUNTS)
    return PSP_ACCOUNTS[idx]
```

3 accounts = 300 req/s effective ceiling, with consistent routing for retry safety.

#### Strategy 2 — Kafka Rate-Limited Consumer (Go)

```go
import "golang.org/x/time/rate"

func RunPSPSubmitter(ctx context.Context, consumer KafkaConsumer) {
    // 90% of PSP limit — leave headroom for retries and reconciliation
    limiter := rate.NewLimiter(rate.Limit(90), 90)

    for msg := range consumer.Messages() {
        if err := limiter.Wait(ctx); err != nil {
            return
        }
        go submitToPSP(msg)
    }
}
```

The consumer applies natural backpressure. Kafka holds the queue; the PSP is never overrun.

#### Strategy 3 — PSP Fallback Routing (Python)

```python
PRIMARY_PSP  = "stripe"
FALLBACK_PSP = "adyen"

def submit_with_fallback(payment: Payment):
    try:
        return psp(PRIMARY_PSP).charge(payment)
    except PSPRateLimitError:
        return psp(FALLBACK_PSP).charge(payment)
    except PSPOutageError:
        return psp(FALLBACK_PSP).charge(payment)
```

> **This is what staff+ means by "fallback processing paths."** Your dependency on a single PSP is a single point of failure. Multiple PSPs is not just for cost — it's for survivability.

### Layer 5 — Ledger Write Contention

**The problem:** A hot merchant account (`nike`) receives 10,000 simultaneous credits. Row-level locks serialize them, throughput collapses to ~100/s.

**Solution 1 — Batch ledger entries (Python):**

```python
def batch_writer():
    buffer = []
    deadline = time.time() + 0.1   # 100ms window
    while time.time() < deadline and len(buffer) < 1000:
        try:
            entry = entry_queue.get(timeout=deadline - time.time())
            buffer.append(entry)
        except Empty:
            break

    if buffer:
        db.execute_batch(
            "INSERT INTO ledger_entries (...) VALUES (...)",
            buffer,
        )
```

100 individual INSERTs become 1 bulk INSERT. Throughput goes up by ~50×.

**Solution 2 — Balance as a derived value (SQL):**

```sql
-- NEVER store balance as a mutable column with row-level locks
SELECT SUM(
    CASE WHEN entry_type = 'CREDIT' THEN amount
         ELSE -amount
    END
) AS balance
FROM ledger_entries
WHERE account_id = $1;

-- For dashboards, refresh a materialized view periodically
CREATE MATERIALIZED VIEW account_balances AS
SELECT account_id,
       currency,
       SUM(CASE WHEN entry_type = 'CREDIT' THEN amount ELSE -amount END) AS balance
FROM ledger_entries
GROUP BY account_id, currency;
```

No hot row, no contention. See [../databases/materialized-views.md](../databases/materialized-views.md).

### Full Architecture Under Burst

```
   100,000 requests
        │
        ▼
  ┌────────────────────┐
  │  API Gateway       │  rate limiter
  │  + Waiting Room    │ ──▶ excess users see queue
  └────────┬───────────┘
           │
           ▼
  ┌────────────────────┐
  │  Payment Service   │ ──▶ 202 Accepted (< 50ms)
  └────────┬───────────┘
           │
           ▼
  ┌────────────────────┐
  │  Kafka             │  payment.submitted
  └────────┬───────────┘
           │
           ▼
  ┌────────────────────┐
  │  PSP Submitter     │  rate-limited (90 req/s)
  └────────┬───────────┘
           │
   ┌───────┴────────┐
   ▼                ▼
 Primary         Fallback
 (Stripe)        (Adyen)
   │                │
   └───────┬────────┘
           ▼
  ┌────────────────────┐
  │  Kafka             │  payment.psp.response
  └────────┬───────────┘
           │
           ▼
  ┌────────────────────┐
  │  Webhook Handler   │ ──▶ DB (via PgBouncer)
  └────────┬───────────┘
           │
   ┌───────┴────────────┐
   ▼                    ▼
 Payment status     Ledger entries
 updated            (batch insert)
   │                    │
   └────────┬───────────┘
            ▼
  ┌────────────────────┐
  │  Kafka             │  payment.events
  └────────┬───────────┘
           │
           │ ──── failures at any step ──▶  DLQ
           │
           ▼
  ┌────────────────────┐
  │  Merchant Webhook  │
  │  Dispatcher        │
  └────────────────────┘

  Reconciliation Job (hourly) ──▶ catches anything that slipped through
```

### Burst vs Ticketmaster Comparison

| System | Constrained Resource | Mitigation Pattern |
|--------|---------------------|---------------------|
| Ticketmaster | Seats (inventory) | Waiting room + reservation locks |
| Payment system | PSP rate limit + DB connections | Async + rate-limited consumer + fallback PSP |
| Uber surge | Available drivers | Dynamic pricing + matching backoff |
| Stock trading | Order book matching | Sequencing + per-symbol shards |

The constrained resource differs. The pattern — *time-based trigger meets bottleneck* — is the same.

---

# Webhook System

### Polling vs Webhooks

| | Polling | Webhooks |
|-|---------|----------|
| Latency | seconds–minutes | ~immediate |
| Server cost | high (constant polling) | low (event-driven) |
| Reliability | merchant controls retry | sender must retry |
| Complexity | simple | requires signing, retries, DLQ |

For 10k TPS, polling is a non-starter. Webhooks with retries + DLQ + reconciliation is the only viable model.

### Dispatching a Merchant Webhook (Python)

```python
def dispatch_merchant_webhook(merchant: Merchant, payload: dict):
    body = json.dumps(payload)
    signature = hmac.new(
        merchant.webhook_secret.encode(),
        body.encode(),
        hashlib.sha256,
    ).hexdigest()

    backoff = [1, 2, 4, 8, 16]
    for attempt, delay in enumerate(backoff, start=1):
        try:
            r = requests.post(
                merchant.webhook_url,
                data=body,
                headers={
                    "Content-Type": "application/json",
                    "X-Attempt": str(attempt),
                    "X-Signature": f"sha256={signature}",
                },
                timeout=10,
            )
            if 200 <= r.status_code < 300:
                return
        except requests.RequestException:
            pass
        time.sleep(delay)

    kafka.produce("merchant.webhook.dlq", {
        "merchant_id": merchant.id,
        "payload": payload,
    })
```

### Verifying a Webhook on the Merchant Side (Go)

```go
func VerifyWebhook(body []byte, sigHeader, secret string) bool {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(body)
    expected := "sha256=" + hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(expected), []byte(sigHeader))
}
```

`hmac.Equal` is constant-time — never compare HMACs with `==`, that opens you to a timing attack.

---

# Scaling to 10,000 TPS

Three bottlenecks, three mitigations:

| Bottleneck | Mitigation |
|------------|------------|
| **DB writes** | PgBouncer + sharding by `merchant_id` |
| **PSP submission** | Kafka decoupling + rate-limited consumer + fallback PSP |
| **Ledger writes** | Append-only + batch inserts + balance as derived value |

Each of these is independently necessary. Removing any one and the system collapses under burst.

---

# Staff+ Framing

Two explicit signals from the problem spec that mark you as a staff+ candidate:

### 1. Event Sourcing Is the Right Model — Not Just Technically

> "Payment networks are inherently asynchronous. Your ledger should reflect that. Every state change is an event appended to a log, never a mutation of existing state. This aligns your data model with financial reality — banks themselves work this way, accountants work this way, and auditors expect this. The architecture isn't borrowing event sourcing from a textbook; it's recognizing that finance is *already* event-sourced, and the system that mirrors it has the lowest impedance with the real world."

### 2. Defense-in-Depth for Security

> "Tokenization alone isn't enough. Each layer assumes the previous one can fail. The browser might be compromised — so the vault is isolated. The vault might be breached — so the DB stores only tokens. The DB might be dumped — so encryption-at-rest with envelope encryption. The KEK might leak — so it lives in KMS, never on disk. Each layer is independently sufficient to protect against the failure of every layer above it."

---

## Cross-References

- See also: [../distributed-systems/idempotency.md](../distributed-systems/idempotency.md) — idempotency keys, deduplication patterns
- See also: [../databases/materialized-views.md](../databases/materialized-views.md) — balance as a derived value, dashboard reads
- See also: [../messaging/kafka.md](../messaging/kafka.md) — Kafka backpressure, DLQ topics
- See also: [../databases/indexes.md](../databases/indexes.md) — covering indexes on `(account_id, created_at)` for ledger queries

---

## Key Takeaways

- **Money cannot be lost.** Every design decision must be checked against this invariant. It outranks latency, throughput, and operational simplicity.
- **The flow is async.** Designing as if PSP responses are synchronous is the root cause of most payment system failures. Embrace Kafka, embrace pending states, embrace reconciliation.
- **Idempotency is the foundation.** Without it, retries are dangerous and DLQs are loaded guns. With it, every layer becomes safe to retry.
- **The ledger is event-sourced — by necessity, not by fashion.** Append-only, double-entry, balance-as-derived. This is how finance actually works.
- **Defense in depth.** Each security layer must hold even when the layers above it fail. Tokenization, envelope encryption, KMS, network isolation, velocity checks.
- **Burst handling is layered.** Waiting room → async Kafka → rate-limited consumer → fallback PSP → batched ledger writes. No single layer is sufficient.

---

## Interview Talking Points

1. **"Async + event sourcing is the right model for payments because the external world is async and event-sourced."** Banks, card networks, and PSPs operate on their own clock and emit events you must consume. Designing your internal model to mirror that — pending states, append-only ledger, status updates via webhook+reconciliation — gives you the lowest impedance with reality and the strongest auditability.

2. **"Idempotency keys are the foundation that makes safe retries and DLQs possible."** Every layer — API, internal service, PSP — must dedupe on a deterministic key. Without this, every retry is a potential double-charge. With it, retries are mechanically safe at any layer of the stack, including replay from DLQ days later.

3. **"DLQ and reconciliation are complementary safety nets, not redundant."** DLQ catches *known* failures (your code saw the error and routed the event). Reconciliation catches *unknown* failures (the event silently disappeared). You need both: the DLQ for fast automatic recovery, the reconciliation job for the failures you didn't know about.

4. **"Burst handling needs four cooperating layers: virtual waiting room, async Kafka decoupling, rate-limited PSP consumer, and fallback PSP."** Removing any one collapses the system. The waiting room shapes ingress, Kafka decouples the synchronous request from the slow downstream, the rate limiter respects PSP limits, and the fallback PSP keeps the system available when one provider degrades. This is what "staff+ resilience" looks like in practice.
