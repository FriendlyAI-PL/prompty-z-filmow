# Pamięć w Claude Code: własny system store / inject / recall (bez serwera)

Instrukcja krok po kroku, jak zbudować w Claude Code system pamięci podobny do tego, za który ludzie chwalą agenta Hermes: zapamiętywanie ustaleń, automatyczne wczytywanie kontekstu na starcie sesji i wyszukiwanie starych rozmów po znaczeniu, a nie po dokładnych słowach.

**Cel tych promptów:** Hermes sam z siebie jest darmowy i open source, ale to tylko warstwa orkiestracji, nie model. Żeby cokolwiek odpowiedział, potrzebuje osobno opłacanego modelu (Claude, GPT) i osobnego miejsca, gdzie non-stop działa (VPS albo własny komputer, bo ma pracować 24/7, nie tylko w trakcie rozmowy). Claude Code ma model i środowisko uruchomieniowe w jednym, w ramach subskrypcji, którą już masz. Te prompty budują ten sam efekt (store/inject/recall) na tym, co już opłacasz: bez osobnego serwera, bez VPS, bez drugiej subskrypcji, tylko pliki w projekcie plus kilka hooków i skill.

**Na czym to polega:** każdy system pamięci odpowiada na 3 pytania. Store: kiedy coś jest ważne, jak to się zapisuje i gdzie? Inject: co ładuje się automatycznie na starcie sesji, bez pytania? Recall: jak znaleźć starą informację, gdy o nią pytasz? Claude Code bez konfiguracji robi wszystkie trzy słabo: prawie nic nie zapisuje samodzielnie, prawie nic nie wczytuje na starcie, a wyszukiwanie starych sesji to ręczne przeszukiwanie plików po słowach kluczowych.

**Materiał źródłowy:** plan i prompty pochodzą z darmowego przewodnika Agentic Academy ("Claude Code Memory Plan"), oparte na analizie, dlaczego ludzie migrują do agenta Hermes (pamięć jako głównie wymieniany powód).

![Pamięć w Claude Code: wstrzykiwanie, zapamiętywanie, wyszukiwanie](images/pamiec-w-claude-code-widok-glowny.png)

---

## Ile pamięci ma naprawdę Hermes, i jak wypadamy na tym tle

Opinia "Hermes ma dużą pamięć" dotyczy w większości niewłaściwej warstwy. Architektura Hermesa ma 3 poziomy:

- **Poziom 1 (wstrzykiwana migawka):** wcale nie jest duży, celowo. `MEMORY.md` ma twardy limit **2200 znaków**, `USER.md` **1375 znaków**, razem ok. 3575 znaków (~1300 tokenów). To świadoma filozofia zespołu, nie tymczasowe ograniczenie: trzymać w promptcie tylko najważniejsze fakty, głównie dla taniości cache'owania. To praktycznie ta sama skala co nasz limit working-memory.md (2000-2500 znaków) z Kroku 1. **Tu jesteśmy na równi, nie gorzej ani lepiej.**
- **Poziom 2 (baza SQLite ze wszystkimi rozmowami):** to jest ta faktycznie duża, nieograniczona część, wyszukiwana przez FTS5 (indeks pełnotekstowy) w ~20ms. Ale to wyszukiwanie tylko po dokładnych słowach, ta sama wada, którą wideo wytyka Hermesowi (nie znajdzie rozmowy o "Stripe", jeśli nie użyjesz słowa "Stripe" w pytaniu).
- **Poziom 3 (opcjonalny, zewnętrzny dostawca pamięci semantycznej):** dopiero to daje Hermesowi wyszukiwanie po znaczeniu, przez osobną, często płatną/hostowaną usługę (Mem0, Honcho, Membase), dokładaną do podstawowego, darmowego Hermesa.

**Nasza przewaga:** Krok 4 daje wyszukiwanie semantyczne wbudowane od razu, bez trzeciej, zewnętrznej warstwy. To, co u Hermesa wymaga dopłacenia/podpięcia zewnętrznego serwisu, u nas jest częścią tej samej, darmowej instrukcji, w ramach subskrypcji, którą już masz.

