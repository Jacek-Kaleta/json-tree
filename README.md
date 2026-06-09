# Scalable Binary SQL Query Plan Tree (JsonTree)

Language: **English** | [Polish (Polski)](README.pl.md)

---

# Scalable Binary SQL Query Plan Tree (JsonTree)

The `JsonTree` component is a high-performance, object-oriented solution built entirely in pure JavaScript (Vanilla ES6+), HTML5, and CSS3. It is specifically designed to visualize hierarchical database execution plans (such as Oracle or PostgreSQL Explain Plans). The component automatically maps flat relational data structures based on an **Adjacency List** into an object-oriented hierarchical graph and renders it as an interactive tree.

This project has **zero external dependencies** or third-party libraries. The tree structure relies on native, semantic HTML5 elements, ensuring exceptional memory efficiency and rendering speed even when analyzing highly complex database operation trees with thousands of nodes.

---

## 🏗️ 1. Architecture and Logic Core: The `JsonTree` Class

The `JsonTree` class acts as a completely independent, isolated application core. All critical state variables, utility methods, and data transformation algorithms are implemented as private fields and methods (prefixed with `#`). This guarantees full encapsulation of the state and prevents memory leaks or pollution of the global scope.

### Structural Design and Lifecycle

1. **Constructor and Key Mapping:** Upon instantiation, the class accepts a custom relational key mapping configuration. This allows developers to flexibly define the names of the source fields responsible for the unique node identifier (`id`) and its parent identifier (`parentId`). By default, these fields map to `'ID'` and `'PARENT_ID'` respectively.
2. **Technical Validation and DFS Algorithm:** Before rendering begins, a technical validation routine is triggered within a private initialization method. It uses a **Depth-First Search (DFS)** algorithm to detect cyclical references (infinite loops) in the plan structure, count the number of root nodes (exactly one main root is required), and track down "orphans" (nodes pointing to non-existent parents). If errors are detected, the tree construction is safely aborted, and detailed structural errors are logged to the console.
3. **Linear Graph Transformation (`#buildTreeStructure`):** The flat relational input array is transformed into a hierarchical object graph in linear time $O(N)$ using an internal lookup reference map. This prevents expensive, nested iterations and guarantees scalability.
4. **Semantic HTML5 and Event Delegation:** The tree is rendered recursively using native HTML5 tags: `<ul>`, `<li>`, `<details>`, and `<summary>`. As a result, the open/closed state of the branches (`details[open]`) is managed natively by the browser's rendering engine. To minimize RAM consumption and optimize performance, event listeners (`click`, `mouseover`) are attached exclusively to the root container (Event Delegation).

### Advanced Search Engine and Logical Operators

The built-in search subsystem queries an indexed text dictionary and supports three advanced matching modes:

* **Standard Text Search:** Evaluates the presence of a substring (case-insensitive).
* **Regular Expressions (RegExp):** Wrapping a search string in slashes (e.g., `/^index/i`) causes the engine to automatically compile a native `RegExp` object with flags and perform a regex pattern match.
* **Multi-layered Logical Operators (`+` and `*`):**
* The `+` operator acts as a **logical OR (alternation)**. Typing `index + unique` highlights nodes that match at least one of these conditions.
* The `*` operator acts as a **logical AND (conjunction)** and binds tighter than the OR operator. Typing `index * scan` highlights a node only if it contains both substrings simultaneously. These operators can be chained (e.g., `hash * join + nested * loops`).



---

## 🛠️ 2. Public API (The `getAPI()` Interface)

Secure programmatic interaction with the tree component is funneled through a public facade returned by the `getAPI()` method. Below is the detailed specification of the public methods.

### Data Validation and Modification

#### `validateStructure()`

* **Parameters:** None.
* **Return Type:** `Object` (`{ isValid: boolean, errors: string[] }`)
* **Description:** Executes the internal DFS traversal over the raw input graph to check data integrity, looking for cyclic dependencies, missing parent IDs ("orphans"), or invalid root node counts.

