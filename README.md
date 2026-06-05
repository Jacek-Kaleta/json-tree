# Scalable Binary SQL Query Plan Tree (JsonTree)

Language: **English** | [Polish (Polski)](README.pl.md)

---

An interactive, high-performance tree component designed for database SQL query execution plans (e.g., Oracle/PostgreSQL Explain Plan). The component automatically maps flat relational structures (Adjacency List) into an object-oriented hierarchical graph and renders a binary structure (a maximum of two child nodes for each operator, such as for `HASH JOIN` or `NESTED LOOPS`), simulating a real execution plan consisting of **multiple complex operations**.

The project is implemented using a pure technological stack of **Vanilla HTML5, CSS3, and ECMAScript 6 (in an object-oriented architecture)**, without any external libraries or dependencies.

## 🚀 Key Features

* **True Binary Structure:** The generation and mapping algorithm ensures that no node (except leaves) has more than 2 children, mirroring the actual execution logic of binary database joins.
* **Modern Class Architecture:** All component logic is encapsulated within an autonomous `JsonTree` class using private fields and methods (`#`), ensuring full code encapsulation, preventing global scope leaks, and protecting the application state.
* **Advanced Search Subsystem:** The component features a built-in search engine operating on an indexed text dictionary (`indexTree`), which supports three match modes:
  * *Text Search (Standard):* Analyzes the occurrence of a phrase within formatted labels.
  * *Multi-layered Logical Operator (`+`):* Entering a `+` sign (e.g., `index + unique`) splits the criterion into substrings and highlights nodes matching **at least one (ANY)** of the specified conditions.
  * *Regular Expression Search (RegExp):* Inputting a phrase enclosed in forward slashes (e.g., `/^index/i`) automatically compiles a native `RegExp` object along with its flags.
* **Full Keyboard Navigation (Arrows):** The component allows intuitive tree control directly from the keyboard (provided the search input field is not active):
  * `Up Arrow` / `Down Arrow` — Smoothly navigates and activates the next or previous visible branches of the tree, complete with automatic centering (*scroll into view*).
  * `Right Arrow` — Dynamically expands the currently selected branch (`<details>`).
  * `Left Arrow` — Collapses the active branch; if it is already collapsed, it immediately moves the selection to the parent node.
* **Advanced Subtree Control (Ctrl Shortcut):**
  * *Standard Click:* Expands or collapses only the clicked parent node.
  * *Click with **Ctrl** held down:* Captures the event and recursively expands or collapses the **entire subtree along with all its deep sub-branches**.
* **Flexible UI Splitting (Resizer):** The two-panel layout interface (tree on the left, event monitor on the right) features a native, mouse-driven splitter bar (`.resizer`). It supports dynamic drag-and-drop operations with panel width constraints (strict safety limits of 250px per panel).
* **Minimalist Mode (`circle="off"`):** The component supports dynamic style switching via the `circle` attribute on the main container. When set to `circle="off"`, the dynamic green action circles on hover are completely disabled in favor of elegant geometric icons, while maintaining a strict scaling restriction (fixed 6px size) for the root icon.
* **Intelligent Icon Interface (CSS Pseudoelements):**
  * **Closed State:** The icon is represented by a minimalist right-pointing triangle (vector SVG converted to Data URI in CSS) that is hollow (white) inside with a dark grey border.
  * **Open State:** A smooth 90° downward rotation animation of the triangle combined with an automatic solid color fill of the shape.
  * **Root Exception:** The main database node (ID: 0, `SELECT STATEMENT`) features a unique, static black square icon symbolizing the query's start/end point, which remains unaffected by hover-induced circular transformations.

---

## 🏗️ Class Architecture and Documentation (`JsonTree`)

The `JsonTree` class serves as the independent logical core of the component, fully isolated from the presentation layer of the demo page itself.

### 1. Structural Design and DOM Generation
* **Adjacency List Mapping:** Within the private `#buildTreeStructure` method, the flat relational array is transformed into a hierarchical object graph in linear time using an identifier map.
* **Semantic HTML5:** The `#renderTreeDOM` method recursively builds the DOM tree structure using `<ul>`, `<li>`, and native `<details>` and `<summary>` element pairs. This ensures native support for open/closed states without needing to constantly manipulate structures using JavaScript.
* **Event Delegation:** Event listeners (`click`, `mouseover`) are bound exclusively to the main root element (`#setupEventListeners`), optimizing memory consumption when handling very large trees (tens of thousands of operations).

### 2. Public API Fasade (`getAPI()`)
The class instance exposes a secure public facade, isolating internal variables and private fields:

