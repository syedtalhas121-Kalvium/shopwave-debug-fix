# Verification Notes

## Product Search

The local app loaded at `http://localhost:3000/products`. The search input accepted `head`, then was rapidly replaced with `headphones`. The page showed the latest query and a matching **Wireless Headphones Pro** result rather than stale results from the earlier query. The loading state appeared during the debounced request, and the page remained responsive.

The production build completed successfully with `npm run build`. The repository contains no automated test files; `npm test -- --watchAll=false` exited with the expected `No tests found` status.

## Order Manager

The local `/orders` page loaded with seven orders. The first order (`ORD-1001`) was changed from `delivered` to `shipped`; immediately after selection the UI showed the expected saving state while the API request was pending. A final post-request check is still required to confirm the badge settles on `shipped`.

The post-request browser check confirmed that `ORD-1001` now shows `shipped` in both the coloured status badge and the controlled dropdown, with no page reload. This verifies the immutable update fix in the running application.
