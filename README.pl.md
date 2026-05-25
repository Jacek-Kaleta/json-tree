# Skalowalne Binarne Drzewo Planu Zapytania SQL (JSON-Tree)

Język: [English (Angielski)](README.md) | **Polski**

---

Interaktywny, wysokowydajny komponent drzewiastego planu wykonania zapytania SQL bazy danych (np. Oracle/PostgreSQL Explain Plan). Drzewo generuje i renderuje strukturę binarną (maksymalnie dwa węzły podrzędne dla każdego operatora, np. dla złączeń `HASH JOIN` czy `NESTED LOOPS`), symulując rzeczywisty plan wykonania składający się z **200 złożonych operacji**.

Projekt został zaimplementowany w czystym technologicznym stosie **Vanilla HTML5, CSS3 oraz ECMAScript 6**, bez użycia jakichquiera zewnętrznych bibliotek czy zależności.

## 🚀 Główne Funkcjonalności

* **Prawdziwa Struktura Binarna:** Algorytm generujący dba o to, aby żaden węzeł (poza liśćmi) nie posiadał więcej niż 2 dzieci, odzwierciedlając faktyczną logikę wykonania bazodanowych złączeń binarnych.
* **Wysoka Wydajność:** Bezproblemowe renderowanie i obsługa zdarzeń dla ponad 200 dynamicznie generowanych rekordów za pomocą delegacji zdarzeń (*Event Delegation*).
* **Inteligentny Interfejs Ikoniczny (CSS Pseudoelements):**
    * **Stan zamknięty:** Ikona reprezentowana jest przez minimalistyczny trójkąt skierowany w prawo, który jest **pusty (biały) w środku** z ciemnoszarą ramką.
    * **Stan otwarty:** Płynna animacja obrotu trójkąta o 90° w dół wraz z automatycznym zapełnieniem kształtu kolorem.
    * **Wyjątek Korzenia:** Główny węzeł bazy danych (ID: 0, `SELECT STATEMENT`) posiada unikalną, statyczną ikonę czarnego kwadratu, symbolizującą punkt startowy/końcowy zapytania.
* **Dynamiczny Efekt Hover:** Najechanie myszką na dowolną ikonę (w tym na korzeń) natychymastowo przekształca ją w zielone kółko akcji z symbolem plusa `+` (dla gałęzi zwiniętych) lub minusa `-` (dla gałęzi rozwiniętych).
* **Zaawansowane Sterowanie Poddrzewem (Skrót Ctrl):**
    * *Kliknięcie standardowe:* Rozwija lub zwija wyłącznie kliknięty węzeł nadrzędny.
    * *Kliknięcie z przytrzymanym klawiszem **Ctrl**:* Rekurencyjnie rozwija lub zwija **całe poddrzewo wraz ze wszystkimi jego głębokimi podgałęziami**.
* **Separacja Akcji Tekstowej:** Kliknięcie w tekst linii (etykietę) nie powoduje otwarcia/zamknięcia drzewa – zamiast tego bezpiecznie podświetla wybraną gałąź i loguje jej wewnętrzne systemowe ID w konsoli programistycznej.

## 🛠️ Architektura i Struktura Kodu

Komponent zamknięty jest w jednym, samowystarczalnym pliku HTML i dzieli się na trzy logiczne warstwy:

### 1. Warstwa Strukturalna (HTML)
Używa semantycznych tagów HTML5: `<ul class="json-tree">` jako kontenera głównego, zestawów list `<li>` oraz natywnych elementów kontrolnych `<details>` i `<summary>`, co gwarantuje pełną dostępność (accessibility).

### 2. Warstwa Prezentacji (CSS3)
* `.json-tree` zarządza zmiennymi CSS (`--spacing`, `--radius`, `--accent`), ułatwiając szybki branding (konfigurację kolorów i odstępów).
* Wszystkie ikony (trójkąty, kwadraty, plusy, minusy) are renderowane jako wektorowe grafiki inline **SVG (Data URI)** wewnątrz pseudoelementów `::before`. Eliminuje to potrzebę dociągania zewnętrznych fontów z ikonami (np. FontAwesome).

### 3. Warstwa Logiki (JavaScript ES6)
* `generateLargeSqlPlan(count)`: Generator losujący realistyczne kroki planu SQL (`HASH JOIN`, `INDEX RANGE SCAN`, itp.) dbający o rygorystyczne zachowanie reguł drzewa binarnego.
* `buildTreeStructure(rows)`: Mapuje płaską strukturę relacyjną `(id, parent_id)` pochodzącą z bazy danych na zagnieżdżony obiekt grafu JSON.
* `renderTreeDOM(node)`: Rekurencyjnie buduje elementy DOM i wstrzykuje je do dokumentu.
* `setupTreeEventListeners(treeContainer)`: Centralny punkt nasłuchiwania zdarzeń. Odpowiada za detekcję klawisza modyfikatora i masowe przełączanie stanu poddrzew.

## 🤝 O projekcie i współpracy

Ten projekt powstał w formule *AI Pair Programming* podczas interaktywnej współpracy z modelem Gemini (Google AI). Architektura drzewa binarnego, zaawansowana logika ikon w czystym CSS oraz obsługa skrótów klawiszowych z modyfikatorem były wspólnie iterowane i dopracowywane w toku deweloperskiej konwersacji. 

Szkielet strukturalny CSS oraz koncepcja czystego drzewa bez JavaScriptu zostały zainspirowane minimalistycznym projektem Kate Morley ([iamkate.com](https://iamkate.com/code/tree-views/)), udostępnionym w domenie publicznej na licencji CC0.

## 💻 Jak Uruchomić Projekt

1. Sklonuj to repozytorium lub pobierz plik z kodem źródłowym.
2. Otwórz plik `index.html` bezpośrednio w dowolnej nowoczesnej przeglądarce internetowej (Chrome, Firefox, Safari, Edge).
3. Otwórz Konsolę Deweloperską (F12), aby obserwować logowanie identyfikatorów wębłów po kliknięciu w tekst planu.

## 📝 Licencja

Projekt udostępniany na licencji MIT. Możesz go dowolnie modyfikować i wdrażać w swoich systemach monitoringu baz danych oraz analizy wydajnościowej.