**Uczciwe zastrzeżenie, jedna rzecz na korzyść Hermesa:** ich wyszukiwarka zwraca dosłowne, oryginalne wiadomości, nie streszczenia. Nasz system przechowuje surowe transkrypty osobno (`memory/transkrypty-archiwum/` z Kroku 3), ale to, co faktycznie się przeszukuje w Kroku 4, to streszczenia zrobione przez hook Stop, nie oryginalny tekst słowo w słowo. Jeśli zależy Ci na dosłownym cytacie, nie streszczeniu, to miejsce, w którym Hermes ma realną przewagę, nie iluzoryczną.

---

## ⚠️ Liczy się zakładka Code, nie Home

Podział, który ma tu znaczenie, to nie "aplikacja Desktop kontra CLI", tylko **która zakładka w Twojej aplikacji**:

- **Home (czat):** zwykła rozmowa z Claude, bez dostępu do plików projektu, bez hooków, bez basha. Tu ten plan się nie zmieści, bo nie ma zdarzeń (start sesji, koniec każdej tury), na których dałoby się zaczepić automatyczne wstrzykiwanie czy przechwytywanie.
- **Code:** to jest sam Claude Code, uruchomiony w oknie aplikacji zamiast w terminalu, ten sam silnik co CLI i wtyczki VS Code/JetBrains. Ma dostęp do plików projektu, basha, i obsługuje hooki (`SessionStart`, `Stop`) zdefiniowane w `.claude/settings.json` oraz skille, dokładnie tak samo jak CLI.

Jeśli pracujesz w zakładce **Code** (tak jak na Twoim zrzucie: widać tam selektor trybu uprawnień "Accept edits", model "Sonnet 5", poziom wysiłku "High" i folder projektu "Analiza komputera", to wszystko są elementy Claude Code, nie zwykłego czatu), to **tak, to w zupełności wystarczy**. Wklejasz poniższe prompty kolejno w tej właśnie zakładce, w projekcie, w którym chcesz mieć pamięć.

Jedyna sytuacja, w której to nie zadziała, to próba zrobienia tego w zakładce **Home**, bo tam nie ma hooków ani dostępu do plików projektu.

---

## 📌 Kolejność ma znaczenie, ale nie przez numerki

Prompty poniżej (poza jednym zastrzeżeniem w Kroku 6) celowo nie odwołują się do "Kroku 1" czy "Kroku 3", tylko do konkretnych plików (`working-memory.md`, "dzisiejszy plik dziennika"). To zrobione świadomie: Claude Code nie ma pojęcia, co to "Krok 3" znaczy, to etykieta z tego dokumentu, nie coś, co rozpoznaje w projekcie. Gdyby prompty odwoływały się do numerów, wystarczyłby jeden wklejony prompt niezwiązany z tematem pomiędzy nimi, nowa sesja, albo kompresja długiej rozmowy, żeby to odwołanie przestało mieć sens.

Dzięki temu możesz wykonywać kroki w osobnych sesjach, nawet dni czy tygodnie od siebie, i możesz między nimi swobodnie pracować nad czymkolwiek innym w tym samym projekcie. Jedyny realny warunek: krok, do którego coś się odwołuje, musi już istnieć na dysku, zanim wkleisz kolejny. Jeśli nie jesteś pewien, czy poprzedni krok faktycznie się udał, zapytaj wprost przed wklejeniem następnego promptu, np. "sprawdź, czy w tym projekcie jest już hook SessionStart i plik working-memory.md, zanim zaczniesz".

> 💡 **Testuj po każdym kroku, nie czekaj na wszystkie 6.** To właśnie takie, na bieżąco robione testy złapały większość realnych błędów przy budowie tej instrukcji (fałszywy wpis w skillu, ryzyko rekurencji w hooku, zła granulacja chunkingu). Czekanie z testami do samego końca oznacza, że jak coś nie działa, nie wiesz, który z 6 kroków jest winny. Co jest testowalne kiedy: **Inject** samodzielnie po Kroku 1, **Store** samodzielnie po Kroku 2, zapis do dziennika samodzielnie po Kroku 3. **Recall** po znaczeniu wymaga Kroku 3 z jakimikolwiek danymi w dzienniku, więc ma sens dopiero po Kroku 4. **Cytowanie** wymaga Kroku 4, ma sens dopiero po Kroku 5.

