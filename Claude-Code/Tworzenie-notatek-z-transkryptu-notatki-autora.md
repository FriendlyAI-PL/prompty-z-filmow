# Notatki z wideo — wersja rozszerzona (notatki autora)

Poniżej znajduje się kompletny zestaw promptów i instrukcji pozwalający odtworzyć aplikację **"Notatki z wideo"** od zera w Claude Code. Aplikacja zamienia plik wideo/audio w sformatowany dokument Word z notatkami.

Pipeline: **plik audio/wideo** → Whisper (transkrypt) → Claude API (notatka .md) → python-docx (.docx)

> **Ta wersja pliku** różni się od podstawowej dwoma rzeczami: Krok 2 ma dodatkowy Wariant A (z plikami szablonów, nie pokazany w filmie), a na końcu znajdziesz prompty wysłane podczas sesji nagraniowej do naprawienia błędów aplikacji.

---

## Krok 0: Przygotowanie środowiska

> **Wymagania wstępne (ręcznie, przed otwarciem Claude Code):**
> 
> 1. **Python 3.10+** — https://www.python.org/downloads/ (zaznacz "Add to PATH")
> 2. **Klucz API Anthropic** — https://console.anthropic.com/settings/keys
> 3. Utwórz pusty folder dla projektu, np. `C:\Projekty\Notatki z wideo\`
> 4. Otwórz ten folder w Claude Code

Następnie wklej poniższy prompt w Claude Code:

### Prompt

```
Sprawdź środowisko i przygotuj projekt:

1. Sprawdź czy Python jest zainstalowany (py -V lub python --version) i podaj wersję.

2. Sprawdź czy ffmpeg jest dostępny na PATH (ffmpeg -version). Jeśli nie — wyświetl instrukcję:
   "Zainstaluj ffmpeg komendą: winget install Gyan.FFmpeg"
   Nie przerywaj pracy — ffmpeg zostanie znaleziony dynamicznie przez aplikację nawet jeśli nie jest na PATH.

3. Zainstaluj wymagane pakiety Python:
   pip install flask anthropic openai-whisper python-docx python-dotenv

4. Stwórz plik .env z placeholderem klucza API:
   ANTHROPIC_API_KEY=sk-ant-WKLEJ-SWOJ-KLUCZ

5. Stwórz pusty folder output/

6. Wypisz podsumowanie: co jest gotowe, co wymaga uwagi.
```

### Notatka

> FFmpeg to najczęstszy punkt awarii. Whisper wymaga go do odczytu plików audio/wideo. Jeśli ffmpeg nie jest na PATH (a po `winget install` często nie jest od razu), aplikacja szuka go dynamicznie w typowych lokalizacjach Windows — to jest uwzględnione w prompcie do kroku 3.
> 
> Instalacja pakietów (`openai-whisper`) może trwać 2-3 minuty — Whisper pobiera model przy pierwszym użyciu (~140 MB dla `base`).

---

## Krok 1: CLAUDE.md i struktura projektu

### Prompt

```
Tworzę aplikację "Notatki z wideo" — lokalne narzędzie konwertujące filmy/audio w sformatowane notatki Word.

Stwórz plik CLAUDE.md z kontekstem projektu oraz plik requirements.txt, a następnie zainstaluj zależności (pip install -r requirements.txt).

Opis projektu:
- Pipeline: plik audio/wideo → transkrypt (Whisper lokalnie) → notatka Markdown (Claude API) → dokument .docx (python-docx)
- Użytkownik może też wgrać gotowy .txt z transkryptem (wtedy Whisper jest pomijany)
- Serwer: Flask na porcie 5001, interfejs w przeglądarce
- Komunikacja real-time: Server-Sent Events (SSE) z keepalive ping co 30s
- Język interfejsu i notatek: polski
- Konwencja nazw plików: {nazwa-pliku}_{RRMMDD_HHMMSS}_notes.docx (timestamp na końcu)

Struktura katalogów:
- pipeline/ — moduły Python (transcribe.py, generate_notes.py, md_to_docx.py)
- static/ — pliki HTML
- output/ — generowane pliki (tworzony automatycznie)

Zależności (requirements.txt):
- flask>=3.0
- anthropic>=0.25
- openai-whisper>=20231117
- python-docx>=1.1
- python-dotenv>=1.0

Plik .env (stwórz z placeholderem):
ANTHROPIC_API_KEY=sk-ant-WKLEJ-SWOJ-KLUCZ

