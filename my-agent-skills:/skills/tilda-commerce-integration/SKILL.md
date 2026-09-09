---
name: tilda-commerce-integration
description: Audit, debug, and extend custom Tilda catalog, cart, pricing, inventory, form, and B2B checkout integrations, while preserving native Tilda behavior and clearly separating client-side state from server-side guarantees.
metadata:
  short-description: Harden Tilda commerce integrations
---

# Tilda Commerce Integration

Use this skill for the data and transaction layer of custom Tilda commerce code. It complements `tilda-ui-engineering`: that skill owns layout, responsive behavior, accessibility, and visual interaction; this skill owns catalog data, cart state, pricing, inventory, forms, and checkout boundaries.

## Scope and boundary

- Treat custom HTML/CSS/JavaScript as a client adapter over Tilda services unless a real server endpoint is present.
- Do not describe `localStorage`, `window.tcart`, hidden form fields, or `#order` values as secure backend state.
- If the user asks for a review or explanation, diagnose without mutating. If the user asks for a change, implement the smallest safe change and verify it.
- Preserve the user's IDs, copy, URLs, business rules, and Tilda insertion context unless changing them is required and in scope.

## Runtime contracts to identify first

Locate and classify every use of:

- `TildaCatalogSDK.Api.getProductsList`, `getProductsByUid`, `getProduct`, and `getProductTabs` — catalog reads and enrichment.
- `TildaCatalogSDK.Markup.markProductCard` and `data-catalog` attributes — the bridge that makes custom markup recognizable to Tilda Store Cart.
- `localStorage.tcart`, `window.tcart`, `tcart__addProduct`, `tcart__saveLocalObj`, `tcart__updateTotalProductsinCartObj`, `tcart__reDrawCartIcon`, and `tcart__openCart` — client-side cart state and internal Tilda runtime hooks.
- `#order:...`, `#opencart`, `t706`, `ST100`, cart selectors, record IDs, and form IDs — compatibility surfaces, not stable application APIs.
- `forms.tildacdn.com/procces/`, `t_forms__initBtnClick`, `tildaform:*`, `formservices[]`, and success callbacks — native Tilda form processing and routing.

Record which parts are public SDK behavior, which are internal Tilda implementation details, and which are custom code. Do not assume that a selector or global function is stable merely because it works in one published version.

### Public API versus internal hooks

- Keep public catalog reads (`TildaCatalogSDK.Api`) conceptually separate from internal runtime hooks such as `tcart__*`, CSS selectors, record IDs, and synthetic `#order` links.
- Treat internal hooks as compatibility adapters: isolate them behind a small namespaced wrapper, feature-detect each member, and provide a bounded fallback when one is unavailable.
- Do not use a successful internal hook call as evidence that the business operation was completed; verify the resulting Tilda state separately.

## Required workflow

1. Read the complete supplied block and determine whether it belongs in a Vibe Block, page HTML, global Head, catalog page, cart page, or popup.
2. Map the data flow from catalog/API → normalized product → UI → cart → checkout/form. Name the source of truth for each field: UID, edition, SKU, title, image, quantity, price, stock, and campaign.
3. Search for duplicate loaders, duplicate click handlers, hardcoded IDs, `setTimeout`-driven state changes, fallback URLs, and global document listeners.
4. Normalize before changing behavior. Use one stable cart-line key such as `uid + editionuid`; deduplicate catalog products; keep enrichment data separate from transaction data.
5. Fix the smallest root cause, then test the original flow and adjacent failure states.

## Safe mutation and verification

Apply this rule to every write to a Tilda-managed object, native cart state, page/block configuration, or form configuration:

1. Read the complete current object immediately before the write. Do not build a replacement from a partial list response or from only the field being changed.
2. Merge only the intended fields and explicitly preserve all required fields that the endpoint or runtime may reset, including identity, variants, active state, parts, delivery, metadata, and existing custom fields.
3. Perform the write once, inspect the response for errors, then read the same object again and verify both the requested change and the preserved invariants.
4. Do not continue a bulk operation from an unverified mutation. Record the target, changed fields, response, and re-read result; report the operation as unverified if the follow-up read is unavailable.

For a client cart, the complete object means the current `tcart` plus its product lines, variant identifiers, quantities, prices, delivery fields, and totals. For a Tilda API object, it means the full record assembled from every endpoint needed to avoid silent field resets.

## Data, money, and pricing invariants

