# Skalowalne Binarne Drzewo Planu Zapytania SQL (JsonTree)

Język: [English (Angielski)](README.md) | **Polski**

---

Interaktywny, wysokowydajny komponent drzewiastego planu wykonania zapytania SQL bazy danych (np. Oracle/PostgreSQL Explain Plan). Komponent generuje i renderuje strukturę binarną (maksymalnie dwa węzły podrzędne dla każdego operatora, np. dla złączeń `HASH JOIN` czy `NESTED LOOPS`), symulując rzeczywisty plan wykonania składający się z **wielu złożonych operacji**.

Projekt został zaimplementowany w czystym technologicznym stosie **Vanilla HTML5, CSS3 oraz ECMAScript 6 (w architekturze zorientowanej obiektowo)**, bez użycia jakichkolwiek zewnętrznych bibliotek czy zależności.

## 🚀 Główne Funkcjonalności

* **Prawdziwa Struktura Binarna:** Algorytm generujący dba o to, aby żaden węzeł (poza liśćmi) nie posiadał więcej niż 2 dzieci, odzwierciedlając faktyczną logikę wykonania bazodanowych złączeń binarnych.
* **Nowoczesna Architektura Klasowa:** Całość logiki komponentu została zamknięta w autonomicznej klasie `JsonTree` z wykorzystaniem prywatnych pól i metod (`#`), co gwarantuje pełną enkapsulację kodu i zapobiega wyciekom do globalnego zakresu (*scope*).
* **Pełna Nawigacja Klawiszowa (Strzałki):** Komponent umożliwia intuicyjne sterowanie drzewem bezpośrednio z klawiatury:
* `Strzałka w Górę` / `W Dół` — płynne przechodzenie i aktywacja kolejnych widocznych gałęzi drzewa wraz z automatycznym centrowaniem widoku (*scroll into view*).
* `Strzałka w Prawo` — dynamiczne rozwinięcie aktualnie wybranej gałęzi.
* `Strzałka w Lewo` — zwinięcie aktywnej gałęzi, a w przypadku gdy jest już zwinięta — natychmiastowe przeniesienie zaznaczenia na węzeł nadrzędny (rodzica).


* **Zaawansowane Sterowanie Poddrzewem (Skrót Ctrl):**
* *Kliknięcie standardowe:* Rozwija lub zwija wyłącznie kliknięty węzeł nadrzędny.
* *Kliknięcie z przytrzymanym klawiszem **Ctrl**:* Rekurencyjnie rozwija lub zwija **całe poddrzewo wraz ze wszystkimi jego głębokimi podgałęziami**.


* **Zewnętrzne API i Monitor Zdarzeń (Callbacks):** Klasa udostępnia publiczne metody pozwalające na rejestrację callbacków zewnętrznych (`onNodeClick`, `onNodeHover`). Umożliwia to łatwe przesyłanie danych o wybranym węźle do zewnętrznych pulpitów menedżerskich i monitorów zdarzeń bez modyfikacji rdzenia komponentu.
* **Tryb Minimalistyczny (`circle="off"`):** Komponent obsługuje dynamiczny mechanizm przełączania stylistyki za pomocą atrybutu `circle`. Po ustawieniu `circle="off"`, dynamiczne, zielone kółka akcji na hoverze zostają całkowicie zablokowane na rzecz eleganckich ikon geometrycznych, zachowując przy tym rygorystyczną blokadę skalowania (stały rozmiar 6px) dla ikony korzenia.
* **Inteligentny Interfejs Ikoniczny (CSS Pseudoelements):**
* **Stan zamknięty:** Ikona reprezentowana jest przez minimalistyczny trójkąt skierowany w prawo, który jest pusty (biały) w środku z ciemnoszarą ramką.
* **Stan otwarty:** Płynna animacja obrotu trójkąta o 90° w dół wraz z automatycznym zapełnieniem kształtu kolorem.
* **Wyjątek Korzenia:** Główny węzeł bazy danych (ID: 0, `SELECT STATEMENT`) posiada unikalną, statyczną ikonę czarnego kwadratu, symbolizującą punkt startowy/końcowy zapytania.



## 🛠️ Architektura i Struktura Kodu

Komponent zamknięty jest w jednym, samowystarczalnym pliku HTML i dzieli się na trzy logiczne warstwy:

### 1. Warstwa Strukturalna (HTML)

Używa semantycznych tagów HTML5: `<ul class="json-tree">` jako kontenera głównego, zestawów list `<li>` oraz natywnych elementów kontrolnych `<details>` i `<summary>`, co gwarantuje pełną dostępność (*accessibility*).

### 2. Warstwa Prezentacji (CSS3)

* Zmienne CSS (`--spacing`, `--radius`, `--accent`) pozwalają na błyskawiczny branding i dopasowanie kolorystyczne komponentu.
* Wszystkie ikony (trójkąty, kwadraty) są renderowane jako wektorowe grafiki inline **SVG (Data URI)** wewnątrz pseudoelementów `::before`.
* Precyzyjne kaskady CSS (w tym połączenia specyficzności selektorów z `!important`) odpowiadają za perfekcyjne zachowanie proporcji ikon w trybie `circle="off"`.

### 3. Warstwa Logiki (Klasa `JsonTree` i skrypty uruchomieniowe)

* `class JsonTree`: Serce aplikacji. Przyjmuje konfigurację wejściową, automatycznie mapuje płaskie kolekcje danych, buduje strukturę i inicjalizuje nasłuchiwanie zdarzeń myszy i klawiatury.
* `getAPI()`: Publiczny interfejs klasy eksponujący metody sterujące (np. `expandAll`, `collapseAll`, `goUp`, `goDown`, `goLeft`, `goRight`) oraz rejestratory zdarzeń.
* `generateLargeSqlPlan(count)`: Zewnętrzny generator losujący realistyczne kroki planu SQL (`HASH JOIN`, `INDEX RANGE SCAN`, itp.) dbający o rygorystyczne zachowanie reguł drzewa binarnego na potrzeby prezentacji.

## 🤝 O projekcie i współpracy

Ten projekt powstał w formule *AI Pair Programming* podczas interaktywnej współpracy z modelem Gemini (Google AI). Architektura drzewa binarnego, zaawansowana logika ikon w czystym CSS oraz obsługa skrótów klawiszowych z modyfikatorem były wspólnie iterowane i dopracowywane w toku deweloperskiej konwersacji.

Szkielet strukturalny CSS oraz koncepcja czystego drzewa bez JavaScriptu zostały zainspirowane minimalistycznym projektem Kate Morley ([iamkate.com](https://iamkate.com/code/tree-views/)), udostępnionym w domenie publicznej na licencji CC0.

## 💻 Jak Uruchomić Projekt

1. Sklonuj to repozytorium lub pobierz plik z kodem źródłowym.
2. Otwórz plik `index.html` bezpośrednio w dowolnej nowoczesnej przeglądarce internetowej (Chrome, Firefox, Safari, Edge).
3. Kliknij dowolny element w drzewie, aby go uaktywnić, a następnie użyj **strzałek na klawiaturze**, aby przetestować zaawansowaną nawigację.
4. Użyj panelu bocznego, aby obserwować działanie zewnętrznego Monitora Zdarzeń zasilanego przez API callbacków.

## 📝 Licencja

Projekt udostępniany na licencji MIT. Możesz go dowolnie modyfikować i wdrażać w swoich systemach monitoringu baz danych oraz analizy wydajnościowej.