* **`expandAll()` / `collapseAll()`**: Globally expands or collapses all `<details>` branches within the DOM tree.
* **`getActiveId()`**: Returns the identifier (`data-id`) of the currently selected node.
* **`onNodeClick(callback)`**: Registers a callback function triggered upon mouse click or arrow-key activation. Passes the parameters: `(nodeData, elementSpan)`.
* **`onNodeHover(callback)`**: Registers a callback function triggered when hovering the mouse cursor over a node text label.
* **`indexTree(indexFn)`**: Builds an internal, optimized text dictionary to accelerate search operations based on fields specified by the developer in `indexFn`.
* **`search(criterion, options)`**: Executes the graph search procedure. The `options.expandMode` parameter controls tree visibility behavior:
  * `'lazy'`: Opens only the specific nodes that directly hide search results underneath (leaves other branches' states unchanged).
  * `'isolate'`: Opens exclusively the paths leading to the matched results, automatically collapsing all other branches in the tree.
  * `'all'`: Expands absolutely every element within the DOM tree.
* **`nextMatch()` / `prevMatch()`**: Iterates sequentially through the next or previous search hits, automatically expanding ancestors up the graph hierarchy and smoothly scrolling the target element into view.
* **`firstMatch()` / `lastMatch()`**: Jumps directly to the boundary results (first or last) of the search operation.
* **`getSearchState()`**: Returns an object representing the current state of the search subsystem (the array of matching IDs, the total number of hits, and the current position index).

---

## 🔬 Implementation Example Description (`demo9.html`)

The `demo9.html` file serves as a comprehensive demonstration application, showcasing the deployment of the `JsonTree` class within database performance monitoring systems.

### 1. External Generator and Data Formatting
* **`generateLargeSqlPlan(count)`**: This function simulates a database engine, generating randomized yet realistic SQL plan steps (`HASH JOIN`, `INDEX RANGE SCAN`, `TABLE ACCESS FULL`). It enforces a strict maximum limit of two children per parent node to output a proper binary tree of a given size (default is 40 operations).
* **`sqlLabelFormatter(node)`**: The formatting function passed into the class configuration. It generates rich HTML markup for the node label, appending dynamic cost dots (classes `.cost-high`, `.cost-medium`, and `.cost-low` determined by the node's cardinality `card`) and displaying operation metadata (`Rows`, `Time`).

### 2. Page Layout and Flexible Panel Splitting (Resizer)
* **Two-Panel Layout**: The user interface is split into a tree panel on the left side (`.tree-side`) and an Event Monitor panel on the right side (`.info-side`). The monitor displays data in real time from the `onNodeClick` and `onNodeHover` callbacks as formatted JSON objects.
* **`side-resizer` Script**: The JavaScript block responsible for handling the mouse-driven splitter bar (`.resizer`). It manages dynamic drag-and-drop actions while validating minimum and maximum widths (a strict safety threshold of 250px for both panels), preventing UI breakage during resizing.

### 3. Cascading Search States in CSS
Special presentation classes are defined in the CSS layer and are dynamically managed by the class's search method:
* `.tree-node-dimmed`: Applies a 35% opacity to elements that do not match the search criteria, drawing the user's focus to the relevant hits.
* `.tree-node-matched`: Highlights matching nodes with a yellow background and a dashed border.
* `.tree-node-matched-selected`: Marks the currently focused search navigation item with an orange background and a subtle shadow.

---

## 🤝 Project Origin and Collaboration

This project was developed under an *AI Pair Programming* model through an interactive collaboration with the Gemini model (Google AI). The binary tree architecture, the advanced icon logic utilizing pure CSS, the robust search engine subsystem (including regular expression parsing and multi-layered conditions), and the arrow-key modifier shortcuts were all iteratively co-developed and polished during the engineering conversation.

The structural CSS scaffolding and the core concept of a clean tree layout without heavy JavaScript manipulation were inspired by Kate Morley's minimalist project ([iamkate.com](https://iamkate.com/code/tree-views/)), which is dedicated to the public domain under the CC0 license.

## 💻 How to Run the Project

1. Clone this repository or download the `demo9.html` file.
2. Open the `demo9.html` file directly in any modern web browser (Chrome, Firefox, Safari, Edge) — the project does not require a local HTTP server to run.
3. Click any element inside the tree structure to activate it, and then use the **arrow keys on your keyboard** to test the advanced navigation.
4. Try out the search bar by typing strings with the `+` operator (e.g., `TABLE + FULL`), regular expressions (e.g., `/scan/i`), and switching between the different expansion modes (`lazy` / `isolate`).
5. Use the splitter bar between the panels to adjust the width of your active workspace.

## 📝 License

This project is released under the MIT License. You are free to modify and integrate it into your own database monitoring systems and commercial performance analytics suites.