#### `setData(newData)`

* **Parameters:** `newData` *(Array<Object>)* – A new flat adjacency list representing the SQL plan.
* **Return Type:** `void`
* **Description:** Dynamically replaces the raw data source. It clears the existing DOM, runs a re-validation routine, and completely rebuilds the graphical tree from scratch.

#### `updateVisuals(newFormatter)`

* **Parameters:** `newFormatter` *(Function)* – A new formatting function that accepts a node object and returns an HTML string.
* **Return Type:** `void`
* **Description:** Refreshes the HTML contents inside the `<summary>` labels for all nodes in real time. This operation preserves the underlying DOM node hierarchy, ensuring the current expansion state (`open` attribute) of all branches remains untouched.

---

### Navigation and Visibility Control

#### `selectNode(id, expand)`

* **Parameters:**
* `id` *(String|Number)* – The unique identifier of the target node.
* `expand` *(Boolean, optional)* – If `true`, recursively forces the entire sub-tree of this node to open. If `false`, collapses it.


* **Return Type:** `void`
* **Description:** Visually activates the chosen node (applies the `.active-branch` class), automatically expands all its ancestors to make it visible on screen, and smoothly centers the view on it.

#### `expandAll()`

* **Parameters:** None.
* **Return Type:** `void`
* **Description:** Globally injects the `open="true"` attribute into every single `<details>` element within the component, fully expanding the whole layout.

#### `collapseAll()`

* **Parameters:** None.
* **Return Type:** `void`
* **Description:** Globally strips out the `open` attribute from all `<details>` tags, instantly collapsing the entire tree back to the top-level root node.

#### `expandToLevel(id, level)`

* **Parameters:**
* `id` *(String|Number)* – The identifier of the starting node.
* `level` *(Number, optional)* – The relative maximum depth to expand. If omitted, expands all descendants. Passing `0` collapses the target node's immediate children.


* **Return Type:** `void`
* **Description:** Offers granular control over how deep a specific node's sub-tree should be exposed by altering HTML disclosure states.

---

### Event Callbacks

#### `onNodeSelect(cb)`

* **Parameters:** `cb` *(Function)* – A callback function triggered upon node selection. Arguments passed: `(nodeData, directChildrenArray, elementSpan)`.
* **Return Type:** `void`
* **Description:** Registers a global hook for click or keyboard selection events, allowing developers to pipe node data into external inspector panels.

#### `onNodeHover(cb)`

* **Parameters:** `cb` *(Function)* – A callback function triggered on pointer hover. Arguments passed: `(nodeData, directChildrenArray, elementSpan)`.
* **Return Type:** `void`
* **Description:** Hooks into the `mouseover` lifecycle event, enabling the programmatic attachment of custom dynamic tooltip components.

---

### Search and Indexing Subsystem

#### `indexTree(indexFn)`

* **Parameters:** `indexFn` *(Function)* – A function that maps a node object to a searchable string index.
* **Return Type:** `void`
* **Description:** Constructs an optimized, flat text dictionary map in cache memory, drastically speeding up subsequent text pattern and logical search executions.

#### `search(criterion, options)`

* **Parameters:**
* `criterion` *(String|RegExp|Function)* – The query target (a raw string, regex instance, or custom filter function).
* `options` *(Object)* – Behavioral setup. The `options.expandMode` property accepts `'lazy'` (opens only paths leading to matches) or `'isolate'` (reveals matches while closing all unrelated branches).


* **Return Type:** `void`
* **Description:** Evaluates the search criteria against the cached index, applies matching visual state classes (`.tree-node-matched`), and adjusts branch visibility accordingly.

#### `clearSearch()`

* **Parameters:** None.
* **Return Type:** `void`
* **Description:** Resets the search framework to its default state: purges all highlight and dimming classes across the DOM and zeros out result cursors.

