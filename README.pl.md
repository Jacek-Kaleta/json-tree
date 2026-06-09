# Skalowalne Binarne Drzewo Planu Zapytania SQL (JsonTree)

Język: [English (Angielski)](README.md) | **Polski**

Komponent `JsonTree` to wysokowydajne, zorientowane obiektowo rozwiązanie w czystym JavaScript (Vanilla ES6+), HTML5 i CSS3, przeznaczone do wizualizacji hierarchicznych planów wykonania zapytań SQL (np. Oracle/PostgreSQL Explain Plan). Komponent automatycznie mapuje płaskie struktury relacyjne oparte na liście sąsiedztwa (*Adjacency List*) na obiektowy graf hierarchiczny i renderuje go w postaci interaktywnego drzewa.

Projekt nie posiada żadnych zewnętrznych zależności ani bibliotek (zero-dependency). Struktura drzewiasta opiera się na natywnych elementach semantycznych HTML5, co zapewnia doskonałą wydajność pamięciową nawet przy analizie bardzo złożonych drzew operacji bazodanowych.

---

## 🏗️ 1. Architektura i Rdzeń Logiczny: Klasa `JsonTree`

Klasa `JsonTree` stanowi całkowicie niezależne, izolowane jądro aplikacyjne. Wszystkie kluczowe zmienne stanu, metody pomocnicze oraz algorytmy transformacji danych zostały zaimplementowane jako prywatne pola i metody (oznaczone prefiksem `#`), co gwarantuje pełną enkapsulację stanu oraz chroni przed wyciekami do globalnego zakresu (*global scope*).

### Budowa Strukturalna i Cykl Życia

1. **Konstruktor i Mapowanie Kluczy:** Podczas instancjonowania klasa przyjmuje konfigurację mapowania relacji. Pozwala to na elastyczne zdefiniowanie nazw pól odpowiedzialnych za unikalny identyfikator węzła (`id`) oraz identyfikator jego rodzica (`parentId`). Domyślnie wartości te mapowane są odpowiednio na `'ID'` oraz `'PARENT_ID'`.
2. **Techniczna Walidacja i Algorytm DFS:** Przed przystąpieniem do renderowania, w prywatnej metodzie inicjalizacyjnej wywoływana jest funkcja walidacyjna. Wykorzystuje ona algorytm przeszukiwania w głąb (DFS - *Depth-First Search*), aby wykryć ewentualne cykle (zapętlenia) w strukturze planu, zliczyć liczbę korzeni (wymagany jest dokładnie jeden główny węzeł) oraz namierzyć tzw. "sieroty" (węzły wskazujące na nieistniejących rodziców). W przypadku wykrycia błędów proces budowania zostaje przerwany, a błędy są wypisywane w konsoli.
3. **Liniowa Transformacja Grafu (`#buildTreeStructure`):** Płaska tablica relacyjna jest transformowana w hierarchiczne drzewo obiektów w czasie liniowym $O(N)$ przy użyciu pomocniczej mapy referencji, co zapobiega kosztownym, wielokrotnym iteracjom zagnieżdżonym.
4. **Semantyczny HTML5 i Delegacja Zdarzeń:** Drzewo renderowane jest rekurencyjnie z wykorzystaniem tagów `<ul>`, `<li>`, `<details>` oraz `<summary>`. Dzięki temu natywny stan otwartości gałęzi (`details[open]`) jest zarządzany bezpośrednio przez przeglądarkę. Wszystkie nasłuchiwacze zdarzeń (`click`, `mouseover`) są przypięte wyłącznie do głównego korzenia kontenera (delegacja zdarzeń), minimalizując zużycie pamięci operacyjnej RAM.

### Zaawansowany Silnik Wyszukiwania i Operatory Logiczne

Podsystem wyszukiwania operuje na zindeksowanym słowniku tekstowym i obsługuje trzy zaawansowane tryby dopasowań:

* **Wyszukiwanie standardowe (Tekstowe):** Analizuje występowanie frazy (wielkość liter ignorowana).
* **Wyrażenia Regularne (RegExp):** Wpisanie ciągu ograniczonego ukośnikami (np. `/^index/i`) powoduje automatyczne skompilowanie natywnego obiektu `RegExp` wraz z flagami i wykonanie testu dopasowania.
* **Logiczne Operatory Wielowarstwowe (`+` oraz `*`):**
* Operator `+` działa jako **alternatywa logiczna (OR)**. Wpisanie `index + unique` podświetli węzły spełniające przynajmniej jeden z warunków.
* Operator `*` działa jako **koniunkcja logiczna (AND)** i wiąże silniej niż operator alternatywy. Wpisanie `index * scan` podświetli węzeł tylko wtedy, gdy zawiera on jednocześnie oba podciągi. Obie te operacje można łączyć (np. `hash * join + nested * loops`).



