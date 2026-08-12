# Test pamięci w Claude Code: checklist promptów (Memory1)

Gotowe prompty do wklejenia po kolei, żeby przetestować system zbudowany wg [pamiec-w-claude-code-przewodnik.md](pamiec-w-claude-code-przewodnik.md). Część sesji musi być osobna (np. Sesja A wymaga świeżej rozmowy), zaznaczone przy każdej.

---

## Sesja A: świeży start (test Kroku 1: Inject)

Otwórz nową rozmowę w Memory1 i jako **pierwszą wiadomość**, bez niczego wcześniej:

```
Nad czym ostatnio pracowaliśmy?
```

Oczekiwane: odpowiedź wynika z samej migawki, bez żadnego wyszukiwania czy pytań z Twojej strony.

Dodatkowo, żeby zweryfikować co dokładnie się wstrzyknęło, nie tylko co model zgaduje:

```
Pokaż mi dosłowną treść working-memory.md i dzisiejszego pliku dziennika, tak jak je odczytałeś na starcie tej sesji.
```

---

## Sesja B: test skilla (Krok 2, realna aktywacja, nie symulacja)

```
Zapamiętaj, że nasz kolor testowy w tym projekcie to fioletowy.
```

Sprawdź, czy skill faktycznie się odpalił (realna edycja working-memory.md, nie zwykła odpowiedź tekstowa).

**Zamknij sesję, otwórz nową:**

```
Jaki jest nasz kolor testowy?
```

Powinno wrócić bez przypominania.

Test nadpisania (nie dodania obok):

```
Zapamiętaj, że jednak nasz kolor testowy to niebieski, nie fioletowy.
```

Sprawdź plik: powinien być tylko wpis o niebieskim, nie oba.

Test usuwania:

```
Zapomnij o kolorze testowym.
```

**Nowa sesja**, zapytaj ponownie o kolor testowy, powinno go już nie być.

---

## Sesja C: limit znaków i konsolidacja (Krok 2)

Kilka razy pod rząd, żeby zbliżyć się do limitu ~2000-2500 znaków:

```
Zapamiętaj, że to jest testowa notatka numer [1/2/3.../10], dodana wyłącznie żeby sprawdzić konsolidację przy limicie znaków.
```

Sprawdź working-memory.md: czy przy zbliżaniu się do limitu skill konsoliduje istniejące wpisy (skraca/łączy/usuwa nieaktualne), zamiast urwać plik w połowie zdania albo zignorować limit.

Sprzątanie:

```
Zapomnij o testowych notatkach numer 1-10.
```

---

## Sesja D: recall po znaczeniu + cytowanie (Krok 4 i 5)

Zapytaj o coś z realnej, wcześniejszej rozmowy w tym projekcie, ale innymi słowami niż wtedy padły:

```
Co ustaliliśmy w sprawie [opisz temat innymi słowami niż oryginalnie użyte]?
```

Sprawdź dwie rzeczy naraz: czy w ogóle trafia (semantyka, nie tylko słowa kluczowe), i czy odpowiedź cytuje plik/datę/nagłówek, a nie tylko parafrazuje bez wskazania źródła.

Test uczciwości:

```
Co ustaliliśmy w sprawie [coś, czego na pewno nigdy nie było w tym projekcie]?
```

Oczekiwane: wprost "nie znalazłem", nie zmyślona odpowiedź.

---

## Sesja E: "zmieniłem zdanie" na prawdziwych danych

```
Zapamiętaj, że domyślny model do tego projektu to Sonnet.
```

Potem:

```
Zapamiętaj, że jednak zmieniłem zdanie, domyślny model to teraz Opus, nie Sonnet.
```

**Nowa sesja:**

```
Jaki jest domyślny model do tego projektu?
```

Powinno wygrać świeższe ustalenie (Opus), nie starsze, mimo że oba są semantycznie blisko siebie.

---

## Sesja F: dociekliwe pytania o Krok 6

```
Sprawdź w ~/.claude/projects/ czy po dzisiejszej, normalnej pracy (nie budowie hooka) powstały nowe, małe transkrypty z zagnieżdżonych wywołań claude -p, podobne do tych, które wcześniej odfiltrowałeś przy imporcie. Chcę wiedzieć, czy to się będzie powtarzać przy zwykłym użyciu, czy to był jednorazowy efekt dzisiejszej budowy.
```

```
Wróć do importu historii sprzed chwili: sprawdź, dlaczego 5 z 7 zaimportowanych wymian pochodziło z okresu PO tym, jak hook Stop już istniał. Czy to było okno, w którym hook był w trakcie refaktoru i coś ominął na żywo?
```

```
Pokaż zawartość plików w memory/dziennik/ dla dat sprzed dzisiejszego dnia, tych z importu. Chcę potwierdzić, że mają realną, historyczną datę w nazwie/treści, nie dzisiejszą.
```

---

## Sesja G: kontrola kosztu (Krok 3)

```
Podsumuj, ile razy dotąd odpalił się hook Stop w tym projekcie i jaki jest łączny szacowany koszt wywołań Haiku do tej pory.
```

---

## Autor

**Adam Kopeć**: [friendlyai.pl](https://www.friendlyai.pl/) · [YouTube](https://www.youtube.com/@Friendly_AI_PL) · [adam@friendlyai.pl](mailto:adam@friendlyai.pl)