Motywy kolorystyczne dokumentu .docx (do wyboru przez użytkownika):
- Niebieski (domyślny): dark=#1F4E79, medium=#2E75B6, light=#D6E4F0
- Zielony: dark=#1B5E20, medium=#388E3C, light=#C8E6C9
- Malinowy: dark=#7B1F3A, medium=#C2185B, light=#F8BBD0
```

### Notatka

> Po tym kroku mamy CLAUDE.md, requirements.txt, .env i zainstalowane pakiety. CLAUDE.md zapewnia kontekst — w kolejnych promptach Claude Code "wie" o czym jest projekt.
> 
> Uczestnik musi teraz otworzyć `.env` i wkleić swój klucz API.

---

## Krok 2: Konwerter Markdown → Word (.docx)

Poniżej dwa warianty tego samego kroku. **Wariant A** wymaga przygotowanych plików wzorcowych — daje lepsze i bardziej przewidywalne rezultaty. **Wariant B** jest samodzielny i nie wymaga niczego dodatkowego — Claude buduje styl od zera na podstawie opisu słownego.

### Wariant A — z plikami szablonów

> **Wymaga:** Folderu `Template notatki/` z przykładowymi plikami .docx jako wzorzec stylu. Skopiuj go do folderu projektu przed uruchomieniem promptu.

```
Przeanalizuj pliki .docx w folderze "Template notatki/" — to wzorce stylu docelowych notatek.

Na ich podstawie stwórz plik pipeline/md_to_docx.py — konwerter Markdown → .docx z biblioteką python-docx.

Wymagania:
1. Funkcja convert(markdown_text, output_path, theme="niebieski") parsuje Markdown i tworzy .docx
2. Trzy motywy kolorystyczne: niebieski, zielony, malinowy (kolory w CLAUDE.md)
3. Każdy motyw to słownik z kluczami: dark, medium, light, very_light, curiosity_bg

