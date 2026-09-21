# npp-task-manager

User Defined Language dla Notepad++ do zarządzania zadaniami w plikach `.tsk`.

Nie jest to nowy język programowania — to zestaw reguł kolorowania, który zamienia
zwykły plik tekstowy w czytelną listę zadań z rolowaniem dzień po dniu.

## Format pliku `.tsk`

```
== 2026-09-21
!!007 przygotować dane do audytu // deadline piątek
~~004 zmapować katalog punch-out // created US: ABC-1232
005 wysłać e-mail do klienta
++003 task 3
--002 task 2

== 2026-09-19
...
```

Plik czyta się od góry: najnowszy dzień jest na samej górze.

### Nagłówek dnia

```
== 2026-09-21
```

Prefiks `==` jest **obowiązkowy**. Sam `==` wystarcza, data musi być w formacie `YYYY-MM-DD`.
Nagłówek zjada całą linię, więc myślniki w dacie nie kolidują ze znacznikiem `--`.

### Linia zadania

```
[znacznik]NNN opis zadania // opcjonalny komentarz
```

- `NNN` — numer zadania, ciągły, rosnący wraz z dodawaniem nowych zadań
- znacznik jest opcjonalny; jego brak oznacza zadanie otwarte
- znacznik stoi w kolumnie 1, bezpośrednio przed numerem, bez spacji
- `//` rozpoczyna komentarz do końca linii

### Znaczniki

| znacznik | znaczenie | kolor |
|---|---|---|
| *(brak)* | zadanie otwarte | domyślny, numer szary bold |
| `!!` | important / urgent | czerwony, **bold** |
| `~~` | in progress | bursztynowy, **bold** |
| `??` | waiting for — czekam na kogoś z zewnątrz | niebieski |
| `##` | on hold — wstrzymane przeze mnie | stalowy, *kursywa* |
| `**` | new idea / maybe | fioletowy, *kursywa* |
| `>>` | migrated — wypada z dziennej rotacji, ląduje w backlogu | brązowy, *kursywa* |
| `++` | done | zgaszona zieleń, *kursywa* |
| `--` | won't do / cancelled | jasnoszary, *kursywa* |
| `// …` | komentarz | zielony, *kursywa* |

Zasada kolorów: **jasność = aktualność**. To, co wymaga akcji, jest ciemne i kontrastowe.
To, co zamknięte (`++`, `--`), blaknie w tło. Bold mają tylko `!!` i `~~` — czyli „pali się"
i „tu jestem teraz".

`??` kontra `##`: przy `??` blokuje ktoś inny i trzeba ponaglić, przy `##` wstrzymałeś
zadanie sam i nie wymaga ono ruchu.

### Dlaczego znaczniki są podwojone

Notepad++ UDL nie potrafi zakotwiczyć reguły do początku linii — kolorowanie całej linii
działa na symbolu **gdziekolwiek** w tekście. Pojedynczy `-` zapalałby styl „won't do"
w środku słowa `e-mail` czy `punch-out`, `?` w pytaniu, `>` w strzałce `->`.

Podwojenie znosi te kolizje: `e-mail` ma jeden myślnik, nie dwa. Kosztuje jedno
naciśnięcie klawisza, i tylko przy oznaczaniu zadania.

Uwaga: wewnątrz linii już oznaczonej i wewnątrz komentarza `//` symbole są nieaktywne,
więc `--002 wyślij e-mail` i `// US-1232` są całkowicie bezpieczne. Ryzyko dotyczy
wyłącznie opisu zadania **bez** znacznika.

### Rolowanie dnia

Na początek nowego dnia kopiujesz cały blok poprzedniego dnia na górę pliku, zmieniasz
datę w nagłówku i usuwasz linie `++`, `--` oraz `>>`. Zadania otwarte i wstrzymane
przechodzą dalej z niezmienionymi numerami.

## Instalacja

Są dwa warianty kolorystyczne. **Zainstaluj tylko jeden** — oba rejestrują rozszerzenie
`.tsk` i zainstalowane równocześnie będą się o nie biły.

- `udl/tsk-light.udl.xml` — dla jasnego motywu Notepad++
- `udl/tsk-dark.udl.xml` — dla ciemnego

### Sposób 1 — wrzucenie pliku (zalecany, N++ 7.6+)

Skopiuj wybrany plik do:

```
%APPDATA%\Notepad++\userDefineLangs\
```

i zrestartuj Notepad++.

### Sposób 2 — import z menu

`Language` → `User Defined Language` → `Define your language…` → `Import…`,
wskaż plik XML, zrestartuj Notepad++.

### Weryfikacja

Otwórz `examples/tasks.tsk`. Jeśli kolorowanie nie wskoczyło samo, wybierz je ręcznie:
`Language` → `TSK Tasks (Light)` / `TSK Tasks (Dark)`.

Sprawdź trzy rzeczy:

1. Nagłówek `== 2026-09-21` ma tło i jest pogrubiony.
2. Linia `005 wysłać e-mail do klienta w sprawie US-1232` jest w całości domyślnego
   koloru — żaden myślnik nie zapalił szarości do końca linii.
3. W linii `~~004 … // created US: ABC-1232` komentarz jest zielony, a nie bursztynowy
   jak reszta linii.

Jeśli punkt 3 nie działa, to kwestia zagnieżdżania: `Define your language…` →
zakładka `Operators & Delimiters` → przy każdym Delimiterze 1–8 zaznacz w sekcji
nesting pozycję **Comment**. W plikach XML odpowiada temu atrybut `nesting="256"`.
Bez tego komentarz przejmuje kolor linii — plik nadal jest czytelny, tylko mniej ładny.

## Znane ograniczenia

- **Brak przekreślenia.** Style Notepad++ obsługują wyłącznie bold, kursywę i podkreślenie.
  Zadania `++` i `--` są więc wygaszone kolorem, a nie przekreślone.
- **Ciemny wariant narzuca tło** `#1E1E1E`. Przy mocno odmiennym motywie kolor tła
  nagłówka dnia może się nie zgrywać — podmień `bgColor` w pliku XML.
- **Resztki kolizji.** Podwojenie znaczników przepuszcza `C++`, `Notepad++`, `--save`
  i `-->`, jeśli trafią do opisu zadania **bez** znacznika. Objaw jest natychmiast
  widoczny (ogon linii zmienia kolor), więc wystarczy przeredagować opis.
- **Brak zwijania bloków dnia.** UDL wymagałby jawnego znacznika końca bloku.

## Konfiguracja własnych tagów

Lista `Keywords1` w pliku XML jest pusta i przeznaczona na Twoje słowa kluczowe —
nazwiska, nazwy projektów, stałe etykiety. Wpisz je oddzielone spacjami:

```xml
<Keywords name="Keywords1">jan anna infra billing</Keywords>
```

Żeby były widoczne również w liniach oznaczonych, dopisz do nesting delimiterów
wartość `1024` (`nesting="1280"` zamiast `256`).
