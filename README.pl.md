# Skalowalne Binarne Drzewo Planu Zapytania SQL (JsonTree)

Język: [English (Angielski)](README.md) | **Polski**

---

Interaktywny, wysokowydajny komponent drzewiastego planu wykonania zapytania SQL bazy danych (np. Oracle/PostgreSQL Explain Plan). Komponent automatycznie mapuje płaskie struktury relacyjne (Adjacency List) na obiektowy graf hierarchiczny i renderuje strukturę binarną (maksymalnie dwa węzły podrzędne dla każdego operatora, np. dla złączeń `HASH JOIN` czy `NESTED LOOPS`), symulując rzeczywisty plan wykonania składający się z **wielu złożonych operacji**.

Projekt został zaimplementowany w czystym technologicznym stosie **Vanilla HTML5, CSS3 oraz ECMAScript 6 (w architekturze zorientowanej obiektowo)**, bez użycia jakichkoľvek zewnętrznych bibliotek czy zależności.

## 🚀 Główne Funkcjonalności

* **Prawdziwa Struktura Binarna:** Algorytm generujący i mapujący dba o to, aby żaden węzeł (poza liśćmi) nie posiadał więcej niż 2 dzieci, odzwierciedlając faktyczną logikę wykonania bazodanowych złączeń binarnych.
* **Nowoczesna Architektura Klasowa:** Całość logiki komponentu została zamknięta w autonomicznej klasie `JsonTree` z wykorzystaniem prywatnych pól i metod (`#`), co gwarantuje pełną enkapsulację kodu, zapobiega wyciekom do globalnego zakresu (*scope*) oraz chroni stan aplikacji.
* **Zaawansowany Podsystem Wyszukiwania:** Komponent posiada wbudowany silnik wyszukiwania operujący na zindeksowanym słowniku tekstowym (`indexTree`), który wspiera trzy tryby dopasowań:
  * *Wyszukiwanie tekstowe (Standardowe):* Analizuje występowanie frazy w sformatowanych etykietach.
  * *Logiczny Operator Wielowarstwowy (`+`):* Wpisanie znaku `+` (np. `index + unique`) dzieli kryterium na podciągi i podświetla węzły spełniające **przynajmniej jeden (ANY)** z podanych warunków.
  * *Wyszukiwanie wyrażeniami regularnymi (RegExp):* Wpisanie frazy zamkniętej w ukośnikach (np. `/^index/i`) automatycznie kompiluje natywny obiekt `RegExp` wraz z flagami.
* **Pełna Nawigacja Klawiszowa (Strzałki):** Komponent umożliwia intuicyjne sterowanie drzewem bezpośrednio z klawiatury (o ile pole wyszukiwania nie jest aktywne):
  * `Strzałka w Górę` / `W Dół` — płynne przechodzenie i aktywacja kolejnych wyłącznie widocznych gałęzi drzewa wraz z automatycznym centrowaniem widoku (*scroll into view*).
  * `Strzałka w Prawo` — dynamiczne rozwinięcie aktualnie wybranej gałęzi (`<details>`).
  * `Strzałka w Lewo` — zwinięcie aktywnej gałęzi, a w przypadku gdy jest już zwinięta — natychmiastowe przeniesienie zaznaczenia na węzeł nadrzędny (rodzica).
* **Zaawansowane Sterowanie Poddrzewem (Skrót Ctrl):**
  * *Kliknięcie standardowe:* Rozwija lub zwija wyłącznie kliknięty węzeł nadrzędny.
  * *Kliknięcie z przytrzymanym klawiszem **Ctrl**:* Przechwytuje zdarzenie i rekurencyjnie rozwija lub zwija **całe poddrzewo wraz ze wszystkimi jego głębokimi podgałęziami**.
