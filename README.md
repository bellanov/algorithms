# Algorithm Visualization Project: Spec

## 1. Purpose
An interactive web app that teaches data structures and algorithms by animating each operation step by step, with synchronized pseudocode, complexity notes, and a "predict the next step" practice mode.

## 2. Goals and Non-Goals
**Goals**
- Show what changes in memory/structure at every step.
- Let learners step forward and backward, not just watch playback.
- Connect each visual step to a line of pseudocode and a plain-English explanation.
- Make input editable so learners can test their own cases.

**Non-goals (v1)**
- Production-grade performance on huge inputs (max ~200 elements).
- User accounts, cloud sync, or multiplayer.
- Compiling or running learner-written code.

## 3. Target Learner
Beginner to intermediate programmers learning DS&A, comfortable with one general-purpose language.

## 4. Tech Stack (suggested)
- **Language:** TypeScript
- **Build:** Vite
- **UI:** Vanilla TS or React (either is fine; keep rendering separate from logic)
- **Rendering:** SVG for trees/graphs, HTML/CSS for arrays and bars
- **Testing:** Vitest for algorithm logic; Playwright for a few end-to-end smoke tests

## 5. Architecture
Keep three layers separate:

1. **Algorithm layer (pure functions).** Each algorithm runs on plain data and emits a **trace**: an ordered array of step objects. It never touches the DOM.
2. **Playback engine.** Holds the trace and a cursor. Supports `play`, `pause`, `next`, `prev`, `seek(i)`, and `setSpeed`.
3. **View layer.** Reads the current step and renders it. Stateless per frame.

### Step object shape
```ts
interface Step {
  id: number;
  action: 'compare' | 'swap' | 'set' | 'visit' | 'highlight' | 'insert' | 'remove' | 'done';
  indices?: number[];           // array positions or node ids involved
  state: unknown;               // snapshot of the structure after this step
  pseudocodeLine: number;       // line to highlight in the code panel
  explanation: string;          // plain-English description
}
```
Storing a full `state` snapshot per step makes backward stepping trivial and avoids inverse-operation bugs. Cap the input size to keep memory small.

## 6. Features

### 6.1 Data structures (v1 order)
1. Array and dynamic array
2. Stack and queue
3. Singly linked list
4. Binary search tree (insert, delete, search, traversals)
5. Binary heap (insert, extract-min)
6. Hash table (chaining, with visible collisions)
7. Graph (adjacency list), with BFS and DFS

### 6.2 Algorithms (v1 order)
1. Sorting: bubble, insertion, selection, merge, quick
2. Searching: linear, binary
3. Graph: BFS, DFS, Dijkstra

### 6.3 Playback controls
- Play, pause, step forward, step back, reset
- Speed slider (0.25x to 4x)
- Scrubber that seeks to any step
- Keyboard shortcuts: Space (play/pause), ←/→ (step)

### 6.4 Learning aids
- **Pseudocode panel** with the active line highlighted.
- **Explanation panel** showing the current step's text.
- **Complexity panel** showing best, average, and worst-case Big-O, plus a count of comparisons and swaps so far.
- **Predict mode:** pause before a chosen step type and ask "what happens next?" Show the answer after the learner picks.
- **Custom input:** edit the array or graph by typing values; invalid input shows an inline error.

## 7. UI Layout
```
+---------------------------------------------------------+
| [Structure picker]  [Algorithm picker]  [Input editor]  |
+--------------------------------+------------------------+
|                                |  Pseudocode            |
|   Visualization canvas         |  (active line marked)  |
|                                +------------------------+
|                                |  Explanation           |
|                                |  Stats / Complexity    |
+--------------------------------+------------------------+
| [<<] [<] [Play/Pause] [>] [>>]  Speed  ====scrubber==== |
+---------------------------------------------------------+
```

## 8. Content Format
Each algorithm or structure is a self-contained module:
```
src/algorithms/sorting/quickSort.ts     // emits trace
src/algorithms/sorting/quickSort.meta.ts // name, pseudocode[], complexity, intro text
```
Adding a new algorithm should require only these files plus a registry entry.

## 9. Milestones
| # | Deliverable | Done when |
|---|---|---|
| M1 | Playback engine + array view + bubble sort | Step forward/back works; trace is correct on 10 random inputs |
| M2 | Remaining sorts + searches | All sorts pass a property test (output is sorted, is a permutation of input) |
| M3 | Linked list, stack, queue | Each op emits correct steps and state |
| M4 | BST and heap with SVG rendering | Insert/delete keep invariants; tests check invariants after every step |
| M5 | Graph, BFS/DFS, Dijkstra | Dijkstra matches a reference implementation on random graphs |
| M6 | Hash table + custom input editor | Collisions are visible; invalid input is rejected |
| M7 | Predict mode + complexity panel | Learner can answer and see feedback |
| M8 | Polish: keyboard shortcuts, responsive layout, README | Lighthouse accessibility score ≥ 90 |

## 10. Testing Strategy
- **Unit (Vitest):** Every algorithm's final output is correct. Each trace's last step is `done`. Each step's `state` is valid for its structure (e.g., BST ordering holds).
- **Property tests:** Random inputs for sorts, heap order, and BST invariants.
- **Component tests:** Playback controls change the cursor as expected.
- **E2E (Playwright):** Load the app, pick bubble sort, step through, and confirm the pseudocode line changes.

## 11. Accessibility
- All controls keyboard-reachable with visible focus.
- Color is never the only signal; use shape or text labels for highlighted elements.
- Explanation text is announced via an `aria-live` region.

## 12. Learning Path (suggested order for the learner)
1. Arrays → bubble sort → insertion sort
2. Stack → queue → linked list
3. Binary search → BST
4. Heap → quick sort → merge sort
5. Hash table
6. Graphs → BFS → DFS → Dijkstra

## 13. Open Questions
- Which language should pseudocode and code snippets use (Python, JavaScript, or both)?
- Should the app be fully static (deployable to GitHub Pages) or include a backend?
- Is a predict-mode quiz score worth persisting in local storage in v1?