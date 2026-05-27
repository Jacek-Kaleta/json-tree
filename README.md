# Scalable Binary SQL Execution Plan Tree (JsonTree)

Language: **English (Angielski)** | [Polish (Polski)](README.md)

---

An interactive, high-performance tree component built to visualize database SQL query execution plans (e.g., Oracle/PostgreSQL Explain Plan). The component generates and renders a binary structure (a maximum of two child nodes for each operator, such as `HASH JOIN` or `NESTED LOOPS`), simulating a real-world execution plan consisting of **multiple complex operations**.

The project is implemented using a clean technology stack of **Vanilla HTML5, CSS3, and ECMAScript 6 (built on an Object-Oriented architecture)**, completely free of external libraries or third-party dependencies.

## 🚀 Key Features

* **True Binary Structure:** The generation algorithm ensures that no node (except leaves) has more than 2 children, precisely reflecting the actual logic of database binary joins.
* **Modern Class-Based Architecture:** The entire component logic is encapsulated within a self-contained `JsonTree` class utilizing private fields and methods (`#`). This ensures complete code encapsulation and prevents scope pollution.
* **Full Keyboard Navigation (Arrow Keys):** The component supports intuitive keyboard-driven control directly out of the box:
* `Arrow Up` / `Arrow Down` — Seamlessly navigate and activate the next/previous visible tree branch, complete with automatic centering (*scroll into view*).
* `Arrow Right` — Dynamically expand the currently active branch.
* `Arrow Left` — Collapse the active branch, or if it is already collapsed, instantly shift focus to its parent node.


* **Advanced Subtree Control (Ctrl Shortcut):**
* *Standard Click:* Expands or collapses only the directly targeted parent node.
* *Ctrl + Click:* Recursively expands or collapses the **entire subtree along with all of its deeply nested branches**.


* **External API & Event Monitor (Callbacks):** The class exposes a public interface allowing the registration of external callbacks (`onNodeClick`, `onNodeHover`). This enables effortless data streaming from the selected node to external dashboards and event logs without modifying the component's core files.
* **Minimalist Mode (`circle="off"`):** The component supports a dynamic visual toggle using the `circle` attribute. When `circle="off"` is set, the animated green interaction circles on hover are completely disabled in favor of elegant geometric icons, while strictly maintaining a scale lock (fixed 6px size) for the root node icon.
* **Intelligent Icon Interface (CSS Pseudoelements):**
* **Collapsed State:** Represented by a minimalist right-pointing triangle that is **hollow (white) inside** with a dark gray border.
* **Expanded State:** Features a smooth 90° downward rotation animation accompanied by an automatic solid color fill.
* **Root Exception:** The primary database node (ID: 0, `SELECT STATEMENT`) possesses a unique, static black square icon symbolizing the query's start/endpoint.



## 🛠️ Architecture and Code Structure

The component is bundled into a single, self-sufficient HTML file and is divided into three logical layers:

### 1. Structural Layer (HTML)

Utilizes semantic HTML5 tags: `<ul class="json-tree">` as the main container, nested list items `<li>`, and native `<details>` and `<summary>` control elements, ensuring native accessibility.

### 2. Presentation Layer (CSS3)

* CSS Variables (`--spacing`, `--radius`, `--accent`) allow for instant branding and layout customization.
* All icons (triangles, squares) are rendered as vector inline **SVG graphics (Data URI)** within `::before` pseudoelements.
* Precise CSS cascades (including targeted selector specificity combinations with `!important`) ensure flawless icon proportions in `circle="off"` mode.

### 3. Logic Layer (`JsonTree` Class & Runtime Scripts)

* `class JsonTree`: The heart of the application. It accepts input configurations, maps flat relational collections, renders the DOM structure, and initializes mouse/keyboard event listeners.
* `getAPI()`: The public interface of the class exposing control methods (e.g., `expandAll`, `collapseAll`, `goUp`, `goDown`, `goLeft`, `goRight`) and event registers.
* `generateLargeSqlPlan(count)`: An external mock generator that spins up realistic SQL plan steps (`HASH JOIN`, `INDEX RANGE SCAN`, etc.), enforcing strict binary tree rules for demonstration purposes.

## 🤝 About the Project and Collaboration

This project was developed using the *AI Pair Programming* methodology during an interactive collaboration with the Gemini model (Google AI). The binary tree architecture, advanced clean CSS icon logic, and keyboard shortcuts with modifiers were jointly iterated and refined throughout the developer conversation.

The CSS structural blueprint and the concept of a clean tree without JavaScript were inspired by a minimalist project by Kate Morley ([iamkate.com](https://iamkate.com/code/tree-views/)), which was released into the public domain under the CC0 license.

## 💻 How to Run

1. Clone this repository or download the source code file.
2. Open `index.html` directly in any modern web browser (Chrome, Firefox, Safari, Edge).
3. Click any element within the tree to activate it, and then use your **keyboard arrow keys** to experience the advanced navigation.
4. Use the side panel to monitor the external Event Tracker powered by the callback API.

## 📝 License

This project is released under the MIT License. You are free to modify, distribute, and integrate it into your database monitoring and performance analysis systems.