---

## 🛠️ 2. Publiczne API Klasy (Interfejs `getAPI()`)

Bezpieczny dostęp do manipulacji drzewem odbywa się poprzez fasadę publiczną zwracaną przez metodę `getAPI()`. Poniżej znajduje się szczegółowy opis dostępnych metod publicznych.

### Sprawdzanie i Modyfikacja Danych

#### `validateStructure()`

* **Parametry:** Brak.
* **Typ zwracany:** `Object` (`{ isValid: boolean, errors: string[] }`)
* **Opis:** Uruchamia wewnętrzny algorytm DFS przechodzący przez graf wejściowy w celu wykrycia pętli, brakujących rodziców ("sierot") lub nieprawidłowej liczby węzłów głównych (korzeni).

#### `setData(newData)`

* **Parametry:** `newData` (Array<Object>) – Nowa płaska tablica relacyjna z danymi planu SQL.
* **Typ zwracany:** `void`
* **Opis:** Dynamicznie podmienia surowe dane wejściowe komponentu. Wywołuje pełną procedurę czyszczenia DOM, ponownej walidacji struktury oraz buduje drzewo od nowa.

#### `updateVisuals(newFormatter)`

* **Parametry:** `newFormatter` *(Function)* – Nowa funkcja formatująca, przyjmująca obiekt węzła i zwracająca ciąg znaków HTML.
* **Typ zwracany:** `void`
* **Opis:** Aktualizuje kod HTML etykiet tekstowych wewnątrz elementów `<summary>` dla wszystkich węzłów. Odbywa się to bez niszczenia struktury drzewa, dzięki czemu aktualny stan rozwinięcia gałęzi (`open`) zostaje w pełni zachowany.

---

### Nawigacja i Sterowanie Widocznością

#### `selectNode(id, expand)`

* **Parametry:** * `id` *(String|Number)* – Unikalny identyfikator węzła do zaznaczenia.
* `expand` *(Boolean, opcjonalny)* – Jeśli `true`, rozwija rekurencyjnie całe poddrzewo danego węzła. Jeśli `false`, zwija je.


* **Typ zwracany:** `void`
* **Opis:** Aktywuje wizualnie wskazany węzeł (nadaje klasę `.active-branch`), automatycznie rozwija wszystkich jego przodków, aby stał się widoczny, oraz centruje na nim widok ekranu.

#### `expandAll()`

* **Parametry:** Brak.
* **Typ zwracany:** `void`
* **Opis:** Globalnie ustawia atrybut `open="true"` na wszystkich elementach `<details>` w drzewie, w pełni je rozwijając.

#### `collapseAll()`

* **Parametry:** Brak.
* **Typ zwracany:** `void`
* **Opis:** Globalnie usuwa atrybut `open` ze wszystkich elementów `<details>` w drzewie, zwijając je do samego korzenia głównego.

#### `expandToLevel(id, level)`

* **Parametry:**
* `id` *(String|Number)* – Identyfikator węzła startowego.
* `level` *(Number, opcjonalny)* – Maksymalna względna głębokość rozwinięcia. Jeśli pominięty, rozwija do samego końca. Wartość `0` zamyka poddrzewo.


* **Typ zwracany:** `void`
* **Opis:** Pozwala kontrolować głębokość wyświetlania poddrzewa wybranego węzła, ustawiając odpowiednie stany widoczności komponentów HTML.

---

### Zdarzenia (Callbacks)

#### `onNodeSelect(cb)`

* **Parametry:** `cb` *(Function)* – Funkcja zwrotna wywoływana przy kliknięciu/wyborze. Przekazuje parametry: `(nodeData, directChildrenArray, elementSpan)`.
* **Typ zwracany:** `void`
* **Opis:** Rejestruje globalny callback dla zdarzenia selekcji. Umożliwia integrację drzewa z zewnętrznymi panelami informacyjnymi.

#### `onNodeHover(cb)`

* **Parametry:** `cb` *(Function)* – Funkcja zwrotna wywoływana po najechaniu kursorem. Przekazuje parametry: `(nodeData, directChildrenArray, elementSpan)`.
* **Typ zwracany:** `void`
* **Opis:** Rejestruje funkcję zwrotną dla zdarzenia najechania myszką (`mouseover`), umożliwiając np. budowanie dynamicznych dymków podpowiedzi (tooltips).

