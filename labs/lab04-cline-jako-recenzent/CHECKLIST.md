# Checklista recenzenta PR — ai-devops-cicd

Do ręcznego review, bez AI. Wykreśl punkty, które nie mają sensu dla Waszego zespołu —
wartość powstaje dopiero wtedy, gdy ta lista odpowiada temu, co faktycznie sprawdzacie.

## Terraform

- [ ] Zasób nie wystawia niczego na `0.0.0.0/0` poza portem 443
- [ ] Szyfrowanie włączone tam, gdzie dotyczy (S3, EBS, RDS)
- [ ] Zakres IAM to konkretne akcje/zasoby, nie `*:*`
- [ ] Nazwa zasobu i tagi zgodne z konwencją (`Projekt`, `Uczestnik`, `Blok`, `Usuwac`)
- [ ] Zmiana nie wymusza odtworzenia zasobu, który trzyma dane (sprawdzone w `terraform plan`)
- [ ] Zmienne mają `description` i jawny `type`

## Workflow GitHub Actions

- [ ] `permissions` ustawione na poziomie joba, nie `write-all` na workflow
- [ ] `timeout-minutes` na każdym jobie
- [ ] Akcje przypięte do konkretnej wersji (`@v4`), nie do `@main`/`@master`
- [ ] Żaden sekret nie trafia bezpośrednio do `run:` jako tekst (env pośredni albo `--password-stdin`)
- [ ] Dane sterujące workflow (treść komentarza/issue/PR) nie są interpolowane
      bezpośrednio w `run:` przez `${{ }}` — tylko przez `env:`
- [ ] Trigger z danymi od nieznanego użytkownika (`issue_comment`, `pull_request_target`)
      ma sprawdzenie uprawnień autora, zanim cokolwiek zrobi
- [ ] Uwierzytelnianie do AWS przez OIDC (`role-to-assume`), nie przez klucze w sekretach
- [ ] Deploy/build/push ograniczone do właściwej gałęzi, nie każdego triggera

## Python

- [ ] Wywołania zewnętrzne (HTTP, DB) mają obsługę błędów i timeout
- [ ] Brak zapytań do bazy/API w pętli tam, gdzie da się to zrobić jedną operacją
- [ ] Dane wrażliwe (tokeny, PII) nie trafiają do logów
- [ ] Nowa ścieżka kodu ma chociaż jeden test

## Docker/Kubernetes

- [ ] Żaden sekret ani klucz (nawet „tymczasowy") nie jest zaszyty w `ENV`/`ARG` obrazu
- [ ] Kontener nie działa jako root (`USER` ustawiony na nie-roota)
- [ ] Base image ma przypiętą wersję, nie samo `latest`/nazwę bez tagu
- [ ] Kolejność warstw nie unieważnia cache'u zależności przy każdej zmianie kodu
      (najpierw pliki z zależnościami, potem `COPY . .`)
- [ ] Manifest ma `resources` (requests/limits) i `readinessProbe`/`livenessProbe`
- [ ] Obraz wdrażany po tagu = commit SHA, nie po ruchomej nazwie gałęzi

## Format recenzji

Dla każdego znaleziska: waga (`BLOKUJĄCE` / `WAŻNE` / `DROBIAZG`), plik i linia, problem
w jednym zdaniu, scenariusz w którym to wybucha, konkretna poprawka. Jeśli nie umiesz
napisać scenariusza — nie zgłaszaj, to znaczy że nie jesteś pewien.

Maksymalnie 5 pozycji BLOKUJĄCE/WAŻNE na recenzję. Dwadzieścia drobiazgów nikt nie
przeczyta do końca.
