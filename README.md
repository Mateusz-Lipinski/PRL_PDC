# PRL_PDC

Działanie układu aktywnej kompensacji dryftów opisałem w swojej pracy magisterskiej. Na płytce dodane są jeszcze regulowane tłumiki przed przesuwnikami fazy, do stabiizacji mocy.

Przesuwniki fazy i tłumiki są sterowane ze wzmacniaczy sumujących 2 sygnały z DAC z różnymi wzmocnieniami, co pozwala na sterowanie zgrubne i precyzyjne, w zakresie 0 - 12V. Wzmacniacze zostały wymienione na wszystkich płytkach i w projekcie, bo poprzednie wymagały symetrycznego zasilania. Innych błędów w projekcie nie znalazłem.
Z tego co wiem to rozwiązanie działa całkiem dobrze. Jak to działa można zobaczyć w symulacji w ./symulations.

Przestrajanie w tak szerokim zakresie jest potrzebne, żeby ustawić punkt stabilizacji w odpowiednim miejscu na charakterystyce detektorów (chyba najlepiej w połowieu)
Wskazania detektorów PD1 i PD2, oraz PD3 i PD4 są ze sobą powiązane, więc napisałem algorytm który przeszukuje charakterystykę i minimalizuje błąd średniokwadratowy, tak żeby oba detektory pracowały jak najbliżej środka charakterystyki. Działa to chyba w miarę ok, ale zdażyło się raz czy dwa że ustawił się taki punkt pracy że sterowanie się nasyciło - zależy od rzeczy podłączonych do układu.

Zauważyłem problem, polegający na tym, że przy przestrajaniu przesuwnika od jednego kanału (na jednej częstotliwości), zmieniaja się również wskazania detektorów na drugiej częstotliwości. Problem występował w prototypie, ale zwarcie wejść sterujących przesuwnikow (mają po 2 wejścia BIAS) wydawało się rozwiązać problem (nie wiem z czego to wynika, tymbardziej że piny wydają się byc zwarte w środku scalaka.
Ja podejrzewam że filtry są za słabe i detektory zbierają też drugą częstotliwość, co wpływa na pomiar fazy. Żeby to potwierdzić trzeba by niestety ciąć ścieżki i dołożyć dodatkowe filtry przed detektorami fazy. Można było zrobić miejsce na to na etapie projektu, ale z jakiegoś powodu podjęliśmy decyzję żeby tego nie robić.
(Minimializacja strat?)

W związku z powyższym, pętle stabilizacji fazy włączane pojedynczo wydają się działać, ale razem - zakłócają się nawzajem. W prototypie powodowało to że dryf fazy nie był kompensowany i wręcz ulegał wzmocnieniu.

Gdy próbowałem uruchomić stabilzację mocy, również występował problem zakłócania się pętli. Zmiana tłumienia powoduje zmianę fazy, a fazy zmianę tłumienia. Przez to obie pętle wariowały. W tym wypadku to jest raczej kwestia nastaw PID.

Stabilizacja temperatury działa, ale nie bardzo daje się schłodzić ten układ, nawet z radiatorem. Stabilizowałem na 35\*C - to trochę dużo. Nie wiem jakie warunki będą w docelowym racku. Precyzja stabilizacji jest bardzo dobra moim zdaniem.

Przy odkręcaniu kabli urywają się złącza SMA - lutowanie chyba pomaga.

Mikrokontroler programuje się przez wewnętrzne złącza SWD za pomocą ST-Link, albo przez uart ze złącza USB na frontpanelu.

Do działania programowania przez uart, wymagane jest założenie zaślepki z kondensatorem i rezystorem na złącze SWD na płytce.
Zaślepkę wymyśliłem niedawno, bo pierwszy raz mam doczynienia z tym bootloaderem. Ale działa. Schemat jest w plikach projektowych obok złącza SWD (RF.schdoc).
Bootloader uart jest aktywny jeśli NAJPIERW podłączymy zasilony kabel USB, a dopiero potem włączymy zasilanie systemu. Więcej na ten temat w readme repozytorim z kodem.

Ze strony mikrokontrolera jest zaprogramowane:
DAC (Wartości ustawiane przez makro),
ADC (Wartości odczytywane ~ co sekundę na przerwaniach) ,
Stablizacja temperatury,
Przeliczanie odczytów z ADC na wartości fizyczne,
Komunikacja po uart (proste komendy),
Optymalizacja punktu pracy detekrorów opisana wyżej (ta funkcja trwa kilka sekund).

Nie są zaprogramowane:
Komunkacja z modułem ethernet przez i2c
Stabiliaja mocy

Dodatkowo nie działa kwarc zegarkowy (pewnie kwestia dobrania kondensatorków), ale nie jest w zasadzie potrzebny, dodałem go w razie czego jakby była potrzebna jakaś lepsza referencja czasu.

Więcej w readme w repozytorum PRL_PDC_Source