---

### Podsystem Wyszukiwania i Indeksowania

#### `indexTree(indexFn)`

* **Parametry:** `indexFn` *(Function)* – Funkcja mapująca obiekt węzła na tekstowy ciąg indeksowy.
* **Typ zwracany:** `void`
* **Opis:** Buduje zoptymalizowaną, płaską strukturę słownikową w pamięci podręcznej, drastycznie przyspieszając późniejsze operacje wyszukiwania tekstowego i logicznego.

#### `search(criterion, options)`

* **Parametry:**
* `criterion` *(String|RegExp|Function)* – Kryterium wyszukiwania (fraza tekstowa, wyrażenie regularne lub funkcja filtrująca).
* `options` *(Object)* – Konfiguracja zachowania. Pole `options.expandMode` akceptuje wartości: `'lazy'` (otwiera tylko ścieżki z wynikami) lub `'isolate'` (otwiera dopasowania, zamykając pozostałe niepasujące gałęzie).


* **Typ zwracany:** `void`
* **Opis:** Uruchamia zaawansowany silnik dopasowań, nakłada klasy wizualne podświetlenia (`.tree-node-matched`) oraz steruje widocznością kontenerów.

#### `clearSearch()`

* **Parametry:** Brak.
* **Typ zwracany:** `void`
* **Opis:** Przywraca drzewo do stanu domyślnego: czyści klasy podświetleń, usuwa przezroczystości z niepasujących węzłów oraz zeruje wewnętrzne wskaźniki pozycji wyszukiwarki.

#### `nextMatch()` / `prevMatch()`

* **Parametry:** Brak.
* **Typ zwracany:** `void`
* **Opis:** Sekwencyjnie przenosi aktywne podświetlenie (`.tree-node-matched-selected`) na następny lub poprzedni znaleziony element, automatycznie dbając o otwarcie jego rodziców oraz scentrowanie widoku.

#### `firstMatch()` / `lastMatch()`

* **Parametry:** Brak.
* **Typ zwracany:** `void`
* **Opis:** Natychmiastowo przeskakuje do pierwszego lub ostatniego zarejestrowanego na liście trafień elementu wyszukiwarki.

#### `getSearchState()`

* **Parametry:** Brak.
* **Typ zwracany:** `Object` (`{ matchedNodeIds: Array, total: Number, currentIndex: Number }`)
* **Opis:** Zwraca pełne techniczne podsumowanie aktualnego stanu wyszukiwarki, przydatne do aktualizacji liczników w interfejsie użytkownika.

---

### Analiza Topologii i Migracja Stanu

#### `getVisualState()`

* **Parametry:** Brak.
* **Typ zwracany:** `Object` (`{ activeNodeId: String|Number|null, openNodeIds: Array }`)
* **Opis:** Tworzy zrzut pamięci (snapshot) bieżącego stanu widoku – rejestruje identyfikator podświetlonego węzła oraz listę identyfikatorów wszystkich rozwiniętych gałęzi.

#### `restoreVisualState(state)`

* **Parametry:** `state` *(Object)* – Obiekt stanu pozyskany wcześniej z metody `getVisualState()`.
* **Typ zwracany:** `void`
* **Opis:** Przywraca zapisany układ drzewa. Odpowiednio otwiera/zamyka gałęzie oraz nadaje aktywne zaznaczenie wskazanemu węzłowi.

#### `getParentChainIds(id)`

* **Parametry:** `id` *(String|Number)* – Identyfikator węzła.
* **Typ zwracany:** `Array` (Lista identyfikatorów przodków)
* **Opis:** Przechodzi w górę struktury od podanego elementu aż do korzenia planu, zbierając unikalne klucze identyfikacyjne wszystkich rodziców.

#### `getChildrenIds(id)`

* **Parametry:** `id` *(String|Number)* – Identyfikator węzła.
* **Typ zwracany:** `Array` (Lista identyfikatorów bezpośrednich dzieci)
* **Opis:** Zwraca płaską tablicę zawierającą identyfikatory wyłącznie bezpośrednich potomków (pierwszy stopień pokrewieństwa) danego elementu.

#### `getExecutionOrder(id, depth)`

* **Parametry:**
* `id` *(String|Number)* – Identyfikator węzła startowego.
* `depth` *(Number)* – Maksymalna głębokość analizy.