- A cart line's stored price is the price that was put into that line. Product API data enriches the line; it must not silently overwrite a promotional or negotiated line price.
- Distinguish `displayPrice`, `cartPrice`, and `authoritativePrice`. A value in JavaScript, `#order`, `localStorage`, or a hidden field is user-editable.
- For a quote/request workflow, client-side promo prices may be displayed, but the UI must clearly state that a manager confirms the final price. For payment, guaranteed discounts, tax, or reservation, require server-side recalculation or real Tilda catalog prices; never rely on client state.
- Parse locale money consistently (`1 234,56`, `1234.56`) and calculate in minor units where possible. Never use a parser that removes a decimal comma and turns `1 234,56` into `123456`.
- Keep one campaign configuration. Do not duplicate a promo price independently in card HTML, modal HTML, cart code, and checkout code.

## Catalog loading and filtering

- Load every `nextslice` page when a section can exceed the SDK page size. Protect against repeated slices and cycles.
- Bound concurrency, handle partial section failures, and report whether the catalog is complete or degraded. Prefer a shared cached Promise so catalog, modal, and cart code do not repeat the same bulk requests.
- Deduplicate by the actual product/edition identity. Do not assume that a SKU is globally unique when variants exist.
- Keep brand and stock inference deterministic. Preserve an explicit `unknown` stock state; do not treat missing quantity as either definitely available or definitely unavailable without a business rule.
- Client-side filters and pagination are presentation behavior. They do not restrict what a user can request or prove that inventory is reserved.

## Cart integration

- Prefer one namespaced, idempotent cart adapter for read, write, add, quantity updates, removal, redraw, and change notification. Avoid multiple capture-phase bridges that can add the same item twice.
- If a fallback `#order` link is required, parse and validate it once and use the same normalized line object as the SDK path. Check decimal preservation, image, SKU, UID, edition, quantity, and inventory.
- Keep cart synchronization explicit. Handle first load, same-tab changes, cross-tab `storage` changes, native Tilda cart changes, and newly added lines—not only quantity changes to lines already rendered.
- Treat `tcart` as browser-local and mutable. It is not a shared customer account, database, reservation, or audit log.
- When injecting order details into a native form, include a stable human-readable SKU/quantity summary and, where useful, a structured snapshot. Label it as informational unless it is verified server-side.

## Checkout and form integration

- Prefer native Tilda submission when the requested result is a Tilda email/CRM lead. Preserve native validation, service routing, success events, and callbacks.
- Do not depend on blind timing chains such as “click, wait 100 ms, click again”. Use feature detection, bounded element/state waits, and verify each transition; provide a graceful fallback when the native cart or form is absent.
- Centralize and verify the mapping between CTA hooks, popup hooks, record IDs, form IDs, `formservices[]`, form names, and subject templates. Flag copied site names or service IDs from another domain.
- Scope global listeners to exact hooks and make them idempotent. Avoid intercepting unrelated forms, links, cart items, or future Tilda blocks.
- Moving a popup record to `body` can solve stacking and clipping problems, but it is a permanent DOM mutation. Guard it against repeated initialization and verify close, Escape, overlay, success, focus restoration, and body-scroll cleanup.

## Security and trust boundary

- Assume catalog descriptions, tabs, URLs, prices, quantities, and hidden fields are untrusted at the transaction boundary. Escape attributes and sanitize HTML fields before inserting them with `innerHTML`; admin-only content is a trust assumption, not a security control.
- Never put secrets, private API keys, or authorization decisions in page JavaScript. Public service identifiers are not authentication.
- Do not claim that a disabled button, stock label, client calculation, or manager-facing hidden field enforces a business rule.

## Compatibility and verification

- Load the SDK only as needed and wait for both the SDK and its required API/Markup members. Handle script failure and incomplete initialization.
- Namespace globals, configuration, event names, and initialization flags. Make repeated Tilda preview/rerender initialization harmless.
- Test at minimum: slow/missing SDK, empty catalog, partial API failure, section over the page limit, duplicate SKU/variants, unknown stock, decimal prices, promo add → cart display, quantity/remove, cross-tab change, duplicate clicks, desktop/mobile checkout, popup native/manual open, form validation, success, Escape, and reduced motion.
- Static inspection is not live verification. If no URL or browser is available, say so and list the exact manual checks still required.

## Common high-risk patterns to check

- The cart recalculates from current catalog price and ignores the price stored in the cart line.
- One catalog has `nextslice` pagination while another silently stops at 500 items.
- Both a per-button handler and a document-level bridge call `tcart__addProduct`.
- Missing inventory is treated inconsistently between the regular catalog and a promotion page.
- A promo page uses one popup hook while a global popup script intercepts another.
- A copied `formname`, `formservices[]`, record ID, or domain remains from another site.
- Fallback URLs round decimal prices or lose image/SKU/variant metadata.

## Report format

Lead with the outcome. Then state:

1. the authoritative source for catalog, cart, price, stock, and form delivery;
2. the end-to-end data flow and any compatibility surfaces;
3. critical correctness/security risks, separated from cosmetic issues;
4. the minimal change or implementation plan;
5. behavior intentionally preserved;
6. checks actually performed and remaining live/manual checks.