Elementy do obsługi:
- Blok tytułowy: 2 wyśrodkowane wiersze na tle dark (nazwa kursu + tytuł lekcji) + "Notatki i ściągawka"
- # Heading 1 → biały tekst na tle dark
- ## Heading 2 → tekst w kolorze medium
- **Śródtytuł:** (pogrubiony akapit kończący się dwukropkiem)
- Listy punktowe (- tekst) i numerowane (1. tekst)
- Bloki kodu (```) → Courier New, tło very_light, BEZ odstępów między liniami kodu (space_before=0, space_after=0)
- Blockquote z emoji:
  - > 🧭 → wskazówka (tło #FFF9E6)
  - > 🔍 → ciekawostka (tło curiosity_bg)
  - > ⚠️ → ostrzeżenie (tło #FFF0E6)
  - > "cytat" → cytat (tło very_light, wcięcie)
- Tabele Markdown → Table Grid z kolorowanymi nagłówkami
- *legenda* (kursywa całego wiersza) → szary tekst

Kluczowe detale techniczne:
- Emoji (🧭🔍⚠️) renderuj z czcionką "Segoe UI Emoji" — bez tego Word pokaże je czarno-białe
- Stwórz funkcję _apply_inline(para, text) parsującą **bold**, *italic* i `code` w tekście inline. Użyj regex: r'(\*\*(.+?)\*\*|\*([^*]+?)\*|`(.+?)`)'
  - `code` → Courier New, bold, kolor dark
  - Używaj _apply_inline w: bullet, normal, tip, warning, curiosity i komórkach tabel (NIE w nagłówkach)
- Marginesy strony: 2cm boki, 1.5cm góra/dół

Stwórz też pipeline/__init__.py (pusty).
```

### Wariant B — bez szablonów, opis słowny

```
Stwórz plik pipeline/md_to_docx.py — konwerter Markdown → .docx z biblioteką python-docx.

Styl dokumentu: profesjonalny, kolorowy, z ikonami emoji. Wygląd inspirowany materiałami kursowymi — czytelne nagłówki na kolorowym tle, ramki informacyjne, tabele z kolorowanymi nagłówkami.

Wymagania:
1. Funkcja convert(markdown_text, output_path, theme="niebieski") parsuje Markdown i tworzy .docx
2. Trzy motywy kolorystyczne: niebieski, zielony, malinowy (kolory w CLAUDE.md)
3. Każdy motyw to słownik z kluczami: dark, medium, light, very_light, curiosity_bg

Elementy do obsługi:
- Blok tytułowy: 2 wyśrodkowane wiersze na tle dark (nazwa kursu + tytuł lekcji) + "Notatki i ściągawka"
- # Heading 1 → biały tekst na tle dark
- ## Heading 2 → tekst w kolorze medium
- **Śródtytuł:** (pogrubiony akapit kończący się dwukropkiem)
- Listy punktowe (- tekst) i numerowane (1. tekst)
- Bloki kodu (```) → Courier New, tło very_light, BEZ odstępów między liniami kodu (space_before=0, space_after=0)
- Blockquote z emoji:
  - > 🧭 → wskazówka (tło #FFF9E6)
  - > 🔍 → ciekawostka (tło curiosity_bg)
  - > ⚠️ → ostrzeżenie (tło #FFF0E6)
  - > "cytat" → cytat (tło very_light, wcięcie)
- Tabele Markdown → Table Grid z kolorowanymi nagłówkami
- *legenda* (kursywa całego wiersza) → szary tekst

Kluczowe detale techniczne:
- Emoji (🧭🔍⚠️) renderuj z czcionką "Segoe UI Emoji" — bez tego Word pokaże je czarno-białe
- Stwórz funkcję _apply_inline(para, text) parsującą **bold**, *italic* i `code` w tekście inline. Użyj regex: r'(\*\*(.+?)\*\*|\*([^*]+?)\*|`(.+?)`)'
  - `code` → Courier New, bold, kolor dark
  - Używaj _apply_inline w: bullet, normal, tip, warning, curiosity i komórkach tabel (NIE w nagłówkach)
- Marginesy strony: 2cm boki, 1.5cm góra/dół

Stwórz też pipeline/__init__.py (pusty).
```

### Notatka

> Kluczowe pułapki, które prompt omija:
> 
> - **Emoji czarno-białe w Wordzie** — `Segoe UI Emoji` rozwiązuje problem
> - **Backticki i gwiazdki w tekście** — `_apply_inline` z pełnym regex od razu
> - **Linie kodu rozsunięte** — `space_before=0, space_after=0` eliminuje przerwy
> - **Backticki w tabelach** — wymuszamy `_apply_inline` w komórkach danych

---

## Krok 3: Pipeline — transkrypcja i generowanie notatek

### Prompt

```
twórz dwa moduły pipeline:

### 1. pipeline/transcribe.py

Transkrypcja audio/wideo za pomocą openai-whisper (lokalnie).

Funkcja główna: transcribe_with_progress(input_path, model_name="base", language="pl", progress_cb=None)
- progress_cb(message) — callback do raportowania postępu (SSE)
- Jeśli plik to .txt lub .md → wczytaj tekst, pomiń Whisper
- Zwraca string z transkryptem

WAŻNE — obsługa ffmpeg na Windows:
Whisper wymaga ffmpeg, ale po instalacji przez `winget install` nie trafia on na PATH. Musisz:
1. Stworzyć funkcję _find_ffmpeg() szukającą ffmpeg.exe:
   - Najpierw shutil.which("ffmpeg") (PATH)
   - Potem glob.glob po typowych ścieżkach Windows:
     - C:\Users\*\AppData\Local\Microsoft\WinGet\Packages\Gyan.FFmpeg*\**\bin\ffmpeg.exe
     - C:\ProgramData\chocolatey\bin\ffmpeg.exe
     - C:\ffmpeg\bin\ffmpeg.exe
     - C:\Program Files\ffmpeg\bin\ffmpeg.exe
   - Jeśli nie znaleziono → FileNotFoundError z instrukcją "winget install Gyan.FFmpeg"

2. Stworzyć funkcję _audio_to_numpy(input_path, ffmpeg_exe) konwertującą audio na numpy array:
   - subprocess.run z pełną ścieżką do ffmpeg (nie "ffmpeg" — bo nie jest na PATH!)
   - Parametry: -f s16le -ac 1 -acodec pcm_s16le -ar 16000 -
   - Konwersja: np.frombuffer(stdout, dtype=np.int16).astype(np.float32) / 32768.0

3. Wynik _audio_to_numpy podaj do model.transcribe(audio, ...) — Whisper nie używa ffmpeg sam

### 2. pipeline/generate_notes.py

Generowanie notatek Markdown z transkryptu przez Claude API.

Funkcja główna: generate_notes(transcript, course_name="", lesson_title="", model="claude-sonnet-4-6", progress_cb=None)
- progress_cb(message, chunk=None) — callback: message do logów, chunk do streamowania tekstu
- Użyj anthropic.Anthropic() (klucz z .env)
- Streaming: client.messages.stream() z iteracją po stream.text_stream
- max_tokens=8192

System prompt (wklej dosłownie):
Jesteś ekspertem w tworzeniu zwięzłych, dobrze zorganizowanych notatek edukacyjnych z transkryptów filmów wideo (kursów, wykładów, tutoriali).

Twoim zadaniem jest przeanalizowanie transkryptu i stworzenie notatki w formacie Markdown zgodnie z poniższymi zasadami formatowania.

ZASADY FORMATOWANIA

### Blok tytułowy (zawsze na początku)

Pierwsze dwa wiersze to metadane – BEZ nagłówka Markdown:
NAZWA KURSU / NARZĘDZIA
NR.NR - Tytuł lekcji

### Nagłówki sekcji

- # 1. Tytuł sekcji – główne tematy (Heading 1)
- ## Podtytuł – podsekcje (Heading 2), tylko gdy sekcja ma wyraźne podgrupy
  
### Śródtytuły (przed listami)

**Tytuł podpunktu:** – pogrubiony akapit poprzedzający listę

### Listy punktowe

- **Kluczowy termin** – wyjaśnienie lub opis
- Punkt bez wyróżnienia gdy nie ma terminu do wyróżnienia

### Listy numerowane (tylko gdy kolejność ma znaczenie)

1. **Krok pierwszy** – opis

### Bloki kodu / formuły

Standardowy blok kodu Markdown z trzema backtickami.

### Ramki informacyjne

- > 🧭 tekst – wskazówka praktyczna
- > 🔍 tekst – ciekawostka
- > ⚠️ tekst – ostrzeżenie
- > "cytat" – dosłowny cytat lub definicja
  

### Tabele

Standardowe tabele Markdown (2 lub 3 kolumny).

### Ściągawka (zawsze na końcu)

Ostatnia sekcja zbierająca najważniejsze skróty i pojęcia w tabeli.

## ZASADY OGÓLNE

- Pisz po polsku
- Bądź zwięzły – notatka ma być ściągawką, nie przepisanym transkryptem
- Wyróżniaj TYLKO naprawdę kluczowe terminy pogrubieniem
- Numeruj sekcje kolejno: 1, 2, 3…

User message: opcjonalne metadane (kurs, lekcja) + "Transkrypt do opracowania:\n\n" + transcript

```

### Notatka

> To najdłuższy prompt, ale każdy element jest tam z konkretnego powodu:
> 
> - **_find_ffmpeg + _audio_to_numpy**: Bez tego Whisper rzuci `[WinError 2] Nie można odnaleźć pliku` — był to największy bloker w oryginalnym procesie. Whisper wewnętrznie wywołuje `ffmpeg` z PATH, a po `winget install` ffmpeg trafia do `AppData\Local\Microsoft\WinGet\Packages\...`
> - **System prompt**: Definiuje format Markdown, który potem parser w md_to_docx.py potrafi zamienić na .docx. Bez tego Claude generuje dowolny Markdown, który nie pasuje do parsera.
> - **Streaming**: Dzięki `client.messages.stream()` użytkownik widzi postęp generowania w przeglądarce.

---

## Krok 4: Serwer Flask + interfejs HTML

### Prompt

```
Stwórz serwer Flask (app.py) i interfejs HTML (static/index.html) łączący cały pipeline.

### app.py

Endpointy:

- GET / → serwuje static/index.html
- POST /api/upload → przyjmuje plik, zapisuje w output/, zwraca job_id i stem
- GET /api/transcribe/<job_id>?model=base&language=pl → SSE transkrypcji
- PUT /api/transcript/<job_id> → aktualizacja transkryptu (JSON: {transcript: "..."})
- GET /api/generate/<job_id>?course=...&lesson=... → SSE generowania notatek
- PUT /api/notes/<job_id> → aktualizacja notatki
- POST /api/convert/<job_id> → konwersja do .docx (JSON: {theme: "niebieski"})
- GET /api/download/<filename> → pobieranie pliku

Kluczowe wymagania:

- Port 5001 (nie 5000 — możliwy konflikt)
- Nazwy plików: {stem}_{RRMMDD_HHMMSS}_notes.docx (timestamp na końcu stem)
- SSE: użyj queue.Queue w wątku + generator z yield. Timeout 30s z keepalive "": ping\n\n" (NIE 120s — transkrypcja modelem medium trwa ~4 minuty, połączenie SSE zostanie zerwane)
- Wątki: threading.Thread(daemon=True) dla transkrypcji i generowania
- Zabezpieczenie download: Path(filename).name zapobiega path traversal

### static/index.html

Jednostronicowy interfejs — 4-krokowy wizard:

1. **Plik** — drop zone (drag & drop + click), wybór modelu Whisper (tiny/base/small/medium/large) i języka (pl/en/auto), przycisk "Prześlij i transkrybuj"
2. **Transkrypt** — log SSE postępu, edytowalny textarea z auto-save (debounce 1.5s), pola "Nazwa kursu" i "Tytuł lekcji", przycisk "Generuj notatki"
3. **Notatka** — log SSE + live streaming tekstu, edytowalny textarea z tabs Edytor/Podgląd, wybór motywu kolorystycznego (niebieski/zielony/malinowy), przycisk "Utwórz .docx"
4. **Pobierz** — przyciski pobierania .docx / .md / .txt

Nawigacja: pasek kroków na górze (①②③④), kroki odblokowane po kolei.
Przycisk "🔄 Nowy plik" w headerze (widoczny po pierwszym uploadzie) — resetuje cały formularz.
Przyciski "Prześlij" i "Utwórz .docx" przywracają normalny stan (tekst, nie spinner) po zakończeniu operacji.

Styl: profesjonalny, navy/blue palette (#1F4E79 primary, #2E75B6 accent, #D6E4F0 light, #F4F7FB background).

### Uruchom.bat

Stwórz Uruchom.bat — plik .bat do uruchomienia serwera podwójnym kliknięciem:
@echo off
cd /d "%~dp0"
echo Uruchamianie serwera Notatki z wideo...
py app.py
pause

```

### Notatka

> Ten prompt tworzy 3 pliki. Najważniejsze elementy:
> 
> - **SSE keepalive co 30s**: Bez tego przeglądarka zamyka połączenie po ~2 minutach, a Whisper medium pracuje ~4 minuty. Użytkownik widzi "Transkrypcja w toku" i nagle nic — bo połączenie padło.
> - **Port 5001**: Port 5000 bywa zajęty (AirPlay na Macu, inne serwery). Mały detal, ale oszczędza debugowania.
> - **Auto-save z debounce**: Użytkownik edytuje transkrypt/notatkę, zmiany zapisują się automatycznie po 1.5s — nie trzeba klikać "Zapisz".
> - **Przycisk "Nowy plik" w headerze**: Bez tego jedyny sposób na restart to F5 — nieintuicyjne.

---

## Krok 5: Test i poprawki

> **Ten krok to instrukcja + opcjonalny prompt do poprawek.**

### Instrukcja testowania

1. Otwórz plik `.env` i wklej swój klucz API Anthropic
2. Uruchom `Uruchom.bat` (podwójne kliknięcie)
3. Otwórz http://localhost:5001 w przeglądarce
4. **Szybki test**: Wgraj krótki plik audio (30-60s) z modelem **tiny** (najszybszy)
5. Sprawdź czy transkrypt się pojawił
6. Wpisz nazwę kursu, kliknij "Generuj notatki"
7. Wybierz motyw kolorystyczny, kliknij "Utwórz .docx"
8. Pobierz .docx i otwórz w Wordzie

### Lista kontrolna .docx

- [ ] Nagłówki sekcji na kolorowym tle?
- [ ] Emoji 🧭🔍⚠️ kolorowe (nie czarno-białe)?
- [ ] Kod w Courier New, linie kodu bez dużych przerw?
- [ ] Tabele z kolorowanymi nagłówkami?
- [ ] **Bold**, *italic*, `code` renderowane poprawnie (nie jako gwiazdki/backticki)?
- [ ] Blok tytułowy wyśrodkowany?
- [ ] Przycisk "Nowy plik" działa?

### Prompt do poprawek (jeśli coś nie działa)

```
Otwórz wygenerowany plik .docx z folderu output/ i sprawdź formatowanie.

Problemy do naprawienia:
- [tu opisz co widzisz — np. "emoji są czarno-białe" albo "gwiazdki pojawiają się dosłownie w tekście"]
```

### Notatka

> Najczęstsze problemy na tym etapie:
> 
> - **"Transkrypt błąd"** → Brak ffmpeg. Sprawdź: `winget install Gyan.FFmpeg` i restart serwera.
> - **"Błąd API"** → Zły klucz w `.env` lub brak środków na koncie Anthropic.
> - **Emoji czarno-białe** → Brak czcionki Segoe UI Emoji (Windows 10+ ma ją domyślnie).
> - **Serwer nie odpowiada** → Sprawdź czy okno CMD z `Uruchom.bat` jest otwarte.

---

## Podsumowanie architektury

```
Notatki z wideo/
├── CLAUDE.md                    ← kontekst projektu (krok 1)
├── requirements.txt             ← zależności Python (krok 1)
├── .env                         ← klucz API (krok 1)
├── Uruchom.bat                  ← launcher (krok 4)
├── app.py                       ← serwer Flask (krok 4)
├── pipeline/
│   ├── __init__.py              ← pusty (krok 2)
│   ├── transcribe.py            ← Whisper + ffmpeg (krok 3)
│   ├── generate_notes.py        ← Claude API (krok 3)
│   └── md_to_docx.py            ← Markdown → Word (krok 2)
├── static/
│   └── index.html               ← interfejs (krok 4)
└── output/                      ← generowane pliki (auto)
```

## Wskazówki

1. **Czas**: Cały proces (5 kroków) zajmuje ok. 20-30 minut z Claude Code. Warto mieć przygotowany krótki plik audio (30s) do testu.

2. **Kolejność ma znaczenie**: Krok 2 (md_to_docx) przed krokiem 3 (generate_notes) — bo system prompt w generate_notes musi produkować Markdown kompatybilny z parserem w md_to_docx.

3. **Jeśli coś nie działa**: Nie panikuj. Wklej treść błędu do Claude Code z prośbą o naprawienie. Claude Code widzi kod i potrafi zdiagnozować problem.

4. **Model Claude w generate_notes.py**: Prompt wskazuje `claude-sonnet-4-6`. Można go użyć, bo jest szybszy, tańszy niż Opus, a różnica w notatkach jest niewielka. Można to zmienić w kodzie lub poprosić o to AI (ale to zużyje limity).

---

## Prompty naprawcze z sesji nagraniowej

> Poniżej trzy pytania wysłane do Claude Code podczas nagrywania, gdy aplikacja nie działała poprawnie. Każde ilustruje inny sposób komunikowania problemu.

### Naprawa 1 — problem z formatowaniem nagłówków

```
Nagłówki notatki nie są odpowiednio sformatowane, brakuje kolorów, powinny być 3 pierwsze linie sformatowane jak w plikach testowych @output/test_zielony.docx
```

> **Co tu zadziałało:** odwołanie do konkretnego pliku testowego (`@output/test_zielony.docx`) jako wzorca. Claude Code otworzył oba pliki — testowy i aktualny — porównał ich strukturę i znalazł, gdzie parser pomija formatowanie pierwszych linii.
>
> **Wniosek:** jeśli masz plik, który "wygląda dobrze", możesz go wskazać jako punkt odniesienia. Claude Code sam znajdzie różnicę.

### Naprawa 2 — nagłówek działa w teście, nie działa w wyjściu

```
Nagłówek poprawnie pokolorował się w pliku (test.png) ale nie w tworzonym pliku (aktualny.png). Pierwsze linie w pliku z notatką muszą być odpowiednio sformatowane kolorystycznie.
```

> **Co tu zadziałało:** dołączenie dwóch screenshotów — jeden pokazujący poprawny wygląd, drugi błędny. Claude Code jest multimodalny — analizuje obrazy i na tej podstawie lokalizuje przyczynę rozbieżności w kodzie.
>
> **Wniosek:** screenshoty to skuteczny sposób opisywania problemów wizualnych. Nie musisz tłumaczyć słowami — po prostu pokaż co widzisz.

### Naprawa 3 — błąd API wklejony bezpośrednio

```
Błąd: {'type': 'error': {'details': None; 'overloaded_error', 'message': 'Overloaded'}, 'request_id': 'req_011salshfashfjklas'}
```

> **Co tu zadziałało:** to nie był "prompt do naprawy" — to surowy komunikat błędu z konsoli, wklejony bezpośrednio bez żadnego opisu. Claude Code sam rozpoznał, że API Anthropic było chwilowo przeciążone (`overloaded_error`) i dodał do kodu obsługę ponawiania zapytania po krótkiej przerwie.
>
> **Wniosek:** możesz wkleić dowolny komunikat błędu — JSON, stack trace, wyjątek Pythona — Claude Code go zrozumie i zaproponuje poprawkę. Nie musisz go tłumaczyć ani opisywać.