* **Typ zwracany:** `Array`
* **Opis:** Wyznacza rzeczywistą kolejność wykonywania operacji bazodanowych przy użyciu algorytmu przeszukiwania wstecznego (Post-Order / Bottom-Up).

---

### Nawigacja Klawiaturowa

#### `goUp()` / `goDown()`

* **Parametry:** Brak.
* **Typ zwracany:** `void`
* **Opis:** Przesuwa aktywne zaznaczenie na bezpośrednio sąsiadujący (w górę lub w dół) element graficzny, który jest aktualnie widoczny na ekranie.

#### `goRight()` / `goLeft()`

* **Parametry:** Brak.
* **Typ zwracany:** `void`
* **Opis:** * `goRight`: Rozwija aktualnie zaznaczoną gałąź.
* `goLeft`: Zwija aktualnie zaznaczoną gałąź. Jeśli jest już zwinięta – przenosi podświetlenie na jej bezpośredniego rodzica.



---

## 🔬 3. Moduł Testowo-Demonstracyjny (Aplikacja `demo`)

Warstwa prezentacyjna i skrypty osadzone w dolnej części pliku `json-tree.html` implementują w pełni funkcjonalne środowisko testowe, symulujące konsolę administracyjną lub system monitorowania wydajności baz danych.

### Struktura Interfejsu i Architektura Układu

* **Dwupanelowy Layout (`.layout`):** Ekran podzielony jest na dwa główne obszary: lewy panel drzewa (`.tree-side`) oraz prawy Monitor Zdarzeń i Panel Testowy API (`.info-side`).
* **Płynny Resizer Myszkowy (`side-resizer`):** Pomiędzy panelami osadzono pionowy pasek rozdzielający (`#drag-bar`). Skrypt przechwytuje zdarzenia `mousedown`, `mousemove` oraz `mouseup`, realizując płynne przeciąganie oparte o mechanizm *Drag & Drop*. Zaimplementowano sztywne granice bezpieczeństwa (minimalna szerokość paneli wynosi 250px), co zapobiega bezpowrotnemu schowaniu komponentu poza obszar roboczy.

### Generator Danych Testowych Planów SQL

Funkcja `generateLargeSqlPlan(targetCount)` symuluje zachowanie natywnego optymalizatora kosztowego bazy danych. Generuje ona rygorystyczne struktury binarne (maksymalnie dwoje dzieci dla każdego operatora złączeń, takich jak `HASH JOIN` czy `NESTED LOOPS`).

Generator wyposażono w mechanizm awaryjny (*Branch Recovery Mechanism*): jeśli kolejka generowania węzłów wygaśnie przed osiągnięciem zadeklimowanej liczby obiektów (`targetCount`), algorytm dynamicznie przekształca ostatni liść w węzeł typu `VIEW` z opcją `BRANCH RECOVERY`, sztucznie wymuszając dalszą rozbudowę struktur w głąb grafu. Każdy obiekt reprezentujący operację posiada unikalny zestaw metadanych bazodanowych:

* `nodeUID`: Unikalny identyfikator operacji (odpowiednik mapowanego pola ID).
* `parentUID`: Identyfikator kroku nadrzędnego (odpowiednik mapowanego pola PARENT_ID).
* `operation`: Nazwa bazodanowej operacji (np. `INDEX RANGE SCAN`, `TABLE ACCESS FULL`).
* `options`: Dodatkowe parametry wykonania.
* `card`: Estymowana kardynalność (liczba przetworzonych wierszy).
* `elapsed_ms`: Czas wykonania operacji wyrażony w milisekundach.

### Formatowanie Etykiet i Wizualizacja Kosztów

Moduł demonstracyjny dostarcza dwie niezależne funkcje formatujące, które można dynamicznie przełączać w locie za pomocą przycisku interfejsu w celach testowych:

1. **`sqlLabelFormatter(node)` (Formatowanie Domyślne):** Buduje kod HTML węzła, wyliczając koszt operacji na podstawie **kardynalności** (`node.card`). Przypisuje on dynamiczne klasy okrągłych indykatorów wizualnych (`.cost-high` > 800 wierszy, `.cost-medium` > 400 wierszy, `.cost-low` dla mniejszych wartości).
2. **`sqlLabelFormatterExtra(node)` (Formatowanie Alternatywne):** Zmienia kryteria analityczne. Koszt i kolorystyka wyliczane są na podstawie **czasu wykonania** (`node.elapsed_ms`). Dodatkowo, zamiast okrągłych indykatorów, renderowane są geometryczne, kwadratowe kontrolki kosztów.