#### `nextMatch()` / `prevMatch()`

* **Parameters:** None.
* **Return Type:** `void`
* **Description:** Sequentially cycles the active match highlight (`.tree-node-matched-selected`) forward or backward through the filtered results list, ensuring parents are open and the node is scrolled into view.

#### `firstMatch()` / `lastMatch()`

* **Parameters:** None.
* **Return Type:** `void`
* **Description:** Instantaneously teleports the cursor focus and viewport to the absolute first or last match discovered by the query engine.

#### `getSearchState()`

* **Parameters:** None.
* **Return Type:** `Object` (`{ matchedNodeIds: Array, total: Number, currentIndex: Number }`)
* **Description:** Provides a technical state snapshot of the query engine, ideal for updating search hit counters in UI panels.

---

### Topology Analysis and State Migration

#### `getVisualState()`

* **Parameters:** None.
* **Return Type:** `Object` (`{ activeNodeId: String|Number|null, openNodeIds: Array }`)
* **Description:** Generates a lightweight state snapshot tracking the ID of the currently selected active node along with a list of IDs of all currently expanded branches.

#### `restoreVisualState(state)`

* **Parameters:** `state` *(Object)* – A state object previously captured via `getVisualState()`.
* **Return Type:** `void`
* **Description:** Restores a saved tree layout layout configuration, accurately re-opening historical branches and re-selecting the active node.

#### `getParentChainIds(id)`

* **Parameters:** `id` *(String|Number)* – The unique node ID.
* **Return Type:** `Array` (List of parent IDs)
* **Description:** Climbs up the node ancestry link-by-link starting from the given ID up to the root node, gathering an array of all ancestral unique keys.

#### `getChildrenIds(id)`

* **Parameters:** `id` *(String|Number)* – The unique node ID.
* **Return Type:** `Array` (List of direct children IDs)
* **Description:** Scans the layout and extracts a flat array containing only the immediate first-degree children IDs of the specified node.

#### `getExecutionOrder(id, depth)`

* **Parameters:**
* `id` *(String|Number)* – Starting node ID.
* `depth` *(Number)* – Maximum look-down depth boundary.


* **Return Type:** `Array`
* **Description:** Calculates and returns the true execution order sequence of database steps using a Post-Order (Bottom-Up) traversal approach.

---

### Keyboard Navigation Utilities

#### `goUp()` / `goDown()`

* **Parameters:** None.
* **Return Type:** `void`
* **Description:** Shifts the active element selection border (`.active-branch`) to the adjacent visible HTML node directly above or below it in the layout.

#### `goRight()` / `goLeft()`

* **Parameters:** None.
* **Return Type:** `void`
* **Description:** * `goRight`: Expands the currently focused branch.
* `goLeft`: Collapses the currently focused branch. If it is already shut, focus is safely redirected upwards to its immediate parent node.



---

## 🔬 3. Test and Demonstration Module (The `demo` App)

The presentation layer and script logic packed into the lower section of the `json-tree.html` file form a full-fledged sandbox testing utility. It mirrors an enterprise database monitoring console or performance tuning workstation.

### Interface Structure and Layout Architecture

* **Dual-Panel Layout (`.layout`):** The viewport is split into two primary segments: the left side acts as the Tree Sandbox Container (`.tree-side`), while the right area acts as an Event Monitor and Interactive API Testing Lab (`.info-side`).
* **Fluid Mouse Resizer (`side-resizer`):** A physical vertical separation bar (`#drag-bar`) sits between the panels. The client script hooks into `mousedown`, `mousemove`, and `mouseup` window events, translating them into a smooth *Drag & Drop* workspace customizer. Strict layout clamps are put in place (preventing either side from shrinking past 250px) so that vital controls cannot be accidentally dragged off-screen.

### SQL Query Plan Mock Data Generator