* **Tryb Minimalistyczny (`circle="off"`):** Komponent obsługuje dynamiczny mechanizm przełączania stylistyki za pomocą atrybutu `circle` na kontenerze głównym. Po ustawieniu `circle="off"`, dynamiczne, zielone kółka akcji na hoverze zostają całkowicie zablokowane na rzecz eleganckich ikon geometrycznych, zachowując przy tym rygorystyczną blokadę skalowania (stały rozmiar 6px) dla ikony korzenia.
* **Inteligentny Interfejs Ikoniczny (CSS Pseudoelements):**
  * **Stan zamknięty:** Ikona reprezentowana jest przez minimalistyczny trójkąt skierowany w prawo (wektorowe SVG zamienione na Data URI w CSS), który jest pusty w środku z ciemnoszarą ramką.
  * **Stan otwarty:** Płynna animacja obrotu trójkąta o 90° w dół wraz z automatycznym zapełnieniem kształtu kolorem.
  * **Wyjątek Korzenia:** Główny węzeł bazy danych (ID: 0, `SELECT STATEMENT`) posiada unikalną, statyczną ikonę czarnego kwadratu, symbolizującą punkt startowy/końcowy zapytania, odporną na hoverowe transformacje kołowe.

---

## 🏗️ Architektura i Dokumentacja Klasy `JsonTree`

Klasa `JsonTree` stanowi niezależny rdzeń logiczny komponentu, odizolowany od warstwy prezentacji samej strony demonstracyjnej.

### 1. Budowa Strukturalna i DOM
* **Mapowanie Adjacency List:** Wewnątrz prywatnej metody `#buildTreeStructure` płaska tablica relacyjna jest transformowana w hierarchiczny graf obiektów w czasie liniowym przy użyciu mapy identyfikatorów.
* **Semantyczny HTML5:** Metoda `#renderTreeDOM` rekurencyjnie buduje drzewo elementów DOM, wykorzystując tagi `<ul>`, `<li>` oraz parę `<details>` i `<summary>`. Zapewnia to natywne wsparcie stanów otwartości bez konieczności ciągłego modyfikowania struktur JavaScriptem.
* **Delegacja Zdarzeń:** Listenery zdarzeń (`click`, `mouseover`) są przypinane wyłącznie na głównym korzeniu (`#setupEventListeners`), co optymalizuje zużycie pamięci przy bardzo dużych drzewach (dziesiątki tysięcy operacji).

### 2. Publiczne API Klasy (`getAPI()`)
Instancja klasy udostępnia bezpieczną fasadę publiczną, izolując wewnętrzne zmienne i pola prywatne:

* **`expandAll()` / `collapseAll()`**: Globalnie rozwija lub zwija wszystkie gałęzie `<details>` w drzewie.
* **`getActiveId()`**: Zwraca identyfikator (`data-id`) aktualnie zaznaczonego przez użytkownika lub nawigację węzła.
* **`onNodeClick(callback)`**: Rejestruje funkcję zwrotną wywoływaną po kliknięciu myszką lub aktywacji za pomocą strzałek. Przekazuje parametry: `(nodeData, elementSpan)`.
* **`onNodeHover(callback)`**: Rejestruje funkcję zwrotną wywoływaną po najechaniu kursorem myszy na etykietę węzła.
* **`indexTree(indexFn)`**: Buduje wewnętrzny, zoptymalizowany słownik tekstowy przyspieszający wyszukiwanie na podstawie pól zdefiniowanych przez dewelopera w `indexFn`.
* **`search(criterion, options)`**: Uruchamia procedurę przeszukiwania grafu. Parametr `options.expandMode` steruje zachowaniem widoczności:
  * `'lazy'`: Otwiera tylko te węzły, które bezpośrednio ukrywają pod sobą wyniki wyszukiwania (nie modyfikuje stanu innych gałęzi).
  * `'isolate'`: Otwiera wyłącznie ścieżki prowadzące do wyników, automatycznie zamykając wszystkie pozostałe gałęzie w drzewie.
  * `'all'`: Rozwija absolutnie wszystkie elementy w drzewie DOM.
* **`nextMatch()` / `prevMatch()`**: Przechodzi sekwencyjnie do następnego lub poprzedniego trafienia z automatycznym otwieraniem rodziców w górę grafu oraz płynnym przewijaniem widoku.
* **`firstMatch()` / `lastMatch()`**: Przeskakuje bezpośrednio do skrajnych (pierwszego/ostatniego) wyników wyszukiwania.
* **`getSearchState()`**: Zwraca obiekt reprezentujący aktualny stan podsystemu wyszukiwania (tablicę pasujących ID, całkowitą liczbę wyników oraz aktualny indeks pozycji).

---

## 🔬 Opis Przykładu Implementacji

