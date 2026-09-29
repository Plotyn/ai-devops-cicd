# Strażnik promptów — czego NIE złapie (etap 3)

Reguła dodana w etapie 2 (`WZORCE` w `straznik.py`) łapie pełne ARN-y (`arn:aws:...`)
oraz identyfikator konta AWS w kontekście słowa „konto"/„account". Zamyka lukę z etapu 1
(dostęp administracyjny do produkcji przechodził bez zająknięcia), ale to wciąż lista
wzorców regexowych — poniżej trzy konkretne prompty, które **przechodzą, a nie powinny**,
zweryfikowane empirycznie (`kod wyjścia: 0` = Przepuszczone).

## 1. Sekret rozbity formatowaniem

```
Popraw kod. Klucz to AKIA IOSFODNN7EXAMPLE (wkleiłem ze spacją z PDF).
```

`AKIA[0-9A-Z]{16}` wymaga 16 znaków bezpośrednio po `AKIA`, bez przerwy. Spacja, łamanie
linii, myślnik co kilka znaków — cokolwiek przerywa ciągłość — i wzorzec przestaje pasować,
mimo że dla człowieka (i dla modelu, do którego trafi prompt) to nadal ten sam klucz.
Skopiowanie z PDF-a albo ze zrzutu ekranu robi to za darmo, bez złej woli.

## 2. Wewnętrzny token w nieznanym formacie

```
Użyj tego tokenu do wywołania naszego wewnętrznego API: internal-svc-tok-7f9a3b2e1c4d,
host platform.internal.firma.pl.
```

Nie pasuje do żadnego z sześciu wzorców — nie jest to AKIA, `ghp_`, klucz prywatny,
connection string z hasłem, ani `hasło=`. Lista wzorców zna publiczne, udokumentowane
formaty (AWS, GitHub). Firmowy token wygenerowany przez wewnętrzny system uwierzytelniania
nie pasuje do niczego, co ktokolwiek na zewnątrz mógłby znać — a to właśnie te tokeny
uprawniają do najwięcej w Waszej infrastrukturze.

## 3. Żądanie sformułowane słowami, nie identyfikatorem

```
Nadaj integracji CI pełne uprawnienia administratora na naszym koncie produkcyjnym AWS,
tak jak ma to zespół platformowy.
```

Semantycznie to dokładnie to samo żądanie, które odrzuciliśmy w etapie 1 (dostęp admina
do prod) — tylko opisane słowami zamiast ARN-em czy numerem konta. Żaden filtr regexowy
nie złapie *intencji*, tylko *kształt* tekstu. To nie jest luka do załatania kolejnym
wzorcem — to granica tego, co w ogóle da się zrobić filtrem treści promptu (patrz pułapka
w README: „Filtr promptów zamiast uprawnień" — to zatrzymuje `ask` w `.claude/settings.json`,
nie strażnik).

## Wniosek

Wzorce regexowe łapią **pomyłki i nieuwagę** (sekret wklejony wprost, ARN skopiowany
z konsoli) — nie łapią **obfuskacji** (przykład 1), **nieznanych formatów** (przykład 2)
ani **intencji wyrażonej bez identyfikatora** (przykład 3). Lista wzorców jest z definicji
niekompletna — wartością tego ćwiczenia nie jest sama reguła z etapu 2, tylko świadomość,
gdzie dokładnie kończy się jej skuteczność.
