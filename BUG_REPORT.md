# ShopWave React State Management Bug Report

This report records the three runtime bugs identified in the starter implementation before the code fixes were applied.

## Bug #1 — Infinite effect feedback loop

**File:** `src/pages/ProductSearch.jsx`  
**Line(s):** 34–51, specifically line 51 in the starter file

### Expected Behaviour

The search effect should run when the user changes `query`, issue one search for the current query, and update the displayed results once the response arrives. Updating `results` or `loading` inside the effect should not cause the same search effect to run again.

### Actual Behaviour

After a non-empty search is entered, the Network panel shows repeated identical search requests. Each response updates `results`, which causes the component to render again and retriggers the effect. The cycle continues, consuming CPU and making the page appear frozen.

### Root Cause

The effect reads `query`, but it also lists `results` in its dependency array even though the effect calls `setResults()` internally. React reruns an effect whenever one of its dependencies changes. Because each `setResults(data)` call produces a new results array, `results` changes, the effect runs again, and another request is started. This violates the **effect dependency contract**: state written by an effect should not be included as a dependency unless the effect intentionally reacts to that state.

### Fix Applied

The `results` dependency was removed, leaving only the external value that the effect uses to perform the search. The async work was also placed behind a debounce and given cleanup; those related changes are documented as Bug #2.

**Before:**

```jsx
}, [query, results]);
```

**After:**

```jsx
}, [query]);
```

## Bug #2 — Uncontrolled requests and stale search results

**File:** `src/pages/ProductSearch.jsx`  
**Line(s):** 34–46 and 53–57 in the starter file

### Expected Behaviour

Typing should not start a request for every intermediate keystroke. The page should wait briefly for typing to pause, then request the current query. If an older request resolves after a newer query has been entered, its response must be ignored so the displayed results always correspond to the latest query.

### Actual Behaviour

Every keystroke immediately starts a `searchProducts(query)` request, so typing quickly creates multiple in-flight requests. Because the mock API uses variable delays, a slower request for an earlier query can resolve after a faster request for a later query and overwrite the screen with stale results. The Network panel shows one request per keystroke rather than a controlled, debounced request stream.

### Root Cause

The effect starts asynchronous work without either a debounce timer or a cleanup function. React may run the effect again before the previous promise settles, but the old promise is still allowed to call `setResults()`. The component therefore has a race condition in which the last response to resolve wins, not the response for the newest query. This violates the **async effect cleanup rule**: asynchronous work started by an effect must be cancelled where possible, and late responses must be marked stale and discarded.

### Fix Applied

A 300 ms debounce timer was added so a request starts only after the user pauses. The effect now returns cleanup that clears the pending timer and marks the previous request stale. The promise checks that flag before updating state, so late responses from older queries are ignored.

**Before:**

```jsx
searchProducts(query).then((data) => {
  setResults(data);
  setLoading(false);
});
```

**After:**

```jsx
let isStale = false;
const debounceTimer = setTimeout(() => {
  searchProducts(query).then((data) => {
    if (isStale) return;
    setResults(data);
    setLoading(false);
  });
}, 300);

return () => {
  isStale = true;
  clearTimeout(debounceTimer);
};
```

## Bug #3 — Direct mutation prevents a re-render

**File:** `src/pages/OrderManager.jsx`  
**Line(s):** 52–59 in the starter file

### Expected Behaviour

After the API confirms a status change, the selected order's status badge and dropdown should immediately show the new status without a page reload.

### Actual Behaviour

The Network panel shows a successful `200` response, and the order object is changed in memory, but the coloured badge remains on the previous status. The controlled dropdown also remains visually stale because React does not detect a state reference change.

### Root Cause

`const updatedOrders = orders` copies only the reference to the existing array. The code then mutates an order object inside that same array and passes the unchanged array reference to `setOrders()`. React uses reference equality for state updates; because the next value is the same array object as the current state, React can skip the re-render. This violates the **immutable state update rule** for arrays and objects held in React state.

### Fix Applied

The array is rebuilt with `map()`, and only the matching order is replaced with a new object created using the spread operator. Unchanged orders retain their values, while both the array and changed object receive new references.

**Before:**

```jsx
const updatedOrders = orders;
const order = updatedOrders.find((o) => o.id === orderId);
if (order) {
  order.status = newStatus;
}
setOrders(updatedOrders);
```

**After:**

```jsx
const updatedOrders = orders.map((order) =>
  order.id === orderId ? { ...order, status: newStatus } : order
);
setOrders(updatedOrders);
```

## Verification and Deployment

The corrected application was verified in the browser: rapid Product Search input showed the latest query's results, and changing `ORD-1001` from `delivered` to `shipped` updated the badge immediately after the API response.

Public deployment: <https://syedtalhas121-Kalvium.github.io/shopwave-debug-fix/>

Verification video: <https://drive.google.com/file/d/1Jrr7gsv1LWn48KmpUmv2_vFPzAMTELT3/view?usp=sharing>

