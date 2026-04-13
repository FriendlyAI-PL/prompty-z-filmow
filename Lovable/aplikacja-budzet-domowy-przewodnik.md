# Coin Sage — Jak stworzyć aplikację do zarządzania wydatkami w Lovable

Instrukcja krok po kroku, jak zbudować osobisty tracker wydatków domowych z AI w [Lovable](https://lovable.dev) — bez pisania kodu.

**Czas realizacji:** ~20–30 minut
**Wymagania:** Konto w Lovable (darmowe) — instrukcja wymaga ~15–20 kredytów łącznie (złożone prompty kosztują więcej niż 1 kredyt)
**Efekt końcowy:** Działająca aplikacja z bazą danych, analizą AI, wykresami i tabelą wydatków

> ⚠️ **Darmowy plan Lovable: 5 kredytów dziennie.** Przed każdym promptem sprawdź stan w **Settings → Workspace → Usage**. Wysłanie prompta przy niewystarczającej liczbie kredytów może przerwać generowanie w połowie i zostawić kod w niespójnym stanie.

![Coin Sage — widok główny](images/Coin%20sage%20screenshot.png)

---

## Spis treści

1. [Krok 1: Layout i design (powłoka wizualna)](#krok-1-layout-i-design-powłoka-wizualna)
2. [Krok 2: Podłączenie bazy danych (Supabase)](#krok-2-podłączenie-bazy-danych-supabase)
3. [Krok 3: Analiza wydatków przez AI + modal dodawania](#krok-3-analiza-wydatków-przez-ai--modal-dodawania)
4. [Krok 4: Nawigacja po miesiącach](#krok-4-nawigacja-po-miesiącach)
5. [Krok 5: Edycja i filtrowanie transakcji](#krok-5-edycja-i-filtrowanie-transakcji)
6. [Krok 6: Waluta i porównanie miesięczne](#krok-6-waluta-i-porównanie-miesięczne)
7. [Krok 7: Analiza AI dashboardu (opcjonalny)](#krok-7-analiza-ai-dashboardu-opcjonalny)
8. [Funkcjonalności aplikacji](#funkcjonalności-aplikacji)
9. [📋 Przykładowe dane do testowania](#przykładowe-dane-do-testowania)

---

## Krok 1: Layout i design (powłoka wizualna)

**Co robimy:** Tworzymy finalny wygląd aplikacji — kolory, fonty, układ i wszystkie sekcje w ich docelowych miejscach. Na tym etapie aplikacja wygląda jak gotowa, ale nie ma jeszcze żadnych danych ani funkcjonalności — wszystko pokazuje puste stany. Dane i logikę dodamy w kolejnych krokach.

**Co można zmienić:**

- Kolory (`#2D6A4F` → dowolny inny kolor akcentowy)
- Fonty (`Plus Jakarta Sans`, `Lora` → inne z Google Fonts)
- Nazwy i liczbę kategorii wydatków
- Nazwę aplikacji ("Coin Sage" → cokolwiek)

**Częste problemy Lovable:**

- ⚠️ Lovable może nie załadować fontów z Google Fonts — sprawdź czy fonty się wyświetlają. Jeśli nie, wyślij: `"Fonty Plus Jakarta Sans i Lora nie działają — dodaj poprawny import z Google Fonts w pliku index.html"`
- ⚠️ Przełącznik języka może nie zmieniać wszystkich tekstów — wyślij: `"Przełącznik EN/PL nie zmienia tekstu w [nazwa sekcji] — zaktualizuj tłumaczenia"`
- ⚠️ Wykres donut może być za mały — wyślij: `"Powiększ wykres donut: ustaw outerRadius na 60% i innerRadius na 38%, skalowane responsywnie"`

> 💡 **Dla widza:** W wygenerowanym kodzie pojawi się plik lub obiekt o nazwie `i18n` — to skrót od "internationalization", standardowe podejście do wielojęzyczności w aplikacjach webowych.

```
Zbuduj jednostronicową aplikację webową o nazwie "Coin Sage" —
osobisty tracker wydatków domowych. Na tym etapie tworzę tylko
wygląd i układ — bez funkcjonalności. Wszystkie sekcje pokazują
puste stany (zero danych).

SYSTEM DESIGNU
- Jasny motyw, minimalistyczna estetyka w stylu Apple
- Tło: ciepła biel (#FAFAF7), karty: #F5F3EE z subtelnym borderem #E8E2D9
- Kolor akcentowy: ciemnozielony szałwii (#2D6A4F), drugorzędny: #52B788,
  bursztynowy akcent: #D4A017, sukces: #40916C, błąd: #C1440E (terracotta)
- Font: Plus Jakarta Sans (Google Fonts) jako font ciała,
  Lora jako font nagłówków (logo, tytuły kart)
- Border radius: 20px na kartach, 12px na przyciskach/inputach
- Delikatne cienie: 0 2px 6px rgba(0,0,0,0.06)
- Responsywny, zoptymalizowany pod desktop (1280px+)
  Poniżej 1024px: jedna kolumna
- Płynne przejścia 200ms ease na elementach interaktywnych
- Bez logowania/autoryzacji

HEADER
- Lewo: "Coin Sage" (font Lora, kolor #2D6A4F) + ikona ❋ (kolor #D4A017)
- Środek: aktualny miesiąc/rok (np. "Kwiecień 2026") — statyczny tekst
- Prawo:
  Przycisk "+ Dodaj wydatki" (zielony #2D6A4F, białe litery, rounded 12px)
  — na razie nieaktywny, tylko wygląd
  Przełącznik języka EN | PL:
  Aktywny: wypełnione tło (#2D6A4F, białe litery)
  Nieaktywny: outlined (#2D6A4F border)
  Zmiana języka natychmiast zmienia WSZYSTKIE teksty UI
  Domyślny język: angielski

GŁÓWNY UKŁAD — DWIE KOLUMNY

Lewa kolumna (65% szerokości):

  Wiersz 1 — trzy karty podsumowania w jednym wierszu:
  - "Suma wydatków": duża kwota "0" (bold 32px, font Lora),
    podpis "ten miesiąc" — waluta dopasuje się do wybranego języka
  - "Główna kategoria": pusty stan "—"
  - "Transakcje": liczba "0" (bold 32px), podpis "ten miesiąc"

  Wiersz 2 — wykres słupkowy "Wydatki dzienne":
  - Pełna szerokość lewej kolumny
  - BarChart (Recharts), oś X: dni 1–30, oś Y: kwoty
  - Kolor słupków: #52B788
  - Pusty stan: "Brak danych"

Prawa kolumna (35% szerokości):

  Karta "Wydatki wg kategorii" — na pełną wysokość obu lewych wierszy:
  - Donut chart (Recharts PieChart), outerRadius 60%, innerRadius 38%,
    skalowany responsywnie
  - Kolory kategorii (paleta ziemna):
    food: #52B788, transport: #D4A017, software: #2D6A4F,
    health: #E9C46A, entertainment: #F4A261, bills: #40916C,
    shopping: #8B5E3C, uncategorized: #B7B7A4
  - Centrum wykresu: suma całkowita
  - Legenda POD wykresem: top 4 kategorii + "Inne" (reszta zgrupowana)
    Format każdego wiersza: [kropka] [nazwa] [kwota] [procent]
    Wyrównanie do lewej
  - Pusty stan: "Brak danych"
  - Min-height karty: 480px

Pod obiema kolumnami — pełna szerokość:

  Karta "Analiza AI ❋":
  - Gradientowy lewy border: #2D6A4F → #D4A017
  - Pusty stan: "Dodaj transakcje, aby zobaczyć analizę."

  Tabela transakcji:
  - Kolumny: Data | Opis | Kategoria (kolorowy badge/pill) | Kwota
  - Pusty stan: "Brak transakcji."
  - Sortowanie wg daty malejąco

INTERNACJONALIZACJA (i18n)
Zaimplementuj obsługę dwóch języków (EN/PL) przez obiekt tłumaczeń (i18n).
Przełączenie języka natychmiast aktualizuje wszystkie napisy,
etykiety, placeholdery i nazwy kategorii.
Nazwy kategorii:
  EN: Food & Dining, Transport, Software & Subs, Health,
      Entertainment, Bills, Shopping, Uncategorized, Other
  PL: Jedzenie i restauracje, Transport, Oprogramowanie,
      Zdrowie, Rozrywka, Rachunki, Zakupy, Niesklasyfikowane, Inne

Stack: React + TypeScript + Vite, Tailwind CSS, Recharts
```

---

## Krok 2: Podłączenie bazy danych (Supabase)

**Co robimy:** Dodajemy bazę danych, żeby transakcje nie znikały po odświeżeniu strony. Lovable ma wbudowaną integrację z Supabase — wystarczy jeden prompt.

**Co można zmienić:**

- Nazwy kolumn w tabeli (ale lepiej zostawić domyślne)
- Politykę RLS (zabezpieczenia) — na razie otwarta, bo to prywatna aplikacja

**Częste problemy Lovable:**

- ⚠️ Lovable może poprosić o ręczne połączenie Supabase — kliknij przycisk "Connect Supabase" w interfejsie Lovable i postępuj zgodnie z instrukcjami
- ⚠️ Mogą pojawić się ostrzeżenia o bezpieczeństwie RLS — to normalne, bo celowo ustawiamy otwartą politykę (prywatna aplikacja jednego użytkownika)
- ⚠️ Jeśli transakcje nie zapisują się po odświeżeniu, wyślij: `"Transakcje nie ładują się z bazy po odświeżeniu — sprawdź czy useEffect poprawnie wywołuje supabase.from('transactions').select('*')"`

```
Dodaj integrację z Supabase, żeby transakcje były zapisywane w bazie danych.

1. Stwórz tabelę "transactions" z kolumnami:
   - id (uuid, primary key)
   - description (text)
   - amount (numeric)
   - category (text)
   - date (date)
   - created_at (timestamp, domyślnie now())

2. Zapisuj każdą nową transakcję do Supabase po kliknięciu "Analizuj"
3. Ładuj wszystkie transakcje z Supabase przy starcie aplikacji
4. Dodaj ikonę kosza (trash) na końcu każdego wiersza tabeli —
   kliknięcie usuwa transakcję z bazy i aktualizuje wykresy/karty
5. Aplikacja jest prywatna, jeden użytkownik — bez autoryzacji,
   otwarta polityka RLS (public anon key)
```

---

## Krok 3: Analiza wydatków przez AI + modal dodawania

**Co robimy:** Aktywujemy przycisk "+ Dodaj wydatki" — otwiera modal z polem tekstowym. Wpisany tekst jest analizowany przez AI (Lovable Cloud), który rozpoznaje transakcje, przypisuje kategorie i daty. Wyniki trafiają do bazy i odświeżają dashboard.

> ⚠️ **Wymaga ukończonego kroku 2** — Supabase musi być podłączony zanim wyślemy ten prompt, bo AI po analizie od razu zapisuje transakcje do bazy.

### ⚡ Wymagane: Włączenie Lovable Cloud

`LOVABLE_API_KEY` **nie jest dostępny automatycznie** na każdym nowym projekcie — pojawia się dopiero po włączeniu Lovable Cloud.

**Sposób 1 — automatyczny (zalecany):**
Wyślij poniższy prompt — Lovable wykryje potrzebę backendu i włączy Cloud samodzielnie.

**Sposób 2 — ręczny (jeśli automatyczny nie zadziała):**

1. W górnym menu projektu kliknij ikonę **chmurki (Cloud)**
2. Przejdź do zakładki **Secrets**
3. Sprawdź czy `LOVABLE_API_KEY` istnieje na liście
4. Jeśli nie — aktywuj Lovable Cloud z tego widoku i wróć do prompta

**Po włączeniu Cloud:**

- `LOVABLE_API_KEY` pojawia się automatycznie w Secrets — Lovable zarządza nim sam, nigdy go nie wpisujesz
- Klucz daje dostęp do modeli AI (Google Gemini, GPT) przez wewnętrzny gateway Lovable
- ⚠️ Darmowy miesięczny limit — po wyczerpaniu: **Settings → Workspace → Usage**

**Co można zmienić:**

- Listę kategorii w prompcie systemowym edge function
- Placeholder w polu tekstowym modala

**Częste problemy Lovable:**

- ⚠️ Lovable może zaproponować klucz OpenAI w przeglądarce — **odmów**, poproś o Lovable Cloud
- ⚠️ Modal może nie zamykać się po dodaniu — wyślij: `"Modal nie zamyka się po Analizuj — dodaj automatyczne zamknięcie po sukcesie"`
- ⚠️ AI nie rozpoznaje polskich nazw — wyślij: `"Dodaj Biedronka, Żabka, Orlen, Lidl jako przykłady w prompcie systemowym edge function"`
- ⚠️ Błąd 429 (rate limit) — poczekaj chwilę i spróbuj ponownie

```
Aktywuj przycisk "+ Dodaj wydatki" w headerze i dodaj analizę AI:

1. Kliknięcie przycisku otwiera wycentrowany modal (backdrop: czarny 40%).
   Modal zawiera:
   - Tytuł "Dodaj wydatki"
   - Pole tekstowe (4 wiersze) z placeholderem:
     PL: "np. Biedronka 67,50 zł, Uber 12,30 zł 3 kwi, Netflix 15,99 zł"
     EN: "e.g. Biedronka $67.50, Uber $12.30 Apr 3, Netflix $15.99"
   - Przycisk "Analizuj" (pełna szerokość, kolor #2D6A4F)
   - Przycisk X w prawym górnym rogu
   - Zamknięcie przez kliknięcie poza modalem lub Escape
   - Automatyczne zamknięcie po sukcesie z toastem "✓ Dodano X transakcji"

2. Włącz Lovable Cloud i stwórz edge function "parse-transactions":
   - Odbiera tekst użytkownika i język (EN/PL)
   - Wysyła do AI przez Lovable gateway
   - Prompt systemowy: wyodrębnij transakcje jako JSON [{description,
     amount, category, date}], kategorie: food, transport, software,
     health, entertainment, bills, shopping, uncategorized.
     Jeśli brak daty — użyj dzisiejszej daty.
   - Zwraca sparsowane transakcje

3. Po analizie: zapisz transakcje do Supabase i odśwież dashboard
4. Obsługa błędów: rate limit (429), brak kredytów (402) — komunikaty toast
5. Przycisk "Analizuj" zablokowany podczas analizy (stan loading)
6. Klucz API wyłącznie po stronie serwera — nigdy w przeglądarce
```

---

## Krok 4: Nawigacja po miesiącach

**Co robimy:** Dodajemy strzałki ◀ ▶ w headerze, żeby przeglądać wydatki z poprzednich miesięcy. Wszystkie dane na dashboardzie filtrują się do wybranego miesiąca.

**Co można zmienić:**

- Czy blokować strzałkę w prawo na bieżącym miesiącu (domyślnie: tak)
- Format wyświetlania miesiąca

**Częste problemy Lovable:**

- ⚠️ Wykresy mogą nie filtrować się po zmianie miesiąca — wyślij: `"Zmiana miesiąca strzałkami nie aktualizuje wykresów — przefiltruj transakcje po selectedMonth i selectedYear przed przekazaniem do komponentów"`
- ⚠️ Wykres dzienny może pokazywać złą liczbę dni (np. 30 dni dla lutego) — wyślij: `"Wykres dzienny powinien używać selectedMonth/selectedYear do obliczenia liczby dni w miesiącu, nie new Date()"`

```
Dodaj nawigację po miesiącach do dashboardu:

1. W headerze zamień statyczny tekst miesiąca na nawigację:
   ◀  Kwiecień 2026  ▶
   - Lewa strzałka: poprzedni miesiąc
   - Prawa strzałka: następny miesiąc (zablokowana na bieżącym miesiącu)
   - Strzałki w kolorze #2D6A4F, delikatny hover effect

2. Wszystkie dane dashboardu (karty, wykresy, analiza AI, tabela)
   filtrują się do wybranego miesiąca.

3. Przycisk "+ Dodaj wydatki" dodaje transakcje z datami aktualnie
   wybranego miesiąca (jeśli tekst nie zawiera dat).

4. Miesiąc bez transakcji: puste stany na kartach i wykresach
   z komunikatem "Brak transakcji w tym miesiącu."
```

---

## Krok 5: Edycja i filtrowanie transakcji

**Co robimy:** Dodajemy możliwość edycji transakcji (opis, kwota, kategoria) oraz filtry nad tabelą (kategoria, data, kwota).

**Co można zmienić:**

- Czy dodać też edycję daty transakcji (domyślnie: nie, tylko opis/kwota/kategoria)
- Czy filtry wpływają na wykresy (domyślnie: nie — wykresy zawsze pokazują pełny miesiąc)

**Częste problemy Lovable:**

- ⚠️ Lovable może nie dodać polityki UPDATE w Supabase — jeśli edycja nie zapisuje się, wyślij: `"Dodaj politykę RLS dla UPDATE na tabeli transactions — taka sama jak dla INSERT i DELETE"`
- ⚠️ Filtry dat mogą pozwalać na wybór dat spoza aktualnego miesiąca — dodajemy ograniczenie w tym samym prompcie
- ⚠️ Po edycji transakcji wykresy mogą się nie aktualizować — sprawdź czy `handleEditSave` aktualizuje state

```
Dodaj dwie funkcje do tabeli transakcji:

1. EDYCJA TRANSAKCJI
   - Ikona ołówka (pencil) obok ikony kosza w każdym wierszu
   - Kliknięcie otwiera modal z trzema polami:
     * Opis (text input, wypełniony aktualną wartością)
     * Kwota (number input, wypełniony aktualną wartością)
     * Kategoria (dropdown ze wszystkimi kategoriami)
   - Dwa przyciski: "Zapisz" (#2D6A4F) i "Anuluj"
   - Zapis aktualizuje transakcję w Supabase i odświeża wykresy/karty
   - Styl modala spójny z istniejącym modalem "Dodaj wydatki"

2. FILTRY NAD TABELĄ
   Trzy filtry w jednym wierszu nad tabelą:
   - Dropdown kategorii: "Wszystkie kategorie" + lista użytych kategorii
   - Zakres dat: dwa date pickery "Od" i "Do"
     (domyślnie: pierwszy i ostatni dzień aktualnie wybranego miesiąca)
     Zablokowane do zakresu aktualnego miesiąca — nie da się wybrać
     dat spoza wybranego miesiąca
   - Zakres kwot: dwa pola "Min" i "Max"
   - Link "Wyczyść filtry" resetujący do domyślnych
   Filtry działają natychmiast (bez dodatkowego przycisku).
   Filtry NIE wpływają na wykresy ani karty — te zawsze pokazują pełny miesiąc.
   Zmiana miesiąca strzałkami ◀ ▶ automatycznie resetuje filtry.
```

---

## Krok 6: Waluta i porównanie miesięczne

**Co robimy:** Dodajemy dwie ostatnie funkcje — automatyczną zmianę waluty przy przełączaniu języka (EN → $, PL → zł) oraz prawdziwe obliczenie porównania wydatków z poprzednim miesiącem.

**Co można zmienić:**

- Dodatkowe waluty (np. EUR)
- Separator dziesiętny (EN: kropka, PL: przecinek)
- Precyzja wyświetlania (2 miejsca po przecinku vs zaokrąglenie)

**Częste problemy Lovable:**

- ⚠️ Symbol waluty może nie zmienić się na wykresach — wyślij: `"Symbol waluty nie zmienił się na osi Y wykresu słupkowego i w środku donuta — użyj t.formatAmountShort() zamiast hardcoded $"`
- ⚠️ Badge porównania może pokazywać złe dane — sprawdź czy pobiera dane z poprzedniego miesiąca z Supabase, nie z lokalnego state
- ⚠️ Parser AI może nie rozpoznawać kwot z "zł" — upewnij się, że edge function akceptuje oba formaty

```
Zmiana 1 — Waluta powiązana z językiem:

Przełączenie języka na PL zmienia symbol waluty z $ na zł.
Przełączenie na EN cofa do $.

1. Wszystkie wyświetlane kwoty używają symbolu waluty:
   EN → $ (prefix, np. $45.00)
   PL → zł (suffix, np. 45,00 zł) z przecinkiem jako separator dziesiętny
2. Placeholder w polu tekstowym:
   PL: "np. Wydałem 45 zł w Biedronce, 12,50 zł za Ubera, 299 zł za subskrypcję Figma"
   EN: istniejący angielski placeholder
3. Symbol waluty zmienia się WSZĘDZIE:
   karty podsumowania, środek donuta, legenda, oś Y wykresu, tabela, analiza AI
4. Parser tekstu akceptuje zarówno . jak i , jako separator dziesiętny
   niezależnie od wybranego języka

Zmiana 2 — Prawdziwe porównanie z poprzednim miesiącem:

Dodaj badge z porównaniem do poprzedniego miesiąca w karcie "Suma wydatków":

1. Pobierz sumę wydatków dla aktualnie wybranego miesiąca
2. Pobierz sumę wydatków z poprzedniego miesiąca z Supabase
3. Oblicz różnicę procentową:
   ((aktualny - poprzedni) / poprzedni) * 100, zaokrąglenie do 1 miejsca

4. Reguły wyświetlania:
   - Wydatki WZROSŁY: "↑ X% vs zeszły miesiąc" — kolor czerwony (#C1440E)
   - Wydatki SPADŁY: "↓ X% vs zeszły miesiąc" — kolor zielony (#40916C)
   - Identyczne: "= 0% vs zeszły miesiąc" — szary
   - Brak danych z poprzedniego miesiąca: ukryj badge całkowicie

5. Badge aktualizuje się automatycznie przy:
   - Nawigacji do innego miesiąca
   - Dodaniu lub usunięciu transakcji
```

---

## Krok 7: Analiza AI dashboardu (opcjonalny)

**Co robimy:** Wypełniamy kartę "Analiza AI ❋" na dashboardzie — po dodaniu transakcji AI automatycznie generuje krótkie wnioski o wydatkach danego miesiąca.

> 💡 **Krok opcjonalny** — karta istnieje od Kroku 1, ale pozostaje pusta bez tego prompta. Możesz go pominąć, jeśli chcesz zaoszczędzić kredyty.

**Co można zmienić:**

- Liczbę i charakter spostrzeżeń w prompcie systemowym
- Maksymalną długość odpowiedzi (domyślnie: 120 słów)

**Częste problemy Lovable:**

- ⚠️ Karta może pozostać pusta po dodaniu transakcji — sprawdź czy `analyze-spending` jest wywoływana w `useEffect` po zmianie listy transakcji lub wybranego miesiąca
- ⚠️ Analiza może się nie odświeżać po zmianie miesiąca strzałkami — upewnij się, że `selectedMonth` i `selectedYear` są w tablicy zależności `useEffect`

```
Wypełnij kartę "Analiza AI ❋" na dashboardzie:

1. Stwórz edge function "analyze-spending":
   - Wywoływana automatycznie po załadowaniu transakcji i po każdej zmianie
     (dodanie, usunięcie transakcji, zmiana miesiąca) dla aktualnie wybranego miesiąca
   - Przyjmuje: listę transakcji [{description, amount, category, date}] + język (EN/PL)
   - Jeśli lista jest pusta — nie wywołuje funkcji, karta zostaje w pustym stanie
   - Prompt systemowy do AI: "Przeanalizuj poniższe wydatki i napisz 3–4 konkretne
     spostrzeżenia w języku [EN/PL]: która kategoria dominuje, czy widać
     nieoczekiwane wydatki, jedna praktyczna sugestia oszczędności.
     Odpowiedź w formacie markdown, max 120 słów."
   - Klucz API wyłącznie po stronie serwera — użyj istniejącego Lovable Cloud

2. Wynik wyświetl w karcie "Analiza AI ❋" jako sformatowany markdown
3. Podczas generowania: skeleton loader (animowane linie) zamiast pustego stanu
4. Po błędzie: "Analiza niedostępna — spróbuj ponownie." (bez ikony crash)
```

---

## Funkcjonalności aplikacji

### Co mamy po 7 krokach:

| Funkcja        | Opis                                                   |
| -------------- | ------------------------------------------------------ |
| 🎨 Design      | Minimalistyczny, responsywny interfejs z ciepłą paletą |
| 💾 Baza danych | Supabase — transakcje nie znikają po odświeżeniu       |
| 🤖 AI          | Lovable Cloud analizuje tekst i kategoryzuje wydatki   |
| 📊 Wykresy     | Donut (kategorie) + słupkowy (wydatki dzienne)         |
| 📅 Miesiące    | Nawigacja ◀ ▶ z filtrowaniem danych                    |
| ✏️ Edycja      | Edycja i usuwanie transakcji                           |
| 🔍 Filtry      | Filtrowanie tabeli po kategorii, dacie, kwocie         |
| 💱 Waluta      | Automatyczna zmiana $ ↔ zł przy zmianie języka         |
| 📈 Porównanie  | Prawdziwy % zmiany vs poprzedni miesiąc                |
| 💡 Analiza AI  | Automatyczne wnioski o wydatkach miesiąca (opcjonalne) |

### 📋 Przykładowe dane do testowania

Skopiuj i wklej do pola tekstowego w modalu "+ Dodaj wydatki". Zacznij od **lutego**, żeby porównanie miesięczne działało poprawnie od razu.

**Kwiecień:**

```
Biedronka 67,50 zł 1 kwi, Żabka 8,40 zł 2 kwi, Uber 12,30 zł 3 kwi,
Spotify 9,99 zł 3 kwi, Orlen 58 zł 5 kwi, Netflix 15,99 zł 6 kwi,
Empik 27,90 zł 7 kwi, Apteka Dbam o Zdrowie 34,50 zł 8 kwi,
OLX dostawa 6,50 zł 9 kwi, Allegro 43,20 zł 9 kwi,
McDonald's 22,80 zł 10 kwi, Siłownia 49 zł 11 kwi,
Rossmann 31,60 zł 12 kwi, Play 45 zł 13 kwi,
Prąd 119 zł 14 kwi, H&M 89 zł 15 kwi,
Lidl 54,30 zł 17 kwi, Booking.com 210 zł 19 kwi
```

**Marzec:**

```
Żabka 6,20 zł 2 mar, Biedronka 71,40 zł 3 mar, Netflix 15,99 zł 5 mar,
Allegro 38,50 zł 6 mar, Orlen 54 zł 8 mar, Siłownia 49 zł 9 mar,
Rossmann 28,30 zł 10 mar, Uber 9,80 zł 11 mar,
McDonald's 31,50 zł 13 mar, Play 45 zł 14 mar,
Lidl 62,10 zł 15 mar, Empik 19,90 zł 16 mar,
Spotify 9,99 zł 17 mar, Apteka Dbam o Zdrowie 41,20 zł 19 mar,
Prąd 108 zł 20 mar, H&M 67 zł 22 mar,
Amazon 54,30 zł 25 mar, Ikea 129 zł 27 mar
```

**Luty:**

```
Biedronka 58,90 zł 1 lut, Żabka 11,30 zł 3 lut, Orlen 61 zł 4 lut,
Netflix 15,99 zł 5 lut, Spotify 9,99 zł 5 lut, Uber 14,20 zł 7 lut,
Siłownia 49 zł 8 lut, Rossmann 33,60 zł 10 lut, Play 45 zł 12 lut,
Lidl 49,80 zł 13 lut, McDonald's 18,90 zł 14 lut,
Apteka Dbam o Zdrowie 27,50 zł 16 lut, Empik 44 zł 18 lut,
Prąd 97 zł 20 lut, Allegro 76,20 zł 22 lut,
CCC 89 zł 24 lut, Żabka 8,40 zł 26 lut, Ikea 210 zł 27 lut
```

### Wskazówki ogólne

- **Testuj po każdym kroku** — nie wysyłaj następnego prompta zanim nie sprawdzisz czy poprzedni działa
- **Lovable rozumie polski** — możesz pisać prompty w obu językach
- **Jeśli Lovable zepsuje coś co działało** — użyj przycisku "Undo" w interfejsie Lovable
- **Zrzuty ekranu pomagają** — jeśli coś wygląda źle, zrób screenshot i załącz go do następnego prompta
- **Mniej = więcej** — krótsze, konkretne prompty działają lepiej niż długie opisy

---

## Autor

**Adam Kopeć** — [friendlyai.pl](https://www.friendlyai.pl/) · [YouTube](https://www.youtube.com/@Friendly_AI_PL)
