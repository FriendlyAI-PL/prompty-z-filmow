# Analiza zasobów komputera — przewodnik

Instrukcja do przeprowadzenia kompleksowej analizy systemu: co zużywa pamięć i procesor, które aplikacje można usunąć, jakie zagrożenia warto sprawdzić.

**Aplikacja:** Claude Code — aplikacja desktopowa ([claude.com/download](https://claude.com/download))  
**Cel:** diagnoza wydajności, porządkowanie systemu, wykrycie zagrożeń

> **Chat czy Claude Code?**  
> Używaj **Claude Code**, nie zwykłego chatu. Chat może tylko zasugerować komendy — musisz je wklejać ręcznie. Claude Code wykonuje je samodzielnie, widzi wyniki i kontynuuje analizę bez przerywania Tobie.

---

## Spis treści

1. [Jak uruchomić PowerShell lub CMD jako administrator](#jak-uruchomić-powershell-lub-cmd-jako-administrator)
2. [Prompt 1 — Analiza wstępna (główny)](#prompt-1--analiza-wstępna-główny)
3. [Prompt 2 — Bezpieczeństwo systemu](#prompt-2--bezpieczeństwo-systemu)
4. [Prompt 3 — Stan sprzętu](#prompt-3--stan-sprzętu)
5. [Prompt 4 — Audyt oprogramowania](#prompt-4--audyt-oprogramowania)
6. [Prompt 5 — Logi systemowe i stabilność](#prompt-5--logi-systemowe-i-stabilność)
7. [Prompt 6 — Porządki na dysku](#prompt-6--porządki-na-dysku)
8. [Uwagi](#uwagi)

---

## Jak uruchomić PowerShell jako administrator

Claude Code używa PowerShell jako domyślnego shella na Windows. Niektóre operacje wymagają uprawnień administratora — jeśli pojawi się błąd dostępu, uruchom PowerShell jako admin przed startem Claude Code.

**Metoda 1 — menu kontekstowe Start (najszybsza):**
1. Kliknij prawym przyciskiem myszy przycisk **Start** (logo Windows)
2. Wybierz **Terminal (Administrator)**
3. Kliknij **Tak** w oknie UAC

**Metoda 2 — wyszukiwarka:**
1. Naciśnij **Win**, wpisz `powershell`
2. Kliknij prawym przyciskiem na wynik
3. Wybierz **Uruchom jako administrator**

**Metoda 3 — skrót klawiszowy:**
1. Naciśnij **Win + X**
2. Wybierz **Terminal (Administrator)**

---

## Prompt 1 — Analiza wstępna (główny)

Wklej poniższy prompt na początku rozmowy z Claude Code.

```
Przeprowadź kompleksową analizę zasobów mojego komputera. Wykonaj poniższe kroki w podanej kolejności:

## 1. Rozpoznanie systemu
- Wykryj system operacyjny, wersję, architekturę i czas działania od ostatniego restartu.
- Sprawdź ilość zainstalowanej pamięci RAM, liczbę rdzeni CPU oraz model procesora.

## 2. Analiza zużycia CPU
- Wylistuj 20 procesów zużywających najwięcej CPU (posortowanych malejąco).
- Dla każdego procesu podaj: nazwę, PID, % CPU, czas działania i użytkownika, który go uruchomił.
- Sprawdź średnie obciążenie systemu (load average) z ostatnich 1, 5 i 15 minut.

## 3. Analiza zużycia RAM
- Wylistuj 20 procesów zużywających najwięcej pamięci RAM (posortowanych malejąco).
- Dla każdego procesu podaj: nazwę, PID, % RAM, zużycie w MB i użytkownika.
- Pokaż ogólne statystyki pamięci: całkowita, używana, wolna, buforowana/cache, swap (użyty vs dostępny).

## 4. Procesy w tle i usługi
- Wylistuj wszystkie usługi/demony działające w tle (np. przez systemctl, launchctl lub Menedżer zadań, w zależności od systemu).
- Oznacz te, które nie są krytyczne dla działania systemu i mogą być bezpiecznie wyłączone.
- Zwróć szczególną uwagę na: programy uruchamiane przy starcie systemu, procesy typu "zombie", zduplikowane instancje tych samych programów.

## 5. Analiza dysku i I/O
- Sprawdź zajętość dysków i partycji.
- Zidentyfikuj procesy generujące największe obciążenie dyskowe (I/O).

## 6. Analiza sieci
- Wylistuj procesy z aktywnymi połączeniami sieciowymi.
- Zidentyfikuj te, które generują największy ruch.

## 7. Raport końcowy
Na podstawie zebranych danych przygotuj podsumowanie w formie tabeli zawierającej:
- Kategorię problemu (CPU / RAM / Dysk / Sieć / Startup)
- Opis problemu
- Poziom ryzyka wyłączenia (niski / średni / wysoki)
- Konkretną rekomendację działania (komenda lub instrukcja krok po kroku)

Podziel rekomendacje na trzy grupy:
1. Szybkie wygrane - rzeczy, które można zrobić od razu i bezpiecznie.
2. Wymagające uwagi - zmiany, które przyniosą poprawę, ale trzeba je przemyśleć.
3. Do dalszej analizy - anomalie lub procesy, które wymagają głębszego zbadania.

Zanim wykonasz jakiekolwiek zmiany w systemie, zawsze pytaj o zgodę. Twoim zadaniem jest analiza i rekomendacja, nie automatyczne wyłączanie procesów.
```

---

## Prompty uzupełniające

Każdy z poniższych promptów możesz uruchomić jako osobną rozmowę po wykonaniu rekomendacji z Promptu 1.

---

### Prompt 2 — Bezpieczeństwo systemu

```
Przeprowadź analizę bezpieczeństwa mojego komputera. Wykonaj poniższe kroki:

## 1. Konta użytkowników i uprawnienia
- Wylistuj wszystkie konta użytkowników w systemie.
- Wskaż które mają uprawnienia administratora.
- Sprawdź czy są konta nieaktywne lub podejrzane (np. ukryte konta systemowe spoza standardu).

## 2. Zaplanowane zadania (Task Scheduler)
- Wylistuj wszystkie zaplanowane zadania spoza folderów Microsoft i Windows.
- Dla każdego podaj: nazwę, ścieżkę wykonywanego pliku, częstotliwość uruchamiania, autora.
- Oznacz zadania, których ścieżki plików wyglądają podejrzanie (temp, AppData, losowe nazwy).

## 3. Wpisy startowe w rejestrze
- Sprawdź klucze rejestru odpowiedzialne za autostart:
  HKCU\Software\Microsoft\Windows\CurrentVersion\Run
  HKLM\Software\Microsoft\Windows\CurrentVersion\Run
  HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
- Wylistuj wszystkie wpisy i oceń czy są standardowe.

## 4. Wyjątki w zaporze sieciowej
- Wylistuj wszystkie reguły zapory Windows Firewall zezwalające na ruch przychodzący.
- Oznacz te, które nie są standardowymi regułami systemowymi lub znanych aplikacji.

## 5. Raport końcowy
Przygotuj tabelę z potencjalnymi zagrożeniami:
- Kategoria (Konto / Task Scheduler / Rejestr / Firewall)
- Opis
- Poziom ryzyka (niski / średni / wysoki)
- Rekomendacja

Zanim wykonasz jakiekolwiek zmiany, zawsze pytaj o zgodę.
```

---

### Prompt 3 — Stan sprzętu

```
Sprawdź stan fizyczny sprzętu mojego komputera:

## 1. Temperatury
- Odczytaj temperatury CPU, GPU oraz dysków (jeśli dostępne przez WMI lub narzędzia systemowe).
- Wskaż czy któraś z wartości przekracza bezpieczne progi (CPU >90°C, dysk >55°C).

## 2. Zdrowie dysku (S.M.A.R.T.)
- Pobierz dane S.M.A.R.T. dla każdego dysku.
- Zwróć szczególną uwagę na: Reallocated Sectors, Pending Sectors, Uncorrectable Errors, Power On Hours.
- Oceń ogólny stan dysku: dobry / ostrzeżenie / krytyczny.

## 3. Zużycie GPU
- Sprawdź aktualne zużycie GPU (%), ilość użytej pamięci VRAM i temperaturę.
- Wylistuj procesy korzystające z GPU.

## 4. Raport końcowy
Podsumuj stan sprzętu w tabeli: komponent, stan, ewentualne ostrzeżenia, rekomendacja.

Zanim wykonasz jakiekolwiek zmiany, zawsze pytaj o zgodę.
```

---

### Prompt 4 — Audyt oprogramowania

```
Przeprowadź audyt zainstalowanego oprogramowania i przeglądarek:

## 1. Lista zainstalowanych programów
- Wylistuj wszystkie zainstalowane aplikacje z datą instalacji i wersją.
- Podziel na kategorie: systemowe, narzędziowe, użytkowe, nieznane/podejrzane.
- Oznacz programy, które nie były używane od ponad 6 miesięcy (jeśli ta informacja jest dostępna).
- Wskaż aplikacje z potencjalnie przestarzałymi wersjami zawierającymi znane luki bezpieczeństwa.

## 2. Aktualizacje systemu Windows
- Sprawdź datę ostatniej aktualizacji systemu.
- Wylistuj brakujące aktualizacje (jeśli dostępne przez Windows Update API).
- Wskaż czy brakuje krytycznych łatek bezpieczeństwa.

## 3. Rozszerzenia przeglądarek
- Sprawdź zainstalowane rozszerzenia w Chrome, Edge i Firefox (te, które są zainstalowane w systemie).
- Dla każdego podaj nazwę, wydawcę i uprawnienia.
- Oznacz te, które mają dostęp do wszystkich stron lub odczytują dane formularzy.

## 4. Sterowniki
- Wylistuj sterowniki urządzeń z datami i wersjami.
- Oznacz sterowniki starsze niż 2 lata lub z flagą błędu.

## 5. Raport końcowy
Tabela: program/sterownik, problem, priorytet aktualizacji lub usunięcia, sposób działania.

Zanim wykonasz jakiekolwiek zmiany, zawsze pytaj o zgodę.
```

---

### Prompt 5 — Logi systemowe i stabilność

```
Przeanalizuj logi systemowe Windows z ostatnich 30 dni:

## 1. Błędy krytyczne (Event Viewer)
- Pobierz zdarzenia poziomu Critical i Error z logów: System, Application, Security.
- Pogrupuj po źródle zdarzenia i zlicz powtórzenia.
- Wylistuj top 10 najczęściej powtarzających się błędów.

## 2. Crashe i zrzuty pamięci
- Sprawdź czy istnieją pliki minidump w C:\Windows\Minidump.
- Jeśli tak — podaj daty i kody błędów (BSOD stop codes).

## 3. Czas uruchamiania systemu
- Sprawdź logi Event ID 6005 i 6006 (start/stop systemu) — ile razy komputer był restartowany.
- Sprawdź Event ID 100 z Microsoft-Windows-Diagnostics-Performance — czas bootowania i co go spowalnia.

## 4. Błędy dysków i pamięci
- Sprawdź Event Viewer pod kątem błędów źródła: disk, nvme, volmgr, WHEA-Logger.
- Wskaż czy pojawiają się błędy sprzętowe pamięci RAM.

## 5. Raport końcowy
Tabela: rodzaj zdarzenia, częstość, powaga, rekomendacja.

Zanim wykonasz jakiekolwiek zmiany, zawsze pytaj o zgodę.
```

---

### Prompt 6 — Porządki na dysku

```
Przeanalizuj zajętość dysku i wskaż co można bezpiecznie usunąć:

## 1. Największe foldery i pliki
- Znajdź 20 największych plików na dysku C (i innych dyskach jeśli istnieją).
- Wylistuj 10 największych folderów na każdym dysku.
- Zwróć szczególną uwagę na: Downloads, Desktop, AppData\Local\Temp, C:\Windows\Temp.

## 2. Cache aplikacji
- Sprawdź rozmiar cache przeglądarek (Chrome, Edge, Firefox).
- Sprawdź rozmiar folderów tymczasowych systemu Windows.
- Sprawdź foldery cache popularnych aplikacji (Teams, Spotify, Discord, Zoom).

## 3. Pliki do bezpiecznego usunięcia
- Wskaż pliki tymczasowe (.tmp), logi (.log) starsze niż 30 dni, zrzuty pamięci (.dmp).
- Sprawdź czy folder C:\Windows\SoftwareDistribution\Download zajmuje dużo miejsca (stare pliki aktualizacji).
- Sprawdź rozmiar Kosza.

## 4. Duplikaty (jeśli możliwe do sprawdzenia)
- Sprawdź foldery Downloads i Desktop pod kątem plików o tej samej nazwie lub rozmiarze.

## 5. Raport końcowy
Tabela: lokalizacja, rozmiar, czy bezpiecznie usunąć (tak/nie/zapytaj), sposób usunięcia.

Zanim wykonasz jakiekolwiek zmiany, zawsze pytaj o zgodę.
```

---

## Uwagi

- Claude nie wykonuje żadnych zmian bez potwierdzenia — tylko analizuje i rekomenduje

---

## Autor

**Adam Kopeć** — [friendlyai.pl](https://www.friendlyai.pl/) · [YouTube](https://www.youtube.com/@Friendly_AI_PL)
