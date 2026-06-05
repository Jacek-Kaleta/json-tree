Oto zaktualizowana i przeorganizowana treść dokumentu **README.pl.md**.

W nowej wersji uwzględniłem zaawansowane funkcjonalności wykryte bezpośrednio w pliku `demo9.html` (takie jak **kompleksowy podsystem wyszukiwania**, operator `+`, obsługa wyrażeń regularnych oraz dynamiczny, myszkowy **splitter paneli layoutu**). Zgodnie z Twoją prośbą, wyraźnie odseparowałem specyfikację techniczną samej klasy `JsonTree` od opisu jej wdrożenia w demonstracyjnej aplikacji.

---

```markdown
# Skalowalne Binarne Drzewo Planu Zapytania SQL (JsonTree)

Język: [English (Angielski)](README.md) | **Polski**

---

Interaktywny, wysokowydajny komponent drzewiastego planu wykonania zapytania SQL bazy danych (np. Oracle/PostgreSQL Explain Plan). Komponent generuje i renderuje strukturę z płaskiej relacji rodzic-dziecko (*Adjacency List*), optymalizując ją pod kątem planów binarnych (maksymalnie dwa węzły podrzędne dla każdego operatora złączeń, np. `HASH JOIN` czy `NESTED LOOPS`).

Projekt został zaimplementowany w czystym technologicznym stosie **Vanilla HTML5, CSS3 oraz ECMAScript 6 (w architekturze zorientowanej obiektowo)**, bez użycia jakichkolwiek zewnętrznych bibliotek czy zależności.

## 🚀 Główne Funkcjonalności Komponentu

* **Prawdziwa Struktura Binarna:** Algorytm generujący dba o to, aby żaden węzeł (poza liśćmi) nie posiadał więcej niż 2 dzieci, odzwierciedlając faktyczną logikę wykonania bazodanowych złączeń binarnych.
* **Nowoczesna Architektura Klasowa:** Całość logiki komponentu została zamknięta w autonomicznej klasie `JsonTree` z wykorzystaniem prywatnych pól i metod (`#`), co gwarantuje pełną enkapsulację kodu i zapobiega wyciekom do globalnego zakresu (*scope*).
* **Pełna Nawigacja Klawiszowa (Strzałki):** Komponent umożliwia intuicyjne sterowanie drzewem bezpośrednio z klawiatury:
  * `Strzałka w Górę` / `W Dół` — płynne przechodzenie i aktywacja kolejnych widocznych gałęzi drzewa wraz z automatycznym centrowaniem widoku (*scroll into view*).
  * `Strzałka w Prawo` — dynamiczne rozwinięcie aktualnie wybranej gałęzi.
  * `Strzałka w Lewo` — zwinięcie aktywnej gałęzi, a w przypadku gdy jest już zwinięta — natychmiastowe przeniesienie zaznaczenia na węzeł nadrzędny (rodzica).
* **Zaawansowane Sterowanie Poddrzewem (Skrót Ctrl):**
  * *Kliknięcie standardowe:* Rozwija lub zwija wyłącznie kliknięty węzeł nadrzędny.
  * *Kliknięcie z przytrzymanym klawiszem **Ctrl**:* Rekurencyjnie rozwija lub zwija **całe poddrzewo wraz ze wszystkimi jego głębokimi podgałęziami**.
* **Tryb Minimalistyczny (`circle="off"`):** Komponent obsługuje dynamiczny mechanizm przełączania stylistyki za pomocą atrybutu `circle`. Po ustawieniu `circle="off"`, dynamiczne, zielone kółka akcji na hoverze zostają całkowicie zablokowane na rzecz eleganckich ikon geometrycznych, zachowując przy tym rygorystyczną blokadę skalowania (stały rozmiar 6px) dla ikony korzenia.
* **Inteligentny Interfejs Ikoniczny (CSS Pseudoelements):**
  * **Stan zamknięty:** Ikona reprezentowana jest przez minimalistyczny trójkąt skierowany w prawo, który jest pusty (biały) w środku z ciemnoszarą ramką.
  * **Stan otwarty:** Płynna animacja obrotu trójkąta o 90° w dół wraz z automatycznym zapełnieniem kształtu kolorem.
  * **Wyjątek Korzenia:** Główny węzeł bazy danych (ID: 0, `SELECT STATEMENT`) posiada unikalną, statyczną ikonę czarnego kwadratu, symbolizującą punkt startowy/końcowy zapytania.

