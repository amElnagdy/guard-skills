# Cohesion: Tell, Don't Ask, Demeter, Feature Envy

A business rule about some data belongs with the code that owns that data. When a caller reads another object's fields, decides something with them, and acts on the answer, the rule has leaked out of its owner. The next caller that needs the same rule writes it again, slightly differently.

## Contents

- Why agents need this check
- Vocabulary
- Check 1: decision made outside the owner (Tell, Don't Ask)
- Check 2: reaching through more than one level (Law of Demeter)
- Check 3: computing from another owner's data (Feature Envy)
- Not violations
- Severity

## Why agents need this check

An agent writes code where it is working: in the endpoint, the handler, or the service it was asked to change. Comparing a value against a field it can see in that spot is the shortest path to green, so the rule lands at the call site instead of in the owner. Repeat that across several tasks and the same rule exists in three places with three edge-case behaviors. That is knowledge duplication (imperative 11) that no text-similarity check catches, because each copy is phrased differently.

## Vocabulary

- **Owner**: the type or module that defines the data and should hold the rules about it. In OO code that is the class. In Go, Rust, or C it is the struct with its methods or associated functions. In functional code it is the module that defines and builds the type, including when the value is a map or dict.
- **Reading foreign state**: accessing another owner's data from outside it, by getter, property, public field, dict/map key, or destructuring a type defined elsewhere.
- **Message**: asking the owner for the answer instead, by calling a method on it or a function in its module.
- **Direct collaborator**: a field of the current unit, a parameter, or a value created in the current function.

## Check 1: decision made outside the owner (Tell, Don't Ask)

Flag a business decision (condition, comparison, validation, branch choice) built from foreign state when the owner could answer it.

```python
# Violation: the trial rule lives in the billing service
if subscription.plan.trial_days > 0 and subscription.started_at + timedelta(days=subscription.plan.trial_days) > now:
    return Invoice.empty()

# Fix: the subscription answers; the caller sends one message
if subscription.in_trial(now):
    return Invoice.empty()
```

The rule now has one home. Changing how trials work touches one method.

**Not a violation:** decisions on the unit's own state (`self`/`this`, or a type defined in the same module), on a primitive parameter, or on a value created in the same function.

## Check 2: reaching through more than one level (Law of Demeter)

Count each read of foreign state in a chain. The second read is made on what the first one returned. Syntax does not matter:

- `order.get_customer().get_address().get_country()`
- `order.customer.address.country`
- `order["customer"]["address"]["country"]`
- `order?.customer?.address?.country` (optional chaining does not reduce the count)
- nested destructuring: `const { customer: { address } } = order`

Storing the middle step in a local (`customer = order.customer` then `customer.address`) is still two levels.

One level is fine: read or message a direct collaborator. Reading one level and then **sending a message** to the result is also fine (`order.customer.can_ship_to(region)`), because a message is not a state read.

Severity depends on what the chain feeds:

- **Feeds a decision or calculation** → Important. Move the logic to the owner.
- **Only re-exposes data** (building a response DTO, a log line, an email body) → Nit at most. Suggest a delegating accessor (`order.shipping_country()`), but do not block. Output structures are allowed to pull data out of the domain.

## Check 3: computing from another owner's data (Feature Envy)

Flag a function whose logic is mostly reads of one other owner's fields, combined into a total, discount, eligibility, or similar derivation.

This includes **deriving from a collaborator's collection**: iterating a collection exposed by a parameter or field and projecting element fields into a new value, whatever the iteration form (loop, comprehension, `map`/`filter`/`reduce`, streams, LINQ, pipes).

```python
# Violation: the caller knows how a cart is totalled
total = sum(line.unit_price * line.quantity for line in cart.lines)

# Fix
total = cart.total()
```

```ts
// Violation
const skus = request.items.map((item) => item.sku);
// Fix
const skus = request.skus();
```

**Not a violation:** the iterated collection is a local variable (not just an alias for the collaborator's collection), the unit's own state, or a parameter that already *is* the collection (a `list[str]` of SKUs passed in).

## Not violations

Do not flag these. Most false positives come from here.

- **Fluent APIs, builders, and pipelines** where each step returns the same object or a wrapper: query builders, `StringBuilder`, logger builders, `Optional`/`Result`/`Maybe` chains, pipe operators between functions. Chaining is not navigation. *Exception:* a pipeline whose **source** is a collaborator's collection and whose body projects element fields is Check 3.
- **Passing an identifier to persistence or lookup**: `repo.get(order.customer_id)`.
- **Output structures reading the domain**: response DTOs, view models, serializers (Check 2's re-expose case).
- **Reading your own state**, including through your own accessors.
- **Plain data types** (DTOs, records, dataclasses without behavior) get no blanket exemption. A derivation over a DTO's own fields still belongs on the DTO. Reading from one to build output is the re-expose case.

When unsure whether code "leaks state and decides outside" or "just reads to re-expose", treat it as the re-expose case. The goal is to catch the clear cases, not to police every field read.

## Severity

- Check 1, Check 3, and Check 2 feeding logic: **Important**. These are design defects with maintenance cost, not correctness bugs.
- Check 2 for output only: **Nit**.
- Prefer the fix that moves behavior to the owner over one that only hides the chain behind another accessor. A delegating accessor is still better than the chain.

**Source.** Fowler, "TellDontAsk" bliki; Lieberherr, Holland & Riel, "Object-Oriented Programming: An Objective Sense of Style" (OOPSLA 1988), the Law of Demeter; Fowler, *Refactoring*, "Feature Envy" smell.