The `generateLargeSqlPlan(targetCount)` function emulates an enterprise cost-based optimizer (CBO) behavior. It outputs strictly binary structural configurations, capping node splits at a maximum of two children for operational join conditions like `HASH JOIN` or `NESTED LOOPS`.

The routine includes an advanced backup safety mechanism (*Branch Recovery Mechanism*): if the generator's active processing queue depletes prior to fulfilling the declared `targetCount` parameter, the logic transforms the terminal leaf node into a specialized `VIEW` operation equipped with a `BRANCH RECOVERY` directive, synthetically spawning alternative deep graph clusters. Every generated node contains complex metadata:

* `nodeUID`: Unique operational step ID (the mapped ID property).
* `parentUID`: ID of the supervisor step (the mapped PARENT_ID property).
* `operation`: The database operational name (e.g., `INDEX RANGE SCAN`, `TABLE ACCESS FULL`).
* `options`: Extra tactical execution parameters.
* `card`: Estimated Cardinality (the rows processed count).
* `elapsed_ms`: The physical execution timeframe measured in milliseconds.

### Label Formatting and Cost Visualization

The demo app supplies two individual node formatting hooks that can be swapped live via the UI layout buttons to test dynamic redraw capabilities:

1. **`sqlLabelFormatter(node)` (Default Setting):** Assembles structural node HTML, assessing overhead through the prism of **Estimated Cardinality** (`node.card`). It applies colorful circular metric indicators (`.cost-high` > 800 rows, `.cost-medium` > 400 rows, and `.cost-low` for minor entries).
2. **`sqlLabelFormatterExtra(node)` (Alternative Setting):** Alters the analytical focus. Operational overhead weights and text colors are bound to the **Elapsed Time** property (`node.elapsed_ms`). Furthermore, circles are swapped out for square-blocked indicator badges.

### Callback Hooks and the Event Monitor Panel

When launching the demo app, custom hooks are registered against the active component API instance:

* When `onNodeSelect` or `onNodeHover` are fired, the selected object payload and an array of its **immediate direct children** (with further descendants stripped out for analytical readability) are transformed into structured strings using `JSON.stringify`.
* This formatted text block is pushed directly into the `.info-box` block on the right-hand panel, giving developers a real-time monitor into active dataset boundaries.

### State Snapshot Test Suite

The interface embeds a practical playground area for validating view state preservation features:

* **"Save View State" Button:** Executes the `treeAPI.getVisualState()` method, retrieving the current arrangement and putting it into an app memory slot. The UI updates with a summary detailing how many open branches were captured and the current active selection key.
* **"Restore View State" Button:** Pipes the stored cache block straight back into `treeAPI.restoreVisualState(snapshot)`, immediately restoring the layout and expanding identical nodes.

---

## 🎨 4. Intelligent CSS Style Management (Cascading & States)

The layout presentation is driven by comprehensive CSS rule mapping that reflects internal logical search conditions and focus selections without relying on brittle JavaScript inline style overrides.

### Cascading Search States

When a query operation executes, specialized semantic state classes are applied across tree nodes:

* **`.tree-node-dimmed`**: Introduces an opacity filter (set to `0.35`) onto any elements that do not fit the search criteria, dimming them out so the user's attention is locked onto valid hits.
* **`.tree-node-matched`**: Accents valid hits with a pastel yellow backdrop color (`#fff59d`) and a dashed border frame.
* **`.tree-node-matched-selected`**: Highlights the exact query result hit currently targeted by navigation handles, painting the node container in a bright orange tint with a bold font weight and subtle outer shadows.

### Iconic Design System (CSS Pseudo-elements & Data URI SVG)