---

## 🛠️ Dokumentacja Klasy `JsonTree`

Klasa `JsonTree` odpowiada za mapowanie danych, budowanie drzewa DOM oraz zarządzanie wewnętrznym stanem interakcji, wyszukiwania i nawigacji.

### Konstruktor

```javascript
const tree = new JsonTree({ data, targetElement, formatter, mapping });

```

* **`data`** *(Array<Object>)*: Płaska tablica obiektów wejściowych (relacja tabelaryczna).
* **`targetElement`** *(HTMLElement)*: Kontener DOM, do którego zostanie wstrzyknięte drzewo.
* **`formatter`** *(Function)*: Callback `function(node): string` zwracający kod HTML reprezentujący zawartość (etykietę) danego węzła.
* **`mapping`** *(Object)*: Opcjonalne mapowanie nazw kluczy identyfikatorów:
* `id` (domyślnie `'id'`): Klucz unikalnego identyfikatora węzła.
* `parentId` (domyślnie `'parent_id'`): Klucz wskazujący na identyfikator rodzica.



### Publiczne API (Fasada udostępniana przez `getAPI()`)

Metoda `getAPI()` zwraca bezpieczny obiekt fasady publicznej, izolujący wewnętrzny stan klasy.

#### 1. Zarządzanie Widocznością i Stanem

* **`expandAll()`**: Rozwija globalnie wszystkie gałęzie w drzewie.
* **`collapseAll()`**: Zwija globalnie wszystkie gałęzie w drzewie.
* **`getActiveId()`**: Zwraca ID aktualnie zaznaczonego elementu (`string | number | null`).

#### 2. Rejestracja Callbacków Zdarzeń

* **`onNodeClick(callback)`**: Rejestruje funkcję wywoływaną po kliknięciu lub klawiaturowej aktywacji węzła. Przekazuje parametry: `(nodeData, htmlElement)`.
* **`onNodeHover(callback)`**: Rejestruje funkcję wywoływaną po najechaniu myszą na tekst węzła. Przekazuje parametry: `(nodeData, htmlElement)`.

#### 3. Podsystem Wyszukiwania i Indeksowania

* **`indexTree(indexFn)`**: Tworzy wewnętrzny, zindeksowany słownik tekstowy na podstawie przekazanej funkcji `indexFn(node)`. Optymalizuje to prędkość wyszukiwania dla dużych struktur danych.
* **`search(criterion, options)`**: Wykonuje zaawansowaną operację filtrowania i wyróżniania wyników.
* `criterion` *(string | RegExp | function)*: Szukana fraza, wyrażenie regularne (np. `/^index/i`) lub funkcja ewaluacyjna. W przypadku ciągu znaków obsługuje **multi-wyszukiwanie za pomocą operatora `+**` (np. `index + unique` dopasuje węzły spełniające *przynajmniej jeden* z tych warunków).
* `options.expandMode` *('lazy' | 'isolate' | 'all')*: Sposób traktowania widoczności gałęzi po wyszukaniu:
* `'lazy'`: Otwiera tylko te zamknięte węzły, które skrywają pod sobą trafienia (nie modyfikuje pozostałych).
* `'isolate'`: Otwiera wyłącznie ścieżki prowadzące do pasujących węzłów, automatycznie zamykając wszystkie inne gałęzie.
* `'all'`: Przymusowo rozwija absolutnie wszystkie elementy w drzewie.




* **`nextMatch()` / `prevMatch()**`: Przechodzi do kolejnego/poprzedniego dopasowania wyszukiwania (zgodnie z kolejnością występowania w strukturze DOM), nadając mu status *selected*, usuwając przygaszenie i automatycznie rozwijając jego logiczną ścieżkę rodziców.
* **`firstMatch()` / `lastMatch()**`: Przeskakuje bezpośrednio do pierwszego lub ostatniego odnalezionego wyniku.
* **`getSearchState()`**: Zwraca klon aktualnego stanu podsystemu wyszukiwawczego w formacie:
```javascript
{ matchedIds: Array, currentIndex: number, totalMatches: number }

```



#### 4. Programowa Nawigacja Klawiaturowa

