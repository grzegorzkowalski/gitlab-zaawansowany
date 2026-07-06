# Warsztat 2 · Organizowanie projektów za pomocą grup i przestrzeni nazw

**Cel:** przenosisz `licznik-app` z grupy szkoleniowej do własnej struktury grup i przechodzisz z uprawnień projektowych na grupowe — tak, jak organizuje się GitLaba w firmie.

**Czas:** ok. 20 min · **Wymagania:** ukończony Warsztat 1 (projekt `licznik-app`, dwóch recenzentów).

> ✅ Na końcu każdego kroku jest **checkpoint** — nie idź dalej, dopóki się nie zgadza.

---

## Krok 1. Utwórz własną grupę i podgrupę

1. Menu → **Groups → New group → Create group**:
   - **Group name:** `firma-<twoje-inicjaly>` (np. `firma-jk`)
   - **Visibility:** *Private*
2. Wejdź do nowej grupy i utwórz podgrupę: **New subgroup**:
   - **Subgroup name:** `produkty`
   - **Visibility:** *Private*

> ✅ **Checkpoint:** istnieje struktura `firma-xx/produkty` (pusta).

---

## Krok 2. Przenieś `licznik-app` do swojej struktury

1. Projekt `licznik-app` → **Settings → General** → rozwiń **Advanced** → sekcja **Transfer project**.
2. Wybierz namespace `firma-xx/produkty` i potwierdź (przepisz nazwę projektu).
3. **Przeczytaj ostrzeżenie GitLaba** przed potwierdzeniem — zanotuj, co się zmieni (wrócimy do tego na omówieniu).

> ✅ **Checkpoint:** projekt jest widoczny pod adresem `gitlab.com/firma-xx/produkty/licznik-app`.

---

## Krok 3. Napraw lokalny remote

Adres repozytorium właśnie się zmienił — Twoja lokalna kopia wskazuje jeszcze stary.

```bash
git remote set-url origin git@gitlab.com:firma-xx/produkty/licznik-app.git
git pull
```

> ✅ **Checkpoint:** `git pull` kończy się bez błędu (`Already up to date.`).

---

## Krok 4. Sprawdź adres GitLab Pages

1. Projekt → **Deploy → Pages** — sprawdź adres strony. GitLab.com domyślnie używa **unikalnej domeny** (opcja *Use unique domain*), więc adres ma postać:

```
https://licznik-app-224fe9.gitlab.io
```

— sama domena z losowym sufiksem, **bez ścieżki** namespace. Przy wyłączonej unikalnej domenie adres zawierałby pełną ścieżkę: `https://firma-xx.gitlab.io/produkty/licznik-app/`.

2. Uruchom pipeline, żeby odświeżyć publikację po transferze: **Build → Pipelines → New pipeline → Run pipeline** (gałąź `main`).
3. Po zielonym pipelinie otwórz adres — licznik działa.

> 📝 **Zanotuj swój adres Pages** — będzie potrzebny w Warsztacie 6 (dokumentacja).

> ✅ **Checkpoint:** aplikacja działa pod nowym adresem Pages.

---

## Krok 5. Dostęp przez grupę zamiast przez projekt

W Warsztacie 1 zapraszałeś recenzentów **do projektu**. Teraz zrobisz to na poziomie **grupy** — z dziedziczeniem w dół.

1. Grupa `firma-xx` → **Manage → Members → Invite members** → dodaj swoich **dwóch recenzentów** z rolą **Developer**.
2. Wejdź w `licznik-app` → **Manage → Members**. Znajdź recenzentów na liście i spójrz na kolumnę **Source** — widnieje tam grupa `firma-xx`, nie projekt. To jest **dziedziczenie członkostwa**.
3. Usuń stare, projektowe zaproszenia tych osób (ikonka usunięcia przy wpisach ze źródłem *Direct member*). Dostęp pozostaje — dziedziczy się z grupy.
4. Poproś recenzenta o potwierdzenie, że nadal widzi Twój projekt.

> ✅ **Checkpoint końcowy:** recenzenci są członkami `licznik-app` ze źródłem = grupa `firma-xx`; bezpośrednich (projektowych) wpisów już nie ma.