* **Vector Graphics with Zero Network Overhead:** Branch expanding and collapsing controls are compiled directly into the style sheet rules as inline, encoded vector SVGs wrapped inside Data URIs using the `::before` pseudo-element on the `<summary>` tag.
* **Transitions and Transformations:** Closed branches show a hollow, dark grey outlined triangle pointing right. When a branch gains the native `details[open]` attribute, the icon's inner background fills with solid color. Hover states activate smooth, circular green scaling accent backdrops around the arrows.
* **The Root Node Exception (Black Square Identity):** The ultimate supervisor database step (ID: 0, typically representing a `SELECT STATEMENT`) bypasses standard arrow logic. It is hardcoded to display a static black square icon symbolizing the query starting point and is completely immune to the circular scaling animations applied to standard branches.
* **Minimalist Mode (`circle="off"`):** Setting a custom `circle="off"` attribute directly on the primary root tree wrapper turns off the circular hover animations, defaulting to sharp, classic geometric node markers with a fixed dimension profile.

---

## ⌨️ 5. Advanced Navigation and Keyboard Shortcuts

The component comes packaged with native keyboard event interceptors (active as long as focus is not inside an editable search box):

* **`Arrow Up` / `Arrow Down**`: Cycles the active selection highlight class (`.active-branch`) through only currently exposed, visible HTML elements. Unrevealed nodes hidden away inside collapsed `<details>` wrappers are bypassed entirely. Upon selecting a new node, `scrollIntoView({ behavior: 'smooth' })` is executed to re-center the viewport.
* **`Arrow Right`**: Forces the currently targeted closed branch to expand (`open = true`).
* **`Arrow Left`**: If the targeted branch is currently open, it immediately collapses it. If it was already closed, focus shifts up to its parent node.
* **`Ctrl + Mouse Click` (Sub-tree Master Override):** A basic click toggle on a branch marker alters the open state of that single element. Holding down the **Ctrl** key while clicking captures the event bubble, triggering a deep recursive script that targets all child containers (`querySelectorAll('details')`) to open or close the **entire nested sub-tree layout simultaneously**.

---

## 🤝 6. Partnership and Co-authorship (Gemini AI Pair Programming)

This open-source component was designed and evolved in a collaborative **AI Pair Programming** workflow alongside **Gemini (Google AI)**.

The core architecture for binary mapping from relational adjacency lists, the state encapsulation patterns (private class properties and the clean public API facade), the multi-layered text-logical search engine supporting RegExp execution, and the deeply recursive keyboard and mouse navigation shortcuts (such as the sub-tree master override via Ctrl+Click) were co-developed, refined, and optimized for high performance through interactive developer dialogue.

---

## 💻 7. How to Run and Test the Project

1. Save the source code file of the demonstration app as `json-tree.html`.
2. Launch the file directly inside any modern web browser (Google Chrome, Mozilla Firefox, Microsoft Edge, Safari). The project functions locally via the **`file://` protocol**, meaning it runs perfectly without needing to spin up local web servers (like Node.js or Apache).
3. Click any text label in the tree structure to activate it, and use the **arrow keys on your keyboard** to test smooth navigation and automatic view centering.
4. Experiment with the search engine in the top control bar: type a search term (e.g., `JOIN`), switch the disclosure configuration to `isolate`, use compound logic patterns (e.g., `INDEX * SCAN + HASH`), or try a regex filter inside slashes (e.g., `/^table/i`). Cycle through the hits using the search control arrow buttons.
5. Use the developer utilities in the bottom-right panel to check structural validation status, generate Post-Order execution chains, query ancestral arrays, or experiment with visual state snapshots.
6. Click and drag the grey vertical dividing bar to adjust the width of the tree side workspace to your liking.

---

## 📝 8. License Conditions

This project is licensed and distributed under the terms of the open **MIT License**. You are free to use, modify, distribute, and integrate this software within proprietary APM frameworks, corporate database query analyzers, or enterprise DBA management consoles free of charge.

The minimalist CSS design patterns and the core concept of utilizing the native HTML5 `<details>` tag for layout trees were inspired by public-domain code blueprints released under the CC0 license by Kate Morley (`iamkate.com`).</Object>