---

## Spis treści

0. [Ile pamięci ma naprawdę Hermes, i jak wypadamy na tym tle](#ile-pamięci-ma-naprawdę-hermes-i-jak-wypadamy-na-tym-tle)
1. [Krok 1: Zamrożona migawka (frozen snapshot)](#krok-1-zamrożona-migawka-frozen-snapshot)
2. [Krok 2: Zapisy kuratorowane przez agenta](#krok-2-zapisy-kuratorowane-przez-agenta)
3. [Krok 3: Pełne przechwytywanie każdej wymiany](#krok-3-pełne-przechwytywanie-każdej-wymiany)
4. [Krok 4: Wyszukiwanie semantyczne](#krok-4-wyszukiwanie-semantyczne)
5. [Krok 5: Cytowanie źródła](#krok-5-cytowanie-źródła)
6. [Krok 6 (opcjonalny): Import historii sprzed instalacji](#krok-6-opcjonalny-import-historii-sprzed-instalacji)
7. [Po uruchomieniu wszystkiego: co dalej](#po-uruchomieniu-wszystkiego-co-dalej)
8. [Wiele projektów: kopiowanie do nowego folderu](#wiele-projektów-kopiowanie-do-nowego-folderu)

---

## Krok 1: Zamrożona migawka (frozen snapshot)

**Co robimy:** plik pamięci roboczej z limitem znaków plus dzienny log, wczytywane automatycznie na starcie każdej sesji przez hook SessionStart.

> 💡 Fakty, o które Claude nie powinien musieć dopytywać (aktualne priorytety, otwarte pytania, decyzje w toku) mają być w kontekście, zanim jeszcze cokolwiek napiszesz. Traktuj migawkę jako zamrożoną na czas sesji: zapisy z Kroku 2 mają efekt dopiero w kolejnej sesji, nie w trakcie trwającej rozmowy. Dzięki temu obraz "co jest prawdą teraz" jest stabilny przez całą rozmowę.

```
Stwórz folder memory/ w tym projekcie z plikiem working-memory.md
o ograniczonym rozmiarze (maksymalnie około 2000-2500 znaków),
z sekcjami: aktywne wątki, notatki warte zachowania, oczekujące
decyzje, oraz osobnym plikiem dziennika z datą na dziś.

Dodaj hook SessionStart w Claude Code, który odczytuje oba pliki
i wstrzykuje je jako kontekst tła na początku każdej sesji, po
cichu, bez powitania czy podsumowania.

Traktuj migawkę jako zamrożoną na czas sesji: zapisy mają efekt
dopiero w kolejnej sesji, nie w trakcie trwającej rozmowy.
```

> ⚠️ **Ten hook prawie na pewno nie zadziała jeszcze w TEJ rozmowie, w której go tworzysz, i to jest normalne.** Jeśli folder `.claude/` nie istniał w projekcie przed wklejeniem tego promptu (a przy pierwszym uruchomieniu tej instrukcji zwykle nie istnieje), mechanizm śledzący config zaczyna go obserwować dopiero, gdy istniał już na starcie sesji. `SessionStart` i tak uruchamia się tylko raz, na samym początku, czyli w momencie, który już minął. Komenda `/hooks` (przeładowanie configu) bywa niedostępna, np. w zakładce Code w Desktopie. Najpewniejszy sposób: po prostu zacznij nową rozmowę w tym samym projekcie, dopiero wtedy zobaczysz migawkę wstrzykniętą po cichu. Od tego momentu `.claude/` już istnieje, więc kolejne kroki nie powinny mieć tego problemu.

---

## Krok 2: Zapisy kuratorowane przez agenta

**Co robimy:** skill, który reaguje na frazy typu "zapamiętaj to", i sam decyduje, czy dodać nowy wpis, zastąpić nieaktualny, czy coś usunąć, bez ręcznej edycji pliku przez Ciebie.

> 💡 Reguły oceny (co jest warte zapamiętania, kiedy coś jest duplikatem) mają być zapisane jako czytelne, edytowalne instrukcje wewnątrz skilla, nie zaszyte na sztywno w kodzie. Dzięki temu możesz je przeczytać, nie zgodzić się z nimi i poprawić w miarę używania systemu.

> ⚠️ Świeżo utworzony skill zwykle nie załaduje się w tej samej turze, w której powstał, lista skilli odświeża się przy kolejnej wiadomości/sesji. To normalne, nie błąd. Jeśli Claude Code od razu proponuje "przetestować logikę ręcznie" zamiast czekać na prawdziwe wywołanie, zastrzeż wprost, żeby robił to na **kopii** pliku working-memory, nie na oryginale, i żeby nic w nim nie wymyślał w Twoim imieniu (np. fałszywej notatki "przypisanej" Tobie jako przykład). Prawdziwy test i tak wykonaj sam, przy następnej wiadomości, mówiąc coś w stylu "zapamiętaj, że...".

```
Stwórz skilla, który uruchamia się na frazy typu "zapamiętaj to",
"zanotuj", "zapisz", "zapomnij o". Po uruchomieniu ma odczytać
CAŁY plik working-memory (to krok deduplikacji), sprawdzić, czy
nowa informacja nie jest duplikatem lub czy nie zastępuje czegoś
nieaktualnego, a następnie dodać, zastąpić lub usunąć treść
odpowiednio do sytuacji, z poszanowaniem limitu znaków z pliku
migawki (jeśli dodanie przekroczyłoby limit, najpierw skonsoliduj
istniejące wpisy).

Zapisz właściwe reguły oceny jako edytowalne instrukcje wewnątrz
skilla, nie jako sztywno zakodowaną logikę.
```

---

## Krok 3: Pełne przechwytywanie każdej wymiany

**Co robimy:** plik z Kroku 1 to skrót, wersja edytowana i ograniczona rozmiarem. Osobno, każda pojedyncza wymiana zdań ma trafiać do trwałego, nieedytowanego archiwum, żeby za miesiące dało się znaleźć dokładną rozmowę, z której wzięła się dana decyzja.

> 📌 **Ważna korekta:** hook `Stop` w Claude Code uruchamia się po **każdej turze** (za każdym razem, gdy Claude kończy odpowiedź i oddaje głos Tobie), nie raz na koniec całej sesji, jak mogłaby sugerować nazwa kroku. To akurat dobrze, dokładnie to daje "przechwytywanie każdej wymiany", o które chodzi w tym kroku, ale ma to wprost przełożenie na koszt: przy sesji z 50 wymianami to 50 osobnych wywołań modelu, nie jedno. Realny, zmierzony koszt pojedynczego wywołania Haiku to ok. $0.005–0.01, głównie narzut cache'owania promptu systemowego. Jeśli robisz dużo długich sesji dziennie, warto od razu poprosić Claude Code o throttling (np. pomijaj streszczanie bardzo krótkich/trywialnych wymian, albo streszczaj nie częściej niż raz na X minut), zamiast dokładać to później.

> ⚠️ **Ryzyko rekurencji.** Skrypt hooka wywołuje `claude -p`, czyli sam w sobie kolejne uruchomienie Claude Code. Bez zabezpieczenia to nowe uruchomienie mogłoby odpalić własny hook `Stop`, a ten kolejny, w nieskończoność. Poproś Claude Code wprost o izolację tego wywołania od hooków projektu (w praktyce: flaga `--setting-sources=` przy `claude -p`, najlepiej dodatkowo zabezpieczona własną zmienną środowiskową odczytywaną na starcie workera), i o realny test potwierdzający, że nie dochodzi do rekurencji, zanim uznasz krok za gotowy.

> ⚠️ "Szybki, tani model" wewnątrz hooka to nie jest coś, co dzieje się samo. Hook to zwykły skrypt powłoki, nie ma automatycznie dostępu do Twojej sesji Claude Code. W praktyce są dwie drogi: albo skrypt wywołuje `claude -p "..."` (reużywa Twojego istniejącego logowania/subskrypcji, bez dodatkowego kosztu), albo używa osobnego klucza API (dodatkowa, płatna droga, poza subskrypcją). Prompt poniżej celowo nie przesądza który wariant, powiedz Claude Code, żeby wybrał `claude -p`, jeśli zależy Ci na tym, żeby nie było to nic dodatkowo płatne.

> 📌 **Myśl już teraz o Kroku 4.** Jeśli hook dokleja kolejne wymiany pod jednym, stałym nagłówkiem "automatycznie przechwycone" bez osobnego podnagłówka na wpis, to chunker z Kroku 4 (dzieli pliki wg `##`/`###`) może skleić cały ruchliwy dzień w jeden gruby fragment zamiast osobnych, precyzyjnie wyszukiwalnych wpisów. Przy małym dzienniku tego nie zobaczysz, ujawni się dopiero przy wielu wymianach dziennie, i to bez żadnego błędu po drodze, po prostu gorszą precyzją wyszukiwania. Każ hookowi dawać każdemu wpisowi własny podnagłówek z czasem (np. `### 14:32`), nie tylko jedną wspólną sekcję na cały dzień.

```
Dodaj hook Stop w Claude Code, który wyciąga ostatnią wymianę
użytkownik/asystent z transkryptu sesji, streszcza ją w kilku
punktach w trzeciej osobie przy pomocy szybkiego, taniego modelu
(wywołanego przez `claude -p`, nie przez osobny klucz API, chyba
że wyraźnie powiem inaczej), i dopisuje do dzisiejszego pliku
dziennika w wyraźnie oznaczonej sekcji "automatycznie przechwycone",
każdy wpis pod własnym podnagłówkiem z godziną (np. "### 14:32"),
nie wszystkie zbite pod jednym wspólnym nagłówkiem na cały dzień.

Zapis ma być idempotentny: policz hash źródłowej wymiany i pomiń
zapis, jeśli już istnieje.

Uruchamiaj hook w tle (fire-and-forget), żeby nigdy nie blokował
ani nie spowalniał zamknięcia sesji.

Dodatkowo archiwizuj surowy transkrypt do folderu wykluczonego
z gita (.gitignore).
```

---

## Krok 4: Wyszukiwanie semantyczne

**Co robimy:** wyszukiwanie po znaczeniu, nie tylko po dokładnych słowach. Pytanie "co ustaliliśmy w sprawie płatności" ma znaleźć rozmowę, w której padło słowo "Stripe", nawet jeśli go nie użyjesz w pytaniu. To najczęstsza przyczyna, dla której systemy pamięci oparte tylko na słowach kluczowych zawodzą.

> ⚠️ Ten krok wymaga wybrania modelu embeddingów: albo lokalny (np. sentence-transformers, działa offline, nic nie kosztuje), albo przez API (np. OpenAI lub Voyage, kosztuje grosze za zapytanie, ale prostsze w konfiguracji). Jeśli nie masz preferencji, zapytaj Claude Code o rekomendację przy realizacji tego kroku, nie jest to rozstrzygnięte w prompcie poniżej celowo, bo zależy od Twojego środowiska.

> ⚠️ Sam skrypt leżący na dysku nikogo nie przypomni. Bez wyraźnej instrukcji Claude Code nie będzie wiedział, że ma go uruchamiać, gdy zapytasz o coś z przeszłości, dlatego prompt poniżej każe też dopisać to wprost do pliku instrukcji projektu (CLAUDE.md).

> ⚠️ Ten krok potrzebuje uruchomionego środowiska do wykonywania skryptów (najczęściej Python albo Node.js) i paru bibliotek do tego celu (baza wektorowa, ewentualnie model embeddingów). Jeśli nie masz żadnego z nich zainstalowanego, nie musisz nic robić z góry: powiedz o tym Claude Code przy wklejaniu promptu, a zainstaluje je sam przez Bash. To po prostu dodatkowe minuty przy tym konkretnym kroku, nie coś blokującego.

```
Skonfiguruj lokalną, wbudowaną bazę wektorową dla tego projektu,
plus spójny model embeddingów (jeśli kiedyś go zmienisz, trzeba
będzie przeembedować wszystko od nowa).

Napisz skrypt indeksujący, który dzieli pliki dziennika na
fragmenty według naturalnych podziałów (nagłówki, sesje), embeduje
każdy fragment i zapisuje razem z metadanymi (plik źródłowy, data,
nagłówek).

Napisz skrypt wyszukujący, który uruchamia równolegle wyszukiwanie
wektorowe i słowo-kluczowe, łączy wyniki, i re-rankuje je według
świeżości oraz wagi źródła.

Na koniec dopisz do CLAUDE.md (lub odpowiednika instrukcji
projektu) krótką zasadę: gdy użytkownik pyta o coś z przeszłości
(decyzję, ustalenie, preferencję sprzed jakiegoś czasu), najpierw
uruchom ten skrypt wyszukujący, zanim odpowiesz z pamięci
rozmowy albo przyznasz, że czegoś nie wiesz.
```

---

## Krok 5: Cytowanie źródła

**Co robimy:** gdy Claude coś przypomina, ma podać dokładne słowa, mniej więcej kiedy padły i skąd pochodzą, nie pewnie brzmiącą parafrazę. Jeśli nic pasującego się nie znajdzie, ma to powiedzieć wprost zamiast zmyślać.

> 💡 To ma największe znaczenie, gdy pamięć nie jest Twoją własną: decyzja współpracownika, preferencja klienta. Niedokładne przypomnienie jest do zaakceptowania, gdy odświeżasz własną pamięć, jest ryzykowne, gdy dotyczy kogoś innego.

```
Zaktualizuj wyniki wyszukiwania tak, żeby każdy zwrócony fragment
zawierał plik źródłowy, datę i nagłówek obok treści, tak żeby
odpowiedzi mogły cytować dokładnie skąd pochodzi dany fakt, zamiast
parafrazować bez wskazania źródła.
```

---

## Krok 6 (opcjonalny): Import historii sprzed instalacji

**Co robimy:** w dniu włączenia tego systemu masz już miesiące historii sesji Claude Code leżące bezużytecznie w logach. Ten krok wciąga je jednorazowo, żeby dzień pierwszy nowego systemu pamięci nie zaczynał się od zera.

> ⚠️ Ten prompt celowo nie mówi "tak jak w Kroku 3" ani "tak jak w Kroku 4". Te numery nic nie znaczą dla Claude Code, jeśli między poprzednimi krokami a tym wkleiłeś coś innego, zamknąłeś sesję, albo rozmowa się skompresowała. Zamiast tego prompt każe Claude Code samodzielnie odnaleźć już istniejące pliki w projekcie, a jeśli ich nie znajdzie, zatrzymać się i zapytać, zamiast zgadywać.

```
Zanim zaczniesz, sprawdź w tym projekcie: (a) czy istnieje hook
Stop, który streszcza sesje do plików dziennika, i (b) czy istnieje
skrypt indeksujący te pliki do wyszukiwania semantycznego. Jeśli
któregoś brakuje, zatrzymaj się i powiedz mi, którego, zamiast
zgadywać jak miałby wyglądać.

Jeśli oba istnieją: napisz jednorazowy skrypt importujący, który
przechodzi przez moją dotychczasową historię sesji Claude Code,
wyciąga sensowne fragmenty dokładnie tą samą metodą co ten
istniejący hook Stop (to samo streszczanie, ten sam format zapisu
do plików dziennika), a następnie indeksuje je tym samym istniejącym
skryptem indeksującym.

Zabezpiecz skrypt plikiem-znacznikiem (sentinel file), żeby
uruchomił się tylko raz.
```

---

## Po uruchomieniu wszystkiego: co dalej

Wklejenie promptów to początek, nie koniec. Zanim zaczniesz na tym polegać w realnej pracy, warto przejść przez poniższe punkty, w tej kolejności.

**1. Sprawdź, czy faktycznie działa.** Store, inject i recall są od siebie niezależne: jak coś nie działa, to konkretna warstwa wymaga poprawki, nie cały system.

- **Inject:** zacznij nową sesję i zapytaj "nad czym ostatnio pracowaliśmy?", bez podpowiadania. Odpowiedź powinna wynikać z samej migawki, bez szukania.
- **Store:** powiedz "zapamiętaj, że [jakiś fakt]", zakończ sesję, zacznij nową. Fakt powinien tam być, bez przypominania.
- **Recall:** zapytaj o coś sprzed tygodni, innymi słowami niż użyłeś oryginalnie. Powinno to znaleźć i powiedzieć, skąd to wzięło.

> 💡 Zanim przetestujesz na prawdziwych ustaleniach, zrób "sztuczny" test: powiedz coś łatwo rozpoznawalnego jako testowe (np. "zapamiętaj, że nasz kolor testowy to fioletowy"), sprawdź czy wraca w nowej sesji, potem usuń to tą samą metodą ("zapomnij o kolorze testowym"). Bezpieczniej niż od razu ufać systemowi na realnych danych.

Pełniejszy zestaw, oparty na tym, co realnie wychodziło na jaw przy budowie tego systemu:

- **Nadpisanie, nie duplikat:** zapamiętaj coś, potem zapamiętaj sprzeczną wersję tego samego faktu ("jednak zmieniłem zdanie, X to teraz Y"). W pliku ma zostać jeden, aktualny wpis, nie dwa obok siebie.
- **Limit i konsolidacja:** dodaj kilkanaście krótkich, jawnie testowych wpisów pod rząd, zbliżając się do limitu znaków. Sprawdź, czy skill konsoliduje istniejące wpisy zamiast urwać plik w pół zdania albo zignorować limit.
- **Uczciwe "nie wiem":** zapytaj o coś, czego na pewno nigdy nie było w tym projekcie. Odpowiedź ma wprost przyznać, że nic nie znaleziono, nie zmyślić wynik.
- **"Zmieniłem zdanie" na prawdziwych danych, nie syntetycznych:** to samo co punkt wyżej, ale sprawdzone przez recall po fakcie ("jaki jest teraz X?"), żeby zweryfikować, że świeższe ustalenie wygrywa w wyszukiwaniu, nie tylko w working-memory.
- **Kontrola kosztu:** poproś Claude Code o podsumowanie, ile razy dotąd odpalił się hook Stop i jaki jest łączny szacowany koszt, zwłaszcza po kilku dłuższych sesjach.

**2. Sprawdź .gitignore, zanim cokolwiek stąd trafi na GitHuba.** Krok 3 każe archiwizować surowe transkrypty do folderu wykluczonego z gita, ale sam `memory/working-memory.md`, dzienne logi i baza wektorowa z Kroku 4 domyślnie NIE są wykluczone. Jeśli ten projekt jest (albo kiedyś będzie) publicznym albo współdzielonym repozytorium, streszczenia rozmów mogą zawierać rzeczy, których tam nie chcesz: dane klienta, klucze, prywatne ustalenia. Poproś Claude Code wprost: "sprawdź, co z folderu memory/ i zaimportowanej historii powinno trafić do .gitignore, zanim to gdziekolwiek wypchniemy".

**3. Przeczytaj i dostosuj reguły w skillu z Kroku 2.** Prompt celowo każe zapisać je jako czytelne instrukcje, nie sztywny kod, właśnie po to, żebyś mógł przejrzeć, co agent uznaje za "warte zapamiętania", i poprawić, jeśli się z tym nie zgadzasz.

**4. Krok 6 (import historii) rób jako ostatni, i tylko raz.** To jedyny krok trudny do odwrócenia bez ręcznego czyszczenia, plik-znacznik ma go zablokować przed powtórnym uruchomieniem, ale warto się upewnić, że faktycznie powstał, zanim uznasz temat za zamknięty.

**5. Po pierwszym tygodniu realnego użycia, zerknij na working-memory.md.** Sprawdź, czy limit znaków rzeczywiście działa (plik nie urósł bez ograniczeń) i czy konsolidacja z Kroku 2 usuwa właściwe rzeczy, a nie te, które chciałeś zachować.

---

## Wiele projektów: kopiowanie do nowego folderu

Chcesz tę samą pamięć w osobnym projekcie/kliencie, nie mieszając tematów? **Nie kopiuj folderu `memory/` 1:1**, to skopiowałoby też treść (Twoje realne ustalenia, dziennik, indeks) do nowego miejsca, czyli zmieszałoby tematy zamiast je rozdzielić. Kopiuje się maszynerię, dane zaczynają się od zera.

**1. Utwórz nowy, pusty folder projektu.**

**2. Z poziomu istniejącego projektu (tego z gotową pamięcią) poproś Claude Code:**

```
Skopiuj do folderu [pełna ścieżka do nowego projektu] następujące
elementy z tego projektu, zachowując strukturę:
- .claude/hooks/ (cała zawartość)
- .claude/skills/zapisz-do-pamieci/
- .claude/settings.json
- scripts/ (cała zawartość, łącznie z lib/)
- wpisy w .gitignore dotyczące memory/, node_modules, .cache
- regułę w CLAUDE.md o wyszukiwaniu przed odpowiedzią na pytania
  o przeszłość (samą regułę, nie resztę pliku, jeśli tam coś już jest)

NIE kopiuj folderu memory/, nowy projekt ma zacząć z pustą pamięcią.
```

> 💡 `node_modules/` i `.cache/` (pobrany model embeddingów) możesz skopiować razem ze skryptami, jeśli chcesz oszczędzić czas: to nie są dane konkretnego tematu, tylko biblioteki i wagi modelu, identyczne niezależnie od projektu, i (przy tym modelu, bez natywnych bindingów) przenośne między folderami na tym samym komputerze.

**3. W nowym folderze, w nowej sesji Claude Code, wklej:**

```
Zainstaluj zależności (npm install), jeśli node_modules nie zostało
skopiowane. Stwórz od zera folder memory/ z pustym working-memory.md
(sekcje: aktywne wątki, notatki warte zachowania, oczekujące decyzje)
i dzisiejszym plikiem dziennika, dokładnie jak w Kroku 1 przewodnika.
Sprawdź, czy hook SessionStart i skill się widzą, jeśli nie,
przypomnij mi, że może być potrzebna kolejna nowa sesja.
```

**4. Test:** zamknij tę sesję, otwórz nową w nowym folderze, zapytaj "nad czym ostatnio pracowaliśmy?". Powinna wrócić pusta/świeża odpowiedź, nie treść ze starego projektu, to potwierdza, że tematy faktycznie się nie mieszają.

---

## Źródło i inspiracja

Ten przewodnik powstał na bazie darmowego planu Agentic Academy ("Claude Code Memory Plan") i towarzyszącego mu wideo: [I Rebuilt Hermes's Best Feature in Claude Code (Steal This)](https://www.youtube.com/watch?v=9CiOwbmOKdU). To była główna inspiracja, ale sama treść tego pliku, w tym prompty, opisy problemów i wszystkie poprawki (per-turn zamiast per-sesja przy hooku Stop, ryzyko rekurencji przy zagnieżdżonym `claude -p`, podnagłówki na wpis dla chunkera, odwołania przez pliki zamiast numery kroków, i reszta) powstały z realnego przetestowania tego na żywo w Claude Code, nie z samego materiału źródłowego.

---

## Autor

**Adam Kopeć**: [friendlyai.pl](https://www.friendlyai.pl/) · [YouTube](https://www.youtube.com/@Friendly_AI_PL) · [adam@friendlyai.pl](mailto:adam@friendlyai.pl)