### Integracja z Callbacami i Monitor Zdarzeń

W momencie inicjalizacji demonstracyjnej aplikacji rejestrowane są funkcje zwrotne interfejsu API:

* Po wywołaniu zdarzenia `onNodeSelect` lub `onNodeHover`, dane zaznaczonego obiektu oraz lista jego **bezpośrednich dzieci** (z odciętymi dalszymi potomkami w celach przejrzystości analitycznej) są przekształcane do czytelnego formatu tekstowego JSON za pomocą `JSON.stringify`.
* Sformatowany ciąg jest natychmiast wstrzykiwany do kontenera `.info-box` po prawej stronie, dając deweloperowi podgląd parametrów operacji w czasie rzeczywistym.

### Panel Odtwarzania i Zapisu Stanu (Snapshot)

Aplikacja demonstracyjna udostępnia dedykowany interfejs do testowania funkcji migracji widoku:

* **Przycisk "Zapisz stan widoku":** Wywołuje metodę `treeAPI.getVisualState()`, pobiera aktualny stan i zapisuje gałęzie w pamięci podręcznej. Na ekranie wyświetla się podsumowanie informujące o liczbie zapamiętanych otwartych gałęzi oraz identyfikatorze aktywnego węzła.
* **Przycisk "Odtwórz stan widoku":** Przekazuje zapisany obiekt z powrotem do metody `treeAPI.restoreVisualState(snapshot)`, automatycznie przywracając pełny układ widoczności elementów, który użytkownik wcześniej skonfigurował.

---

## 🎨 4. Inteligentne Zarządzanie Stylami CSS (Kaskadowość i Stany)

Prezentacja wizualna opiera się na zaawansowanych regułach CSS, które ściśle odzwierciedlają przekazywane logiczne stany wyszukiwania i nawigacji bez konieczności ciągłej manipulacji stylami inline z poziomu kodu JavaScript.

### Kaskadowe Stany Wyszukiwania

Podczas wykonywania operacji wyszukiwania, na elementy drzewa nakładane są specjalne klasy semantyczne:

* **`.tree-node-dimmed`**: Nadaje styl przezroczystości (ustawiony na `0.35`) dla wszystkich węzłów, które nie spełniają kryteriów wyszukiwania, pozwalając użytkownikowi skupić pełną uwagę na pasujących wynikach.
* **`.tree-node-matched`**: Wyróżnia pasujące węzły za pomocą pastelowego żółtego tła (`#fff59d`) oraz przerywanej ramki.
* **`.tree-node-matched-selected`**: Oznacza aktualnie wskazany za pomocą nawigacji wynik wyszukiwania. Węzeł zyskuje jaskrawe pomarańczowe tło, pogrubioną czcionkę oraz delikatny cień wizualny.

### Interfejs Ikoniczny (CSS Pseudo-elements i Data URI SVG)

* **Ikony Wektorowe bez Plików Graficznych:** Ikony otwierania i zamykania gałęzi zostały osadzone bezpośrednio w arkuszu stylów jako zakodowane wektory SVG w formacie Data URI przy użyciu pseudoelementu `::before` na tagu `summary`.
* **Animacja i Transformacja:** Stan zamknięty reprezentowany jest przez pusty w środku trójkąt skierowany w prawo z ciemnoszarą ramką. Gdy element nadrzędny zyskuje atrybut `details[open]`, tło ikony automatycznie wypełnia się kolorem. Przejścia na hoverze aktywują dynamiczne, zielone kółka transformacji kołowej wokół strzałek.
* **Wyjątek Korzenia Planu (Root Black Square):** Główny węzeł bazy danych (ID: 0, reprezentujący instrukcję `SELECT STATEMENT`) posiada nadpisaną, unikalną regułę stylistyczną. Jego ikona to statyczny, czarny kwadrat symbolizujący punkt startowy zapytania, w pełni odporny na transformacje kołowe występujące na pozostałych gałęziach.
* **Tryb Minimalistyczny (`circle="off"`):** Ustawienie atrybutu `circle="off"` na kontenerze głównym powoduje wyłączenie okrągłych, zielonych animacji na hoverze na rzecz surowych, klasycznych ikon geometrycznych o stałym rozmiarze.

---

## ⌨️ 5. Zaawansowane Sterowanie i Skróty Klawiszowe

