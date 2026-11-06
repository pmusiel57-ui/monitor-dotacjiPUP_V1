# Monitor dotacji

Bezobsługowy system, który codziennie o 8:00 sprawdza strony urzędów pracy
i wysyła maila, gdy pojawi się **nowe** ogłoszenie o dotacji
(jednorazowych środkach na podjęcie działalności gospodarczej dla bezrobotnych).

Działa za darmo na GitHub Actions – nie potrzebujesz własnego komputera ani serwera.

## Jak to działa

1. Codziennie rano GitHub uruchamia skrypt `checker.py`.
2. Skrypt pobiera każdą stronę z pliku `urls.txt` i szuka linków oraz nagłówków
   o dotacjach, np. „Nabór wniosków o przyznanie jednorazowo środków na podjęcie
   działalności”, „Dotacje dla bezrobotnych”, „środki na rozpoczęcie działalności”.
3. Porównuje wyniki z bazą `data/monitor.db` (SQLite) i wybiera tylko to, czego jeszcze nie widział.
4. Jeśli jest coś nowego, wysyła mail HTML. Ogłoszenia o naborze dostają zieloną etykietę **NABÓR**.
5. Baza zapisuje się automatycznie w repozytorium, żeby jutro pamiętać, co już było.

Dodatkowo:

- **Raport tygodniowy w poniedziałek.** Jeśli przez tydzień nic się nie pojawiło,
  dostaniesz krótki mail „nic nowego”. Wiesz wtedy, że system działa.
- **Ostrzeżenie o awarii strony.** Gdy strona nie odpowiada 3 dni z rzędu, dostaniesz o tym mail.
- **Odporność na awarie poczty.** Jeśli mail się nie wyśle, ogłoszenia nie przepadną,
  tylko przyjdą przy następnym uruchomieniu.

> Pierwszy mail będzie dłuższy. Pokaże wszystko, co obecnie pasuje na każdej stronie
> (opisane jako „pierwsze sprawdzenie, stan obecny”). Kolejne maile zawierają już tylko nowości.

## Pliki

| Plik | Do czego służy |
|---|---|
| `urls.txt` | lista stron do sprawdzania |
| `checker.py` | skrypt monitorujący |
| `requirements.txt` | biblioteki Pythona (requests, BeautifulSoup) |
| `.github/workflows/monitor.yml` | harmonogram: codziennie o 8:00 i przycisk ręcznego uruchomienia |
| `data/monitor.db` | baza tworzona automatycznie, nie ruszaj jej |

## Instalacja (wszystko w przeglądarce)

1. **Dodaj pliki do repozytorium.** Dla każdego pliku: wejdź na
   `https://github.com/TWOJA-NAZWA/monitor-dotacji/new/main`, wpisz nazwę pliku
   (np. `checker.py` albo `.github/workflows/monitor.yml`), wklej treść i kliknij **Commit changes**.
   Pliki `urls.txt` i `checker.py` zastąp nowymi wersjami: otwórz plik, kliknij ✏️, zamień treść i zapisz.
2. **Usuń stary plik testowy** `.github/workflows/test.yml`: otwórz go, kliknij **⋯ → Delete file**.
   Nie jest konieczne, ale porządkuje zakładkę Actions.
3. **Sekrety:** w **Settings → Secrets and variables → Actions** zostają te same co w teście:
   `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `MAIL_TO`.
   W `MAIL_TO` możesz podać kilka adresów oddzielonych przecinkiem.
4. **Pierwsze uruchomienie:** **Actions → Monitor dotacji → Run workflow**.
   Ręczne uruchomienie działa o każdej porze. Potem system chodzi sam.

## Zmiany i dostosowanie

- **Inne urzędy:** edytuj `urls.txt`, jedna strona w linii, w formacie `Nazwa | adres`.
  Adresy powiatowych urzędów pracy mają zwykle postać `https://MIASTO.praca.gov.pl/`.
  Warto dodawać stronę **aktualności** urzędu, bo tam pojawiają się ogłoszenia o naborach.
- **Słowa kluczowe:** lista `GRANT_PATTERNS` na początku `checker.py`.
- **Dzień raportu tygodniowego:** zmienna `HEARTBEAT_DAY` (0 = poniedziałek … 6 = niedziela, puste = wyłączony).
- **Próg ostrzeżenia o awarii strony:** zmienna `FAIL_ALERT_AFTER` (domyślnie 3).

## Dobrze wiedzieć

- **Opóźnienia.** GitHub czasem uruchamia zadania z harmonogramu z opóźnieniem 5–30 minut.
  Mail może więc przyjść np. o 8:20.
- **Dwa uruchomienia dziennie.** Workflow startuje o 6:00 i 7:00 UTC, żeby trafić w 8:00
  i zimą, i latem. Skrypt sam pilnuje, żeby strony były sprawdzone tylko raz dziennie.
- **Wyłączenie po 60 dniach.** GitHub wyłącza harmonogram w repozytoriach bez aktywności
  przez 60 dni. Codzienny zapis bazy zwykle temu zapobiega. Jeśli dostaniesz od GitHuba
  mail o wyłączeniu, wejdź w Actions i kliknij **Enable workflow**.
- **Strony, które ładują treść skryptem.** Niektóre strony wczytują ogłoszenia przez JavaScript.
  Takich treści skrypt nie zobaczy. Wtedy dodaj do `urls.txt` bezpośredni adres podstrony z aktualnościami.
- **Błąd przy zapisie bazy** (`Permission denied` / `403` w kroku „Zapis bazy”): wejdź w
  **Settings → Actions → General → Workflow permissions**, wybierz **Read and write permissions** i kliknij **Save**.

## Test lokalny (opcjonalnie)

```bash
pip install -r requirements.txt
DRY_RUN=1 FORCE_RUN=true python checker.py   # nie wysyła maila, zapisuje podglad_maila.html
```
