# Lokalna transkrypcja audio/wideo na GPU (Cohere Transcribe)

Skrypt Pythona, który zamienia plik wideo lub audio na tekst **lokalnie, na karcie graficznej NVIDIA**, z paskiem postępu w terminalu. Dla 7-minutowego nagrania: około **30 sekund na GPU** zamiast 42 minut na procesorze.

Instrukcja jest napisana dla osoby, która potrafi otworzyć `cmd` w Windowsie, ale niekoniecznie miała wcześniej do czynienia z Pythonem czy instalacją bibliotek. Każdy krok można wykonać kopiując komendy bez modyfikacji.

---

## Spis treści

1. [Wymagania sprzętowe](#wymagania-sprzętowe)
2. [Krok 1: Sprawdź swoją kartę GPU](#krok-1-sprawdź-swoją-kartę-gpu)
3. [Krok 2: Zainstaluj Pythona 3.12](#krok-2-zainstaluj-pythona-312)
4. [Krok 3: Zainstaluj biblioteki](#krok-3-zainstaluj-biblioteki)
5. [Krok 4: Zainstaluj ffmpeg](#krok-4-zainstaluj-ffmpeg)
6. [Krok 5: Konto i token Hugging Face](#krok-5-konto-i-token-hugging-face)
7. [Krok 6: Zapisz skrypt](#krok-6-zapisz-skrypt)
8. [Krok 7: Uruchom transkrypcję](#krok-7-uruchom-transkrypcję)
9. [Najczęstsze problemy](#najczęstsze-problemy)

---

## Wymagania sprzętowe

* **Windows 10 lub 11**
* **Karta NVIDIA RTX 20, 30, 40 lub 50 serii** (inne karty NVIDIA mogą nie zadziałać, karty AMD/Intel nie zadziałają wcale)
* **RAM: minimum 8 GB**
* **VRAM (pamięć karty graficznej): minimum 6 GB**, model zajmuje około 4 GB
* **Wolne miejsce na dysku: około 10 GB** (sam model to ~6 GB, biblioteki ~3 GB)
* **Połączenie z internetem** do instalacji i pobrania modelu
* **Bezpłatne konto na [huggingface.co](https://huggingface.co)**

---

## Krok 1: Sprawdź swoją kartę GPU

Musisz wiedzieć, jaką masz kartę, bo od tego zależy wybór wersji PyTorch w Kroku 3.

1. Naciśnij `Ctrl + Shift + Esc`, żeby otworzyć **Menedżer zadań**
2. Przejdź na zakładkę **Wydajność** (Performance)
3. Po lewej stronie znajdź pozycję zaczynającą się od **GPU** i sprawdź pełną nazwę

Porównaj nazwę z tabelą:

| Twoja karta | Architektura | Wersja CUDA do instalacji |
|---|---|---|
| RTX 5060, 5070, 5080, 5090 | Blackwell | `cu128` |
| RTX 4060, 4070, 4080, 4090 | Ada Lovelace | `cu121` |
| RTX 3060, 3070, 3080, 3090 | Ampere | `cu121` |
| RTX 2060, 2070, 2080 | Turing | `cu121` |

Zapamiętaj, czy potrzebujesz **`cu121`** czy **`cu128`**. Wrócisz do tego w Kroku 3.

---

## Krok 2: Zainstaluj Pythona 3.12

> **Dlaczego akurat 3.12?** Wersje 3.13 i 3.14 nie mają jeszcze gotowych paczek PyTorch z obsługą CUDA. Wersja 3.12 jest najnowszą, która działa z GPU.

### Sprawdź, czy już masz Pythona

Otwórz **cmd**:
> Start → wpisz `cmd` → Enter

Wpisz:

```cmd
py -0
```

**Co możesz zobaczyć:**

* **Komunikat błędu typu `'py' nie jest rozpoznawane`** — nie masz żadnego Pythona, idź do sekcji "Instalacja od zera" niżej
* **Lista wersji, w której jest `-V:3.12`** — masz już Pythona 3.12, możesz przejść do Kroku 3
* **Lista wersji bez 3.12** (np. tylko 3.11 albo 3.13) — masz Pythona, ale w złej wersji, doinstaluj 3.12 obok według instrukcji niżej. Stara wersja zostanie nietknięta i nadal będzie działać do innych projektów

### Instalacja od zera (lub doinstalowanie 3.12 obok)

1. Wejdź na **[python.org/downloads/release/python-31210](https://www.python.org/downloads/release/python-31210/)**
2. Na samym dole strony, w tabeli "Files", pobierz **Windows installer (64-bit)**, czyli plik `python-3.12.10-amd64.exe`
3. Uruchom pobrany instalator
4. **Bardzo ważne:** na pierwszym ekranie zaznacz checkbox **„Add python.exe to PATH"** na samym dole. Bez tego komendy z dalszej części instrukcji nie zadziałają
5. Kliknij **„Install Now"** i poczekaj, aż instalator skończy

### Sprawdzenie, czy działa

Zamknij stary `cmd` i otwórz nowy (to ważne, żeby zaczytał nowe ustawienia PATH):

```cmd
py -3.12 --version
```

Powinno wyświetlić `Python 3.12.10`. Jeśli tak, krok zakończony.

---

## Krok 3: Zainstaluj biblioteki

Biblioteki to gotowe paczki kodu, które skrypt wykorzystuje. Musimy ich kilka. Każda komenda poniżej pobiera i instaluje to, czego brakuje.

> **Co jeśli mam już niektóre z tych bibliotek?** Polecenie `pip install` sprawdza, co już jest w systemie. Jeśli paczka jest obecna w odpowiedniej wersji, pip pominie ją z komunikatem `Requirement already satisfied`. Jeśli jest w starszej wersji, dodaliśmy flagę `--upgrade`, żeby na pewno mieć aktualną.

### Otwórz PowerShell jako administrator

Część komend wymaga uprawnień administratora:

> Start → wpisz `PowerShell` → kliknij **prawym przyciskiem** na wyniku → **„Uruchom jako administrator"**

### Komendy zależnie od karty GPU

**Jeśli masz RTX 50-serii (Blackwell):**

```powershell
py -3.12 -m pip install --upgrade torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
py -3.12 -m pip install --upgrade transformers soundfile librosa accelerate ffmpeg-python
```

**Jeśli masz RTX 20, 30 lub 40-serii:**

```powershell
py -3.12 -m pip install --upgrade torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
py -3.12 -m pip install --upgrade transformers soundfile librosa accelerate ffmpeg-python
```

Każda z tych komend potrwa kilka minut, bo PyTorch waży około 2.5 GB.

### Co właściwie instalujemy

* **`torch`** — silnik obliczeń AI, w wersji z obsługą CUDA czyli karty graficznej
* **`transformers`** — interfejs do modeli z Hugging Face
* **`soundfile` i `librosa`** — odczyt i przetwarzanie plików audio
* **`accelerate`** — wymagane do efektywnego ładowania dużych modeli
* **`ffmpeg-python`** — wrapper Pythonowy do ffmpeg, używany do konwersji wideo na audio

### Weryfikacja, że PyTorch widzi GPU

To kluczowy test. Bez niego wszystko pojedzie na procesorze i będzie boleśnie wolne.

```powershell
py -3.12 -c "import torch; print('CUDA dostępne:', torch.cuda.is_available()); print('Wersja CUDA:', torch.version.cuda)"
```

Powinno wyświetlić coś takiego:

```
CUDA dostępne: True
Wersja CUDA: 12.1
```

(lub `12.8` jeśli masz RTX 50-serii)

**Jeśli widzisz `CUDA dostępne: False`**, to znaczy że masz zainstalowanego `torch` w wersji CPU (bez GPU). Trzeba go odinstalować i zainstalować ponownie z odpowiedniego indeksu:

```powershell
py -3.12 -m pip uninstall -y torch torchvision torchaudio
```

A następnie powtórz pierwszą komendę z tego kroku (tę z `--index-url`).

---

## Krok 4: Zainstaluj ffmpeg

ffmpeg to zewnętrzne narzędzie do obróbki wideo i audio. Skrypt używa go do wyciągnięcia ścieżki dźwiękowej z pliku wideo.

### Sprawdź, czy już masz

W zwykłym `cmd`:

```cmd
ffmpeg -version
```

Jeśli pokazuje numer wersji, masz ffmpeg i możesz przejść do Kroku 5. Jeśli widzisz `'ffmpeg' nie jest rozpoznawane`, zainstaluj go.

### Instalacja przez winget

W **PowerShell jako administrator** (tym samym co w Kroku 3):

```powershell
winget install ffmpeg
```

Po zakończeniu instalacji **zamknij wszystkie okna cmd i PowerShell** i otwórz nowe (znowu chodzi o przeładowanie PATH).

### Sprawdź ponownie

```cmd
ffmpeg -version
```

Powinno wyświetlić numer wersji i listę opcji kompilacji.

> **Jeśli `winget install` nie zadziała** lub po instalacji `ffmpeg -version` nadal nie działa, pobierz ręcznie build z [gyan.dev/ffmpeg/builds](https://www.gyan.dev/ffmpeg/builds/), rozpakuj do `C:\ffmpeg` i dodaj `C:\ffmpeg\bin` do zmiennej środowiskowej PATH (Start → "zmienne środowiskowe" → edytuj Path).

---

## Krok 5: Konto i token Hugging Face

Model jest hostowany na Hugging Face i wymaga jednorazowej akceptacji warunków oraz tokena dostępowego.

1. Wejdź na **[huggingface.co](https://huggingface.co)** i kliknij **Sign Up** (rejestracja jest bezpłatna i wymaga tylko emaila)
2. Po zalogowaniu wejdź na stronę modelu: **[huggingface.co/CohereLabs/cohere-transcribe-03-2026](https://huggingface.co/CohereLabs/cohere-transcribe-03-2026)**
3. Kliknij przycisk **„Agree and access repository"**, żeby zaakceptować warunki licencji (jednorazowo)
4. Przejdź do **[huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)**
5. Kliknij **„New token"**, nadaj mu dowolną nazwę (np. `transkrypcja`), wybierz typ **„Read"** i kliknij utworzenie
6. **Skopiuj wygenerowany token** w postaci `hf_abc123...` i zapisz go gdzieś tymczasowo, będzie potrzebny w następnym kroku

> Token zawiera tylko litery angielskie, cyfry i podkreślniki. Nigdy nie udostępniaj go publicznie i nie wrzucaj na GitHub, bo umożliwia dostęp do twojego konta.

---

## Krok 6: Zapisz skrypt

1. Utwórz folder, w którym będzie mieszkał skrypt i pliki do transkrypcji, np. `C:\Transkrypcja`
2. Otwórz **Notatnik** (Start → wpisz `Notatnik` → Enter)
3. Wklej cały kod poniżej
4. **Znajdź linię z `HF_TOKEN`** i wklej swój token ze schowka w miejsce `hf_twój_token_tutaj`
5. Kliknij **Plik → Zapisz jako**
   * Folder: `C:\Transkrypcja`
   * Nazwa pliku: `transcribe.py`
   * Typ pliku: **Wszystkie pliki** (to ważne, żeby Notatnik nie dokleił `.txt` na końcu)
   * Kodowanie: **UTF-8**

### Pełny kod skryptu

```python
import os
import sys
import time
import argparse

os.environ["HF_TOKEN"] = "hf_twój_token_tutaj"  # ← WKLEJ SWÓJ TOKEN TUTAJ

import ffmpeg
import soundfile as sf
import torch
from transformers import AutoProcessor, CohereAsrForConditionalGeneration

# ══════════════════════════════════════════════════════════════
# Parametry wywołania z linii poleceń
# ══════════════════════════════════════════════════════════════
parser = argparse.ArgumentParser(
    description="Lokalna transkrypcja audio/wideo na GPU (Cohere Transcribe).",
    formatter_class=argparse.ArgumentDefaultsHelpFormatter
)
parser.add_argument(
    "input_file",
    help="Ścieżka do pliku wideo lub audio do transkrypcji"
)
parser.add_argument(
    "--lang",
    default="pl",
    help="Kod języka nagrania (np. pl, en, de, fr)"
)
parser.add_argument(
    "--segment",
    type=int,
    default=30,
    help="Długość pojedynczego segmentu w sekundach"
)
parser.add_argument(
    "--output",
    default=None,
    help="Ścieżka pliku wyjściowego .txt (domyślnie nazwa wejścia + .txt)"
)
args = parser.parse_args()

INPUT_FILE  = args.input_file
LANGUAGE    = args.lang
SEGMENT_SEC = args.segment

if not os.path.isfile(INPUT_FILE):
    print(f"❌ Nie znaleziono pliku: {INPUT_FILE}")
    sys.exit(1)

SAMPLE_RATE = 16000
OUTPUT_WAV  = "_temp_audio.wav"
OUTPUT_TXT  = args.output or (os.path.splitext(INPUT_FILE)[0] + ".txt")
MODEL_ID    = "CohereLabs/cohere-transcribe-03-2026"

start_time = time.time()

# ── KROK 1: Konwersja do WAV ──────────────────────────────────
print(f"[1/4] Konwersja '{INPUT_FILE}' → WAV...")
ffmpeg.input(INPUT_FILE).output(
    OUTPUT_WAV, ac=1, ar=SAMPLE_RATE
).overwrite_output().run(quiet=True)
print("      Gotowe.")

# ── KROK 2: Wczytaj audio i podziel na segmenty ───────────────
print("[2/4] Wczytywanie audio i podział na segmenty...")
audio, sr = sf.read(OUTPUT_WAV)
if sr != SAMPLE_RATE:
    import librosa
    audio = librosa.resample(audio, orig_sr=sr, target_sr=SAMPLE_RATE)

segment_len   = SEGMENT_SEC * SAMPLE_RATE
total_samples = len(audio)
segments      = [audio[i : i + segment_len] for i in range(0, total_samples, segment_len)]
total_seg     = len(segments)
duration_min  = total_samples / SAMPLE_RATE / 60
print(f"      Długość nagrania: {duration_min:.1f} min → {total_seg} segmentów po {SEGMENT_SEC}s")
print(f"      Język: {LANGUAGE}")

# ── KROK 3: Załaduj model ─────────────────────────────────────
print("[3/4] Ładowanie modelu...")
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print(f"      Używane urządzenie: {device}")
if device.type == "cpu":
    print("      ⚠  UWAGA: GPU niedostępne, transkrypcja na CPU będzie bardzo wolna!")

processor = AutoProcessor.from_pretrained(MODEL_ID)
model     = CohereAsrForConditionalGeneration.from_pretrained(
    MODEL_ID, device_map="auto"
)
print("      Model załadowany.")

# ── KROK 4: Transkrypcja segment po segmencie ─────────────────
print(f"[4/4] Transkrypcja → '{OUTPUT_TXT}'")

with open(OUTPUT_TXT, "w", encoding="utf-8") as out_file:
    for idx, segment in enumerate(segments, start=1):
        inputs = processor(
            segment,
            sampling_rate=SAMPLE_RATE,
            return_tensors="pt",
            language=LANGUAGE
        )
        inputs = {
            k: v.to(device=device, dtype=torch.bfloat16)
               if hasattr(v, "dtype") and v.dtype == torch.float32
               else v.to(device)
               if hasattr(v, "to")
               else v
            for k, v in inputs.items()
        }

        with torch.no_grad():
            generated_ids = model.generate(
                **inputs,
                max_new_tokens=512,
                pad_token_id=processor.tokenizer.pad_token_id
            )

        text = processor.batch_decode(generated_ids, skip_special_tokens=True)[0].strip()

        start_sec = (idx - 1) * SEGMENT_SEC
        h = start_sec // 3600
        m = (start_sec % 3600) // 60
        s = start_sec % 60
        timestamp = f"[{h:02d}:{m:02d}:{s:02d}]"

        out_file.write(f"{timestamp} {text}\n")
        out_file.flush()

        pct = idx / total_seg * 100
        bar = "█" * int(pct / 5) + "░" * (20 - int(pct / 5))
        print(f"\r      [{bar}] {idx}/{total_seg} ({pct:.0f}%)", end="", flush=True)

print(f"\n\n✅ Gotowe! Transkrypcja zapisana do: {OUTPUT_TXT}")

elapsed = time.time() - start_time
print(f"⏱  Czas wykonania: {int(elapsed // 60)}m {int(elapsed % 60)}s")

if os.path.exists(OUTPUT_WAV):
    os.remove(OUTPUT_WAV)
```

---

## Krok 7: Uruchom transkrypcję

1. Skopiuj plik wideo lub audio do folderu `C:\Transkrypcja` (np. `nagranie.mp4`)
2. Otwórz `cmd` i przejdź do tego folderu:

```cmd
cd C:\Transkrypcja
```

3. Uruchom skrypt podając nazwę pliku jako argument:

```cmd
py -3.12 transcribe.py nagranie.mp4
```

### Inne sposoby wywołania

**Z innym językiem** (np. angielski):

```cmd
py -3.12 transcribe.py wywiad.mp4 --lang en
```

**Z dłuższymi segmentami** (60 sekund zamiast domyślnych 30) **i własną nazwą wyjścia**:

```cmd
py -3.12 transcribe.py podcast.m4a --segment 60 --output podcast_tekst.txt
```

**Pomoc i lista wszystkich parametrów**:

```cmd
py -3.12 transcribe.py --help
```

### Co zobaczysz w trakcie

```
[1/4] Konwersja 'nagranie.mp4' → WAV...
      Gotowe.
[2/4] Wczytywanie audio i podział na segmenty...
      Długość nagrania: 7.0 min → 14 segmentów po 30s
      Język: pl
[3/4] Ładowanie modelu...
      Używane urządzenie: cuda
      Model załadowany.
[4/4] Transkrypcja → 'nagranie.txt'
      [████████████████████] 14/14 (100%)

✅ Gotowe! Transkrypcja zapisana do: nagranie.txt
⏱  Czas wykonania: 0m 31s
```

> **Pierwsze uruchomienie potrwa znacznie dłużej**, bo Hugging Face pobierze model (~6 GB). Kolejne wywołania będą używać wersji z cache i zaczną się od razu.

Plik `nagranie.txt` znajdziesz w folderze `C:\Transkrypcja`. Każda linia zawiera znacznik czasu i tekst odpowiadającego segmentu.

---

## Najczęstsze problemy

| Problem | Przyczyna i rozwiązanie |
|---|---|
| `'py' nie jest rozpoznawane jako polecenie` | Python nie jest zainstalowany albo nie zaznaczyłeś "Add Python to PATH" w instalatorze. Zainstaluj ponownie z zaznaczonym checkboxem |
| `No matching distribution found for torch` | Masz Pythona 3.13 lub 3.14. Doinstaluj 3.12 obok według Kroku 2 |
| `CUDA dostępne: False` | Masz `torch` w wersji CPU. Odinstaluj go (`pip uninstall torch torchvision torchaudio`) i zainstaluj ponownie z indeksu cu121 lub cu128 |
| `sm_120 is not compatible with current PyTorch` | Masz RTX 50-serii i zainstalowałeś wersję cu121. Odinstaluj torch i zainstaluj wariant cu128 |
| `401 Unauthorized` lub `403 Forbidden` przy ładowaniu modelu | Nie zaakceptowałeś warunków na stronie modelu albo token jest nieprawidłowy. Sprawdź Krok 5 |
| `UnicodeEncodeError` przy `HF_TOKEN` | W tokenie są jakieś dziwne znaki (może skopiowałeś ze spacją). Wygeneruj nowy token i wklej dokładnie ciąg `hf_...` |
| `'ffmpeg' nie jest rozpoznawane` | ffmpeg nie jest w PATH. Zamknij i otwórz cmd ponownie. Jeśli nie pomogło, zainstaluj ręcznie według uwagi w Kroku 4 |
| `ModuleNotFoundError: No module named 'transformers'` | Uruchamiasz przez `python` zamiast `py -3.12`. Biblioteki zainstalowane pod 3.12 są niewidoczne dla innych wersji Pythona |
| `Expected all tensors to be on the same device` | Używasz starej wersji skryptu. Skopiuj kod z Kroku 6 ponownie |
| Transkrypcja urywa się po kilku linijkach | Stary skrypt bez `max_new_tokens=512`. Skopiuj kod z Kroku 6 ponownie |

---

## Licencja i źródło modelu

Model `CohereLabs/cohere-transcribe-03-2026` jest udostępniany przez Cohere Labs na warunkach widocznych na stronie modelu w Hugging Face. Skrypt z tej instrukcji jest do wykorzystania bez ograniczeń.

---

## Autor

**Adam Kopeć** — [friendlyai.pl](https://www.friendlyai.pl/) · [YouTube](https://www.youtube.com/@Friendly_AI_PL)
