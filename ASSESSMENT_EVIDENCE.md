# ShopWave Assessment Evidence

This file is the submission index for the ShopWave React state-management debugging assessment.

## Walkthrough video

[Watch the walkthrough video on Google Drive](https://drive.google.com/file/d/1Jrr7gsv1LWn48KmpUmv2_vFPzAMTELT3/view?usp=sharing)

The Drive URL is intended to be shared with **Anyone with the link as Viewer**. The walkthrough demonstrates the three bugs, the targeted fixes, and the browser verification:

1. The infinite `useEffect` request loop in Product Search and the corrected dependency array.
2. Debouncing and cleanup/stale-response protection for fast search input.
3. The immutable order-status update and the updated status badge in Order Manager.

## Live project

[Open the deployed ShopWave app](https://syedtalhas121-Kalvium.github.io/shopwave-debug-fix/)

## Pull request

[Open Pull Request #1](https://github.com/syedtalhas121-Kalvium/shopwave-debug-fix/pull/1)

## Commit evidence

The PR history now includes descriptive commit messages and this evidence index:

| Commit | Purpose |
| --- | --- |
| [`17c6c4b`](https://github.com/syedtalhas121-Kalvium/shopwave-debug-fix/commit/17c6c4b7be1235114dc5e904cc6f7da4c3c6d7ee) | Fix the three React state-management bugs. |
| [`e385850`](https://github.com/syedtalhas121-Kalvium/shopwave-debug-fix/commit/e38585091ca23264548bfa6cbceebbc01e4d793b) | Configure GitHub Pages deployment. |
| [`4c96a5d`](https://github.com/syedtalhas121-Kalvium/shopwave-debug-fix/commit/4c96a5df0746f163101c954a088cf2ffcf4ef17d) | Add the initial verification links and notes. |
| **latest evidence commit** | Add this assessment index and make the commit/evidence trail explicit. |

## Verification summary

- `npm run build` completes successfully.
- Product Search remains responsive during rapid input and displays results for the latest query.
- Order Manager updates the status badge and controlled dropdown after the API request completes.
- Root causes and minimal fixes are documented in [`BUG_REPORT.md`](./BUG_REPORT.md).
- Detailed browser checks are recorded in [`verification-notes.md`](./verification-notes.md).
