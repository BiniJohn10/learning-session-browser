**Coercing mins**
Minutes are coerced in the `coerceMins` helper, used only at render time (`{{ coerceMins(s.mins) }}`). This keeps raw session data unchanged while ensuring the UI always displays a safe, finite number (fallback `0`) even if the source provides malformed values.

**Debounce implementation**
Debounce is implemented via a `watch(query)` with `setTimeout` + `clearTimeout`, writing to a separate `debouncedQuery`. This approach is lightweight, dependency-free, explicit, and works cleanly with Vue’s reactivity. It prevents excessive filtering while keeping input updates responsive.

**Stable order for equal popularity**
During load, each session receives an `_originalIndex`. The sort comparator first compares popularity; if equal, it falls back to `_originalIndex`. This enforces deterministic, stable sorting regardless of the JS engine’s sort stability.

**Accessibility for the completion toggle**
The toggle is a native `<button>` with built-in keyboard support. State is exposed via `aria-pressed` and the action via an `aria-label` (“Mark complete/incomplete”), ensuring screen readers understand it as a toggle button. Visible focus styles support keyboard users.

