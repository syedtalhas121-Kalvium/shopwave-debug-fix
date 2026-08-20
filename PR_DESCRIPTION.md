## Summary

This pull request fixes the three intentional React state-management bugs in the ShopWave dashboard and documents each root cause in `BUG_REPORT.md`.

## Fixes

| Bug | Location | Root cause | Fix |
| --- | --- | --- | --- |
| Infinite effect loop | `src/pages/ProductSearch.jsx` | The effect listed `results` while also calling `setResults`, creating a feedback loop of renders and requests. | Depend on `query` only. |
| Stale search results | `src/pages/ProductSearch.jsx` | Every keystroke started an uncancelled request, allowing an older variable-latency response to overwrite newer results. | Add a 300 ms debounce, clear the timer in cleanup, and ignore stale responses. |
| Status badge not updating | `src/pages/OrderManager.jsx` | The existing orders array and order object were mutated in place, so React received the same array reference. | Map to a new array and spread a new object for the changed order. |

## Verification

`npm run build` completed successfully. The repository contains no automated test files, so `npm test -- --watchAll=false` reports `No tests found` rather than executing a test suite.

Browser verification confirmed that rapidly replacing `head` with `headphones` leaves the latest matching product results visible and that changing `ORD-1001` from `delivered` to `shipped` updates both the coloured badge and controlled dropdown immediately after the API call completes.

The public deployment is available at:

<https://syedtalhas121-Kalvium.github.io/shopwave-debug-fix/>

The walkthrough video is available on Google Drive with anyone-with-the-link viewer access:

<https://drive.google.com/file/d/1Jrr7gsv1LWn48KmpUmv2_vFPzAMTELT3/view?usp=sharing>

## Notes

The infinite request loop was the most damaging symptom because it could freeze the browser while leaving no obvious application error. The stale-data bug was harder to spot because it depends on response timing and fast typing. The immutable update bug was especially subtle because the API succeeded and the in-memory object changed, but React did not receive a new state reference.
