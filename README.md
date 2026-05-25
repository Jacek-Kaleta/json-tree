# Scalable Binary SQL Execution Plan Tree (JSON-Tree)

Language: **English** | [Polski (Polish)](README.pl.md)

---

An interactive, high-performance tree component designed for visualizing database SQL execution plans (e.g., Oracle/PostgreSQL Explain Plan). The component dynamically generates and renders a strict **binary tree structure** (a maximum of two child nodes per operator to accurately simulate database joins like `HASH JOIN` or `NESTED LOOPS`), handling a comprehensive mock dataset of **200 complex database operations**.

The project is built entirely using a lightweight vanilla stack: **HTML5, CSS3, and ECMAScript 6**, without any external libraries or framework dependencies.

## 🚀 Key Features

* **Strict Binary Structure:** The generation algorithm ensures that no parent node has more than 2 children, mirroring the actual execution logic of binary database engines.
* **High Performance:** Smooth rendering and ultra-fast interaction handling for 200+ dynamic records using **Event Delegation**.
* **Smart Icon System (CSS Pseudoelements):**
    * **Collapsed State:** Represented by a minimalist, right-pointing triangle that is **hollow (white inside)** with a sharp dark-grey border.
    * **Expanded State:** Smoothly animates into a 90° downward rotation and automatically transitions into a solid, filled triangle shape.
    * **Root Node Exception:** The main query entry point (ID: 0, `SELECT STATEMENT`) features a permanent, static black square icon, symbolizing the start/end point of the execution flow.
* **Dynamic Hover Effects:** Hovering over any active icon (including the root node) instantly transforms it into a green action circle containing a `+` symbol (for collapsed branches) or a `-` symbol (for expanded branches).
* **Advanced Subtree Control (Ctrl Shortcut):**
    * *Standard Click:* Expands or collapses only the single, targeted parent node.
    * *Ctrl + Click:* Recursively expands or collapses the **entire subtree**, including all nested deeply layered child branches at once.
* **Text Selection Guard:** Clicking on the actual line text (label) does not trigger tree collapsing/expanding. Instead, it securely highlights the selected line and logs its internal database row ID into the browser developer console.

## 🛠️ Architecture & Code Layout

The component is entirely self-contained within a single HTML file, cleanly decoupled into three architectural layers:

### 1. Structural Layer (HTML)
Utilizes semantic HTML5 markup featuring `<ul class="json-tree">` as the primary wrapper, layout-friendly list elements `<li>`, and native browser interactive disclosures (`<details>` and `<summary>`), ensuring baseline accessibility.

### 2. Presentation Layer (CSS3)
* The `.json-tree` class acts as the centralized design-system hub, managing CSS custom properties (`--spacing`, `--radius`, `--accent`) for effortless theme branding and padding adjustment.
* All UI state icons (triangles, squares, pluses, minuses) are injected as vector-based **inline SVG (Data URIs)** within `::before` pseudoelements. This eliminates the need for downloading bulky external icon fonts (e.g., FontAwesome).

### 3. Logic Layer (JavaScript ES6)
* `generateLargeSqlPlan(count)`: Randomly builds realistic SQL execution plan steps (`HASH JOIN`, `INDEX RANGE SCAN`, etc.), enforcing a strict binary hierarchy constraint.
* `buildTreeStructure(rows)`: Transforms flat relational database parent-child rows `(id, parent_id)` into a deeply nested JSON graph object.
* `renderTreeDOM(node)`: Recursively builds DOM fragments and injects them into the viewport.
* `setupTreeEventListeners(treeContainer)`: The centralized event dispatcher. It actively checks for keyboard modifier states to toggle massive structural updates.

## 🤝 Development & Acknowledgments

This project was developed through an interactive AI pair-programming collaboration between the author and Gemini (Google AI). The core architecture, custom CSS-driven icon logic, and responsive keyboard event modifiers were designed and iteratively refined through contextual conversation to achieve production-grade UI performance.

## 💻 Getting Started

1. Clone this repository or copy the code files locally.
2. Open `index.html` directly in any modern web browser (Chrome, Firefox, Safari, Edge).
3. Open your browser's Developer Tools Console (F12) to monitor database record ID outputs when interacting with plan labels.

## 📝 License

This project is licensed under the MIT License. Feel free to modify, distribute, and integrate this tool into your own database performance monitors or custom APM dashboards.
