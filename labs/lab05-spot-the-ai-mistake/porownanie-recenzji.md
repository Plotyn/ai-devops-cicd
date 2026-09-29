# PR #47 — porównanie recenzji (do dyskusji w grupie)

Metoda: Część A — świeży agent (bez pamięci, bez dostępu do README/podpowiedzi tego labu,
metoda trzech przebiegów jak `/pr-review`) dostał tylko `PR-opis.md`, `ranking.py`,
`iam-ranking.tf` i ogólny prompt „Zrecenzuj ten pull request." Część B — recenzja własna,
z pełną wiedzą o trzech zamierzonych błędach.

## Trzy zamierzone błędy (klucz)

1. **Bezpieczeństwo** — `iam-ranking.tf:20-29`. Statement `OpisTabel` daje
   `dynamodb:DescribeTable`, `dynamodb:ListTables`, `dynamodb:Scan` z `Resource = "*"`.
   `ListTables` faktycznie wymaga `"*"` (akcja na poziomie konta) — to nie jest błąd samo
   w sobie. Ale `DescribeTable` i zwłaszcza `Scan` da się i trzeba zawęzić do
   `aws_dynamodb_table.stats.arn`, tak jak w pierwszym bloku. `Scan` czyta pełne dane
   tabeli, nie metadane — aplikacja do wyświetlania cytatów dostaje furtkę do odczytu
   dowolnej tabeli DynamoDB w koncie.

2. **Wydajność** — `ranking.py:29-30`. `zbuduj_ranking` woła `pobierz_licznik` osobno dla
   każdego cytatu = jedno zapytanie `GetItem` na cytat. Przy 4 cytatach niezauważalne,
   przy 1000 to 1000 sekwencyjnych zapytań do DynamoDB zamiast jednego/kilku
   `batch_get_item`. Błąd widać dopiero, gdy połączy się pętlę z tym, co robi
   `pobierz_licznik` — każda funkcja osobno wygląda poprawnie.

3. **Styl** — `ranking.py:43`. `zwieksz_licznik(quoteId: str)` łamie konwencję całego
   pliku (wszędzie indziej `quote_id`, snake_case) — zero wpływu funkcjonalnego, czysta
   niespójność.

## Część A — co znalazł ślepy agent

**Zgłoszone jako BLOKUJĄCE (4 pozycje):**

| # | Znalezisko | Ocena po weryfikacji |
|---|---|---|
| 1 | Rozjazd nazwy tabeli (`quotes-stats` vs `${prefix}-quotes-stats`) | **Prawdopodobnie fałszywy alarm** — to tylko wartość domyślna `os.getenv`; agent nie widział, jak `STATS_TABLE` jest realnie ustawiane w deploymencie |
| 2 | `zwieksz_licznik` nigdzie nie wywoływane w pokazanym kodzie | **Prawdopodobnie fałszywy alarm** — to właśnie ta jedna linijka w `main.py`, którą lab celowo pominął jako nieistotną; agent nie mógł o tym wiedzieć |
| 3 | `DescribeTable`/`Scan` z `Resource="*"` zamiast ARN-u tabeli | **Prawdziwy błąd #1 (bezpieczeństwo)** — trafione |
| 4 | Brak testów dla nowej logiki | Zasadna uwaga, poza zakresem trzech zamierzonych błędów |

**Zgłoszone jako niżej priorytetowe (zakopane na dole recenzji):**
- N+1 sekwencyjnych `GetItem` w pętli zamiast batch → **prawdziwy błąd #2 (wydajność)**, trafiony, ale zdegradowany
- `quoteId` vs `quote_id` → **prawdziwy błąd #3 (styl)**, trafiony, ale zdegradowany

## Wniosek do dyskusji w grupie

Agent **technicznie znalazł wszystkie trzy zamierzone błędy** — inny wynik niż sugeruje
ostrzeżenie w README labu („agent chętnie raportuje błąd stylistyczny i przechodzi dalej").
Ale popełnił tę samą pułapkę w odwrotną stronę: **prawdziwy błąd bezpieczeństwa i
wydajności wylądowały niżej w priorytecie niż dwa spekulacyjne, niepewne znaleziska**
oznaczone jako blokujące z powodu braku pełnego kontekstu (brak `main.py`, brak wiedzy
o konfiguracji deploymentu).

Ktoś czytający tylko sekcję „Blokujące" i przerywający tam trafiłby na 2 z 4 pozycji,
które przy pełnym kontekście repo prawdopodobnie w ogóle nie są bugami — podczas gdy
realny problem bezpieczeństwa jest wprawdzie obecny (#3), ale wydajność i styl są
zdegradowane do sekcji, którą łatwo pominąć.

**Pytanie na grupę:** czy to, że agent "znalazł" błąd, wystarczy — czy trzeba jeszcze
sprawdzić, czy oznaczył go z priorytetem odpowiadającym realnemu ryzyku, i czy pozycje
oznaczone jako blokujące rzeczywiście są blokujące, czy tylko brzmią groźnie przez brak
kontekstu, jaki miał recenzent.