* **`goUp()` / `goDown()**`: Przesuwa podświetlenie na sąsiadujący, widoczny wyżej lub niżej element.
* **`goRight()`**: Otwiera aktualnie podświetloną gałąź.
* **`goLeft()`**: Zwija aktualną gałąź lub przechodzi na jej rodzica.

---

## 🖥️ Aplikacja Demonstracyjna (`demo9.html`)

Plik `demo9.html` stanowi kompletną aplikację testową, która integruje klasę `JsonTree` w dwupanelowym, responsywnym środowisku roboczym.

### 1. Architektura Interfejsu (Layout & Splitter)

Aplikacja została oparta na nowoczesnym layoucie Flexbox podzielonym na dwa główne obszary:

* **Panel Lewy (`.tree-side`)**: Zawiera panel wyszukiwania, przyciski sterujące oraz właściwy kontener drzewa (`#plan-container`).
* **Panel Prawy (`.info-side`)**: Pełni rolę zewnętrznego monitora zdarzeń, prezentując surowe dane JSON przechwycone przez callbacki.
* **Myszka Splitter (`.resizer`)**: Pasek rozdzielający (`#drag-bar`), który implementuje natywną obsługę zdarzeń `mousedown`/`mousemove`. Pozwala użytkownikowi na płynne, dynamiczne skalowanie szerokości paneli za pomocą przeciągania myszą, posiadając wbudowane progi bezpieczeństwa (min. 250px szerokości dla każdej ze stron).

### 2. Generowanie Danych i Formatowanie Etykiet

* **Generator Planów (`generateLargeSqlPlan`)**: Funkcja symuluje rzeczywisty proces optymalizatora bazodanowego, tworząc losowy, 40-węzłowy, ściśle **binarny plan zapytania**. Wprowadza operacje takie jak `HASH JOIN`, `NESTED LOOPS`, `TABLE ACCESS (FULL)` itp.
* **Wizualne Badges (`sqlLabelFormatter`)**: Każdy węzeł otrzymuje dynamiczną flagę kosztu (`cost-low`, `cost-medium`, `cost-high`) uzależnioną od wartości parametru kardynalności (`card`). Koszt reprezentowany jest przez kolorową kropkę statusu (zielona, pomarańczowa, czerwona) obok nazwy operacji.

### 3. Zintegrowany Panel Kontrolny

Aplikacja demonstracyjna w pełni mapuje interfejs API na elementy UI:

* **Interaktywny Licznik Wyszukiwania:** Wyświetla aktualną pozycję nawigacji w formacie `X / Y` (np. `3 / 12`). Status odświeża się dynamicznie przy zmianie frazy, kliknięciach myszką oraz nawigacji przyciskami przechodzenia (`|<`, `<`, `>`, `>|`).
* **Selektor Trybów Rozwijania:** Pozwala na bieżąco testować zachowanie drzewa (`lazy`, `isolate`, `all`) podczas filtrowania danych.
* **Przełącznik Stylu:** Przycisk *„Przełącz kółka (on/off)”* modyfikuje atrybut `circle` kontenera, demonstrując natychmiastową transformację wizualną czystego CSS.
* **Zewnętrzny Monitor:** Blokuje i formatuje zdarzenia w locie. Wyświetla dokładny identyfikator UID oraz sformatowany tekst węzła, na którym użytkownik aktualnie postawił kursor myszy lub który został kliknięty.

---

## 🤝 O projekcie i współpracy

Ten projekt powstał w formule *AI Pair Programming* podczas interaktywnej współpracy z modelem Gemini (Google AI). Architektura drzewa binarnego, kompleksowa logika wyszukiwania, zaawansowana mechanika ikon w czystym CSS oraz bezbłędna obsługa skrótów klawiszowych z modyfikatorami były wspólnie iterowane i dopracowywane w toku deweloperskiej konwersacji.

Szkielet strukturalny CSS oraz koncepcja czystego drzewa bez JavaScriptu zostały zainspirowane minimalistycznym projektem Kate Morley ([iamkate.com](https://iamkate.com/code/tree-views/)), udostępnionym w domenie publicznej na licencji CC0.

## 📝 Licencja

Projekt udostępniany na licencji MIT. Możesz go dowolnie modyfikować i wdrażać w swoich systemach monitoringu baz danych oraz analizy wydajnościowej.

```

```