Komponent implementuje pełną nawigację z klawiatury (aktywną pod warunkiem, że kursor tekstowy nie znajduje się w polu wyszukiwania):

* **`Strzałka w Górę` / `Strzałka w Dół**`: Płynnie przenosi aktywne zaznaczenie klasy `.active-branch` pomiędzy wyłącznie aktualnie rozwiniętymi i widocznymi dla oka użytkownika gałęziami. Elementy ukryte wewnątrz zamkniętych sekcji `<details>` są pomijane. Po aktywacji węzła wywoływana jest metoda `scrollIntoView({ behavior: 'smooth' })`, automatycznie centrując widok ekranu.
* **`Strzałka w Prawo`**: Dynamicznie rozwija (`open = true`) aktualnie zaznaczoną gałąź.
* **`Strzałka w Lewo`**: Jeśli gałąź jest rozwinięta – natychmiast ją zwija. Jeśli gałąź była już zamknięta – zaznaczenie automatycznie i bezpiecznie wędruje w górę, aktywując węzeł rodzica.
* **`Myszka + Klawisz Ctrl` (Sterowanie Poddrzewem):** Standardowe kliknięcie w ikonę rozwinięcia modyfikuje stan wyłącznie jednego elementu. Kliknięcie z przytrzymanym klawiszem **Ctrl** przechwytuje bąbelkowanie zdarzenia i rekurencyjnie otwiera lub zamyka **całe poddrzewo wraz ze wszystkimi jego głębokimi rozgałęzieniami potomnymi** za pomocą `querySelectorAll('details')`.

---

## 🤝 6. Współpraca i Współautorstwo (Gemini AI Pair Programming)

Ten projekt powstał i ewoluował w formule **AI Pair Programming** podczas intensywnej, interaktywnej współpracy z modelem **Gemini (Google AI)**.

Architektura binarnego mapowania grafów relacyjnych, zaawansowane mechanizmy enkapsulacji stanu (prywatne pola i fasada API), wielowarstwowy silnik wyszukiwania tekstowo-logicznego z obsługą wyrażeń regularnych oraz rozbudowane sterowanie klawiaturowe z obsługą głębokiej rekurencji przodków (Ctrl+Click) były wspólnie projektowane, iterowane i optymalizowane pod kątem wydajności strukturalnej w toku zaawansowanej konwersacji deweloperskiej.

---

## 💻 7. Jak Uruchomić i Testować Projekt

1. Pobierz plik źródłowy aplikacji demonstracyjnej `json-tree.html`.
2. Otwórz plik bezpośrednio w dowolnej współczesnej przeglądarce internetowej (Google Chrome, Mozilla Firefox, Microsoft Edge, Safari) – projekt działa lokalnie i **na bazie protokołu `file://**`, co oznacza, że nie wymaga uruchamiania lokalnego serwera HTTP (np. Node.js, Apache).
3. Kliknij dowolną etykietę tekstową w strukturze drzewiastej, aby ją uaktywnić, a następnie użyj **strzałek na klawiaturze**, aby przetestować płynną nawigację oraz automatyczne centrowanie widoku.
4. Przetestuj wyszukiwarkę w górnym panelu: wpisz frazę (np. `JOIN`), wybierz tryb izolacji (`isolate`), użyj operatorów złożonych (np. `INDEX * SCAN + HASH`) lub wpisz wyrażenie regularne w ukośnikach (np. `/^table/i`). Przełączaj wyniki przyciskami nawigacyjnymi wyszukiwarki.
5. Przetestuj funkcje zaawansowane w dolnym panelu deweloperskim (pobieranie kolejności wykonywania Post-Order, sprawdzanie błędów walidacji danych, pobieranie łańcucha ID przodków czy symulację zapisu snapshotu widoku).
6. Chwyć myszką za szary pasek rozdzielający panele i dostosuj szerokość obszaru roboczego drzewa do własnych preferencji.

---

## 📝 8. Warunki Licencyjne

Projekt dystrybuowany jest na otwartej licencji **MIT**. Zezwala ona na pełne, darmowe wykorzystanie komercyjne, modyfikowanie kodu, dystrybucję oraz implementację komponentu wewnątrz autorskich systemów APM, korporacyjnych analizatorów wydajnościowych baz danych czy konsol administracyjnych DBA.

Szkielet strukturalny CSS oraz koncepcja czystego renderowania drzewa bazującego na tagu `<details>` zostały zainspirowane minimalistycznymi pracami udostępnionymi w domenie publicznej na licencji CC0 przez Kate Morley (`iamkate.com`).
