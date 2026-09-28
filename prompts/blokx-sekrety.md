# Blok dodatkowy — sekrety i governance

## Audyt: co w tym repo nie powinno trafić do promptu

```
Przejrzyj to repozytorium i wypisz pliki, których NIE należy wciągać do kontekstu
modelu AI. Dla każdego napisz, co konkretnie by wyciekło.

Sprawdź też historię gita (git log -p --all) pod kątem sekretów, które kiedyś
zostały zacommitowane i usunięte później — usunięcie pliku nie usuwa go z historii.
```

## Policy-as-code

```
Napisz polityki Rego dla Conftest, które odrzucą:

1. workflow GitHub Actions bez bloku permissions
2. workflow z permissions: write-all
3. workflow używający akcji przypiętej do ruchomego taga zamiast do SHA
4. zasób Terraform typu aws_s3_bucket bez szyfrowania lub bez blokady dostępu publicznego
5. security group z 0.0.0.0/0 na porcie innym niż 443

Każda reguła ma mieć komunikat, który mówi programiście, co zrobić — nie tylko,
że coś jest nie tak. Dołóż testy: dla każdej reguły jeden plik, który przechodzi,
i jeden, który jest odrzucany.
```

## Checklista, którą warto mieć nad biurkiem

```
Napisz jednostronicową checklistę „czego nie wklejać do promptu AI" dla zespołu DevOps.
Konkretnie i krótko: kategoria, przykład, co zrobić zamiast tego.
Bez ogólników w rodzaju „zachowaj ostrożność".
```