Plik `demo9.html` stanowi kompletną aplikację demonstracyjną, pokazującą wdrożenie klasy `JsonTree` w systemach monitoringu wydajności baz danych. 

### 1. Zewnętrzny Generator i Formatowanie Danych
* **`generateLargeSqlPlan(count)`**: Funkcja symuluje silnik bazy danych i losuje realistyczne kroki planu SQL (`HASH JOIN`, `INDEX RANGE SCAN`, `TABLE ACCESS FULL`). Kontroluje limit maksymalnie dwójki dzieci dla każdego rodzica, generując rygorystyczne drzewo binarne o zadanej wielkości (domyślnie 40 operacji).
* **`sqlLabelFormatter(node)`**: Funkcja formatująca przekazywana do klasy. Generuje bogaty kod HTML dla etykiety węzła: dodaje dynamiczne kropki kosztów (klasy `.cost-high`, `.cost-medium`, `.cost-low` uzależnione od kardynalności `card`) oraz wyświetla metadane operacji (`Rows`, `Time`).

### 2. Layout Strony i Elastyczny Podział Okien (Resizer)
* **Dwupanelowy interfejs**: Układ podzielony jest na panel drzewa po lewej stronie (`.tree-side`) oraz panel Monitora Zdarzeń po prawej stronie (`.info-side`), który w czasie rzeczywistym wyświetla dane z callbacków `onNodeClick` oraz `onNodeHover` w postaci sformatowanych obiektów JSON.
* **Skrypt `side-resizer`**: Sekcja JavaScript odpowiedzialna za obsługę myszkowego paska rozdzielającego (`.resizer`). Obsługuje dynamiczne przeciąganie (*drag&drop*) z walidacją minimalnych i maksymalnych szerokości (sztywne granice bezpieczeństwa 250px dla paneli), zapobiegając uszkodzeniu interfejsu użytkownika.

### 3. Kaskadowe Stany Wyszukiwania w CSS
W warstwie prezentacji zdefiniowano specjalne klasy wizualne sterowane z poziomu metody wyszukiwania klasy:
* `.tree-node-dimmed`: Nadaje 35% przezroczystości elementom, które nie pasują do kryteriów wyszukiwania, skupiając wzrok na wynikach.
* `.tree-node-matched`: Wyróżnia pasujące węzły żółtym tłem i przerywaną ramką.
* `.tree-node-matched-selected`: Oznacza aktualnie podświetlony element nawigacji wyszukiwarki pomarańczowym kolorem oraz cieniem.

---

## 🤝 O projekcie i współpracy

Ten projekt powstał w formule *AI Pair Programming* podczas interaktywnej współpracy z modelem Gemini (Google AI). Architektura drzewa binarnego, zaawansowana logika ikon w czystym CSS, podsystem wyszukiwania (w tym obsługa wyrażeń regularnych oraz operatorów wielowarstwowych) oraz obsługa skrótów klawiszowych były wspólnie iterowane i dopracowywane w toku deweloperskiej konwersacji.

Szkielet strukturalny CSS oraz koncepcja czystego drzewa bez JavaScriptu zostały zainspirowane minimalistycznym projektem Kate Morley ([iamkate.com](https://iamkate.com/code/tree-views/)), udostępnionym w domenie publicznej na licencji CC0.

## 💻 Jak Uruchomić Projekt

1. Sklonuj to repozytorium lub pobierz plik `demo9.html`.
2. Otwórz plik `demo9.html` bezpośrednio w dowolnej nowoczesnej przeglądarce internetowej (Chrome, Firefox, Safari, Edge) — projekt nie wymaga lokalnego serwera HTTP.
3. Kliknij dowolny element w strukturze drzewiastej, aby go uaktywnić, a następnie użyj **strzałek na klawiaturze**, aby przetestować zaawansowaną nawigację.
4. Przetestuj wyszukiwarkę, wybierając różne tryby (`lazy` / `isolate`) lub wpisując filtry z operatorem `+` (np. `TABLE + FULL`) albo wyrażenia regularne (np. `/scan/i`).
5. Użyj paska rozdzielającego pomiędzy panelami, aby dostosować szerokość widoku roboczego.

## 📝 Licencja

Projekt udostępniany na licencji MIT. Możesz go dowolnie modyfikować i wdrażać w swoich systemach monitoringu baz danych oraz komercyjnych analizatorach wydajnościowych.
