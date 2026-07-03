# Warsztat 1 · Pierwsze kroki w GitLab — od konta do działającego CI/CD

**Cel:** po tym warsztacie masz konto na GitLab, projekt z prostą aplikacją HTML/JS, działający potok CI/CD (build → testy → deploy na GitLab Pages) oraz przećwiczony pełny cykl pracy: branch → commit → merge request → review → pipeline → merge → automatyczny deploy.

**Czas:** ok. 30–40 min · **Wymagania:** przeglądarka, terminal z zainstalowanym `git`, edytor kodu.

> ✅ Na końcu każdego kroku jest **checkpoint** — nie idź dalej, dopóki się nie zgadza.

---

## Krok 1. Załóż konto na GitLab

1. Wejdź na https://gitlab.com/users/sign_up
2. Zarejestruj się (możesz użyć konta Google/GitHub) i potwierdź adres e-mail.
3. Przy pytaniach powitalnych wybierz dowolne odpowiedzi (np. *Just me*, *Software Developer*) — nie mają znaczenia.

**Zgłoś się do prowadzącego:**

4. Otwórz arkusz Google: **https://docs.google.com/spreadsheets/d/1mREU449Nmmm1lJ40Beht3Cf_IqKYrsnDSluVggHdDuI/edit?usp=sharing**
5. Dopisz w nowym wierszu: **imię i nazwisko** oraz swój **username GitLab** (znajdziesz go klikając swój awatar w prawym górnym rogu — zaczyna się od `@`).
6. Prowadzący doda Cię do grupy szkoleniowej z rolą **Developer**. Poczekaj na potwierdzenie — dostaniesz e-mail / zobaczysz grupę w menu **Groups**.

> ✅ **Checkpoint:** widzisz grupę szkoleniową na liście swoich grup (menu po lewej → *Groups*).

---

## Krok 2. Dodaj klucz SSH

Klucz SSH pozwala wypychać kod bez podawania hasła/tokenu przy każdym pushu.

### 2.1. Sprawdź, czy masz już klucz

```bash
ls ~/.ssh/id_ed25519.pub
```

Jeśli plik istnieje — przejdź do 2.3. Jeśli nie:

### 2.2. Wygeneruj nowy klucz

```bash
ssh-keygen -t ed25519 -C "twoj-email@firma.pl"
```

Zatwierdź domyślną lokalizację Enterem. Passphrase możesz zostawić puste (Enter) lub ustawić.

### 2.3. Skopiuj klucz publiczny

```bash
# Linux
cat ~/.ssh/id_ed25519.pub

# macOS (kopiuje od razu do schowka)
pbcopy < ~/.ssh/id_ed25519.pub

# Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub | clip
```

Skopiuj **całą** linię zaczynającą się od `ssh-ed25519 …`.

### 2.4. Dodaj klucz w GitLab

1. GitLab → awatar (prawy górny róg) → **Access** → w menu po prawej **SSH Keys** → **Add new key**.
2. Wklej klucz w pole *Key*, nadaj tytuł (np. `laptop-firmowy`), datę wygaśnięcia możesz zostawić domyślną.
3. Kliknij **Add key**.

### 2.5. Przetestuj połączenie

```bash
ssh -T git@gitlab.com
```

Przy pierwszym połączeniu potwierdź fingerprint wpisując `yes`.

> ✅ **Checkpoint:** widzisz komunikat `Welcome to GitLab, @twoj-username!`

---

## Krok 3. Stwórz projekt

1. GitLab → **+** (górny pasek) → **New project/repository** → **Create blank project**.
2. Wypełnij:
   - **Project name:** `licznik-app` (możesz dodać swoje inicjały, np. `licznik-app-jk`)
   - **Project URL / namespace:** wybierz **GISGL-group** (nie swój osobisty namespace!)
   - **Visibility:** *Private*
   - ✅ zostaw zaznaczone *Initialize repository with a README*
3. Kliknij **Create project**.

> ✅ **Checkpoint:** widzisz stronę projektu z plikiem `README.md`.

---

## Krok 4. Sklonuj repozytorium i dodaj kod aplikacji

### 4.1. Sklonuj repo

Na stronie projektu kliknij **Code** → skopiuj adres z sekcji **Clone with SSH**, następnie:

```bash
git clone git@gitlab.com:GRUPA-SZKOLENIOWA/licznik-app.git
cd licznik-app
```

### 4.2. Dodaj pliki aplikacji

> 📦 **Droga szybka:** rozpakuj **paczkę startową** od prowadzącego (`paczka-startowa-licznik.zip`) tak, aby jej zawartość wylądowała w katalogu sklonowanego repo — i przejdź do 4.3. Listingi poniżej pokazują, co jest w środku (warto je przeczytać!).

Aplikacja to prosty **licznik kliknięć** z logiką wydzieloną do osobnego modułu — dzięki temu da się ją testować.

Utwórz następującą strukturę:

```
licznik-app/
├── src/
│   ├── index.html
│   ├── counter.js
│   └── app.js
├── tests/
│   └── counter.test.js
├── package.json
└── .gitignore
```

**`src/index.html`**

```html
<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Licznik — warsztat GitLab</title>
  <style>
    body { font-family: system-ui, sans-serif; display: grid; place-items: center; min-height: 100vh; margin: 0; background: #f5f2fa; }
    .card { background: #fff; padding: 2.5rem 3rem; border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,.08); text-align: center; }
    #wynik { font-size: 4rem; font-weight: 700; color: #2b2140; margin: 1rem 0; }
    button { font-size: 1.1rem; padding: .6rem 1.4rem; margin: 0 .3rem; border: none; border-radius: 8px; cursor: pointer; background: #e2542b; color: #fff; }
    button.szary { background: #6c6880; }
  </style>
</head>
<body>
  <div class="card">
    <h1>Licznik kliknięć</h1>
    <div id="wynik">0</div>
    <button id="plus">+1</button>
    <button id="minus">−1</button>
    <button id="reset" class="szary">Reset</button>
  </div>
  <script type="module" src="app.js"></script>
</body>
</html>
```

**`src/counter.js`** — logika (czysta funkcja, bez DOM):

```js
// Logika licznika — testowalna, bez zależności od przeglądarki.

export function increment(value) {
  return value + 1;
}

export function decrement(value) {
  // Licznik nie schodzi poniżej zera.
  return Math.max(0, value - 1);
}

export function reset() {
  return 0;
}
```

**`src/app.js`** — podpięcie do interfejsu:

```js
import { increment, decrement, reset } from "./counter.js";

let value = 0;
const wynik = document.getElementById("wynik");

function render() {
  wynik.textContent = value;
}

document.getElementById("plus").addEventListener("click", () => { value = increment(value); render(); });
document.getElementById("minus").addEventListener("click", () => { value = decrement(value); render(); });
document.getElementById("reset").addEventListener("click", () => { value = reset(); render(); });
```

**`tests/counter.test.js`** — testy w oparciu o wbudowany runner Node.js (bez instalowania czegokolwiek):

```js
import { test } from "node:test";
import assert from "node:assert/strict";
import { increment, decrement, reset } from "../src/counter.js";

test("increment zwiększa wartość o 1", () => {
  assert.equal(increment(0), 1);
  assert.equal(increment(41), 42);
});

test("decrement zmniejsza wartość o 1", () => {
  assert.equal(decrement(5), 4);
});

test("decrement nie schodzi poniżej zera", () => {
  assert.equal(decrement(0), 0);
});

test("reset zwraca zero", () => {
  assert.equal(reset(), 0);
});
```

**`package.json`**

```json
{
  "name": "licznik-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "test": "node --test tests/counter.test.js",
    "build": "mkdir -p public && cp -r src/* public/"
  }
}
```

**`.gitignore`**

```
node_modules/
public/
```

### 4.3. Sprawdź lokalnie, że wszystko działa

```bash
node --test tests/counter.test.js   # 4 testy powinny przejść (wymagany Node 18+)
```

Możesz też otworzyć `src/index.html` w przeglądarce i poklikać.

> ✅ **Checkpoint:** `node --test` pokazuje `pass 4 / fail 0`.

---

## Krok 5. Wypchnij kod na `main`

```bash
git add .
git commit -m "Dodanie aplikacji licznika z testami"
git push origin main
```

Odśwież stronę projektu w GitLab.

> ✅ **Checkpoint:** w repozytorium widzisz katalogi `src/` i `tests/`.

---

## Krok 6. Dodaj potok CI/CD

### 6.1. Krótkie wprowadzenie: czym jest `.gitlab-ci.yml`?

Cały proces CI/CD w GitLab definiuje **jeden plik YAML** — `.gitlab-ci.yml` — umieszczony w katalogu głównym repozytorium. GitLab wykrywa go automatycznie: od momentu wypchnięcia pliku **każdy push uruchamia potok** (pipeline). Konfiguracja jest wersjonowana razem z kodem, więc zmiany procesu przechodzą przez review jak każda inna zmiana.

Najważniejsze pojęcia i słowa kluczowe, których zaraz użyjemy:

| Słowo kluczowe | Co robi |
|---|---|
| `stages` | Definiuje **etapy** i ich kolejność. Etapy wykonują się **sekwencyjnie** — jeśli `test` się nie powiedzie, `build` nie wystartuje. |
| *job* (np. `testy:`) | **Zadanie** — nazwana sekcja z poleceniami. Joby w tym samym etapie biegną **równolegle**. Nazwa joba jest dowolna (poza kilkoma zarezerwowanymi, jak `pages`). |
| `image` | Obraz Docker, w którym job się wykonuje. Każdy job dostaje **świeży kontener** — czyste, powtarzalne środowisko. |
| `default` | Ustawienia wspólne dla wszystkich jobów (u nas: wspólny `image`), żeby nie powtarzać ich w każdym jobie. |
| `script` | Lista poleceń shell wykonywanych w kontenerze — serce każdego joba. Niezerowy kod wyjścia = czerwony job. |
| `artifacts` | Pliki **zachowywane po zakończeniu joba** i przekazywane do kolejnych etapów (u nas: katalog `public/` z buildu). Można je też pobrać z interfejsu GitLab. |
| `rules` / `workflow` | **Warunki uruchomienia**: `rules` steruje pojedynczym jobem, `workflow` — całym pipeline'em (kiedy w ogóle ma powstać). |
| Zmienne `$CI_*` | Predefiniowane zmienne GitLab opisujące kontekst, np. `$CI_COMMIT_BRANCH` (nazwa gałęzi) czy `$CI_PIPELINE_SOURCE` (co wywołało pipeline — push, merge request…). |

> 💡 Zanim wypchniesz zmiany w pipeline, możesz sprawdzić składnię bez commitowania: projekt → **Build → Pipeline editor** (walidacja + wizualizacja etapów na żywo).

### 6.2. Nasz potok

Potok wykona trzy etapy — wszystkie **w kontenerze Docker** (obraz `node:20-alpine`):

```
test ──► build ──► deploy (GitLab Pages)
```

Logika warunków, którą za chwilę zapiszemy:

- pipeline powstaje dla **merge requestów** i dla gałęzi **`main`** (sekcja `workflow`),
- joby `testy` i `build` wykonują się zawsze, gdy pipeline istnieje,
- job `pages` (deploy) — **wyłącznie na `main`** (sekcja `rules` w jobie), bo nie chcemy publikować niescalonych zmian.

W katalogu głównym repo utwórz plik **`.gitlab-ci.yml`**:

```yaml
# Każdy job uruchamia się w świeżym kontenerze z tego obrazu
default:
  image: node:20-alpine

stages:
  - test
  - build
  - deploy

# Uruchamiaj pipeline dla merge requestów oraz dla gałęzi main
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"

testy:
  stage: test
  script:
    - node --version
    - npm test

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - public/

# Job MUSI nazywać się "pages", a artefakt MUSI być katalogiem "public"
pages:
  stage: deploy
  script:
    - echo "Publikacja zawartości public/ na GitLab Pages"
  artifacts:
    paths:
      - public/
  rules:
    - if: $CI_COMMIT_BRANCH == "main"   # deploy TYLKO z main
```

Wypchnij plik:

```bash
git add .gitlab-ci.yml
git commit -m "Dodanie potoku CI/CD z deployem na Pages"
git push origin main
```

W GitLab wejdź w **Build → Pipelines** i obserwuj wykonanie. Po zakończeniu (~2–3 min) wejdź w **Deploy → Pages** — znajdziesz tam adres swojej strony, np. `https://gisgl-group.gitlab.io/licznik-app/`.

> 💡 Jeśli strona zwraca 404 zaraz po pierwszym deployu — odczekaj 1–2 minuty, pierwsza publikacja Pages bywa opóźniona. Sprawdź też **Settings → General → Visibility → Pages**, jeśli chcesz udostępnić stronę bez logowania (*Everyone*).

> ✅ **Checkpoint:** pipeline zielony (3 joby: `testy`, `build`, `pages`), a pod adresem Pages działa licznik.

---

## Krok 7. Zabezpiecz gałąź `main`

Skonfigurujemy `main` tak, żeby zmiany trafiały tam **wyłącznie przez merge request z 2 zatwierdzeniami**.

### 7.1. Gałąź chroniona

1. Projekt → **Settings → Repository** → rozwiń **Protected branches**.
2. `main` powinien już być chroniony — ustaw:
   - **Allowed to merge:** *Maintainers* (lub *Developers + Maintainers* — wg wskazań prowadzącego)
   - **Allowed to push and merge:** *No one* ← to wymusza pracę przez MR!

### 7.2. Wymagane zatwierdzenia (approvals)

1. Projekt → **Settings → Merge requests**.
2. W sekcji **Merge request approvals** ustaw regułę: **Approvals required = 2**.
3. Niżej, w **Merge checks**, zaznacz ✅ **Pipelines must succeed** — czerwony pipeline zablokuje merge.
4. Zapisz zmiany.

> ⚠️ **Uwaga licencyjna:** wymuszenie *liczby* zatwierdzeń (approval rules) to funkcja płatna (Premium). Na planie **Free** ustawcie zamiast tego zasadę zespołową: „mergujemy po 2 approve" + *Pipelines must succeed* (to działa na Free). Na szkoleniowej instancji/grupie prowadzący potwierdzi, która opcja jest dostępna.

### 7.3. Test zabezpieczenia

```bash
echo "test" >> README.md
git add README.md && git commit -m "Test ochrony main"
git push origin main
```

> ✅ **Checkpoint:** push został **odrzucony** z komunikatem o chronionej gałęzi. Cofnij lokalny commit: `git reset --hard origin/main`

---

## Krok 8. Zaproś dwóch recenzentów do projektu

Za chwilę Twój merge request będzie wymagał **2 zatwierdzeń** — ale żeby ktoś mógł robić review i klikać *Approve*, musi być **członkiem projektu z odpowiednią rolą**. Dodaj więc do swojego projektu dwie osoby z sali (najlepiej sąsiadów — Ciebie za chwilę dodadzą oni).

1. Ustal z dwiema osobami wymianę: **Ty dodajesz ich, oni dodają Ciebie** (username znajdziecie w arkuszu Google z Kroku 1).
2. Projekt → **Manage → Members** → **Invite members**.
3. Wpisz username pierwszej osoby (np. `@jan-kowalski`), wybierz rolę **Developer**, pole *Access expiration date* zostaw puste.
4. Kliknij **Invite** i powtórz dla drugiej osoby.

Dlaczego rola **Developer**? To minimalna rola, która pozwala **zatwierdzać merge requesty** (*Approve*) i komentować kod w review. *Reporter* widziałby kod, ale nie mógłby zatwierdzić; *Maintainer* byłby nadmiarowy — mógłby zmieniać ustawienia Twojego projektu.

> 💡 Zaproszone osoby mają dostęp **tylko do Twojego projektu** — to uprawnienie na poziomie projektu, niezależne od członkostwa w grupie szkoleniowej.

> ✅ **Checkpoint:** w **Manage → Members** widzisz 2 dodane osoby z rolą *Developer* — i analogicznie Ty widniejesz jako Developer w ich projektach (sprawdź: awatar → *Projects*).

---

## Krok 9. Nowa gałąź + przygotowana modyfikacja

Dodamy nową funkcję: przycisk **+10**. Zmiana obejmuje logikę, test i interfejs.

### 9.1. Utwórz gałąź z `main`

```bash
git checkout main
git pull origin main
git checkout -b feature/przycisk-plus-10
```

### 9.2. Wprowadź zmiany (gotowy kod)

**`src/counter.js`** — dopisz na końcu pliku:

```js
export function incrementBy(value, step) {
  return value + step;
}
```

**`tests/counter.test.js`** — dopisz na końcu pliku:

```js
test("incrementBy zwiększa wartość o podany krok", () => {
  assert.equal(incrementBy(0, 10), 10);
  assert.equal(incrementBy(32, 10), 42);
});
```

…oraz **zmień pierwszą linię importu** w tym pliku na:

```js
import { increment, decrement, reset, incrementBy } from "../src/counter.js";
```

**`src/index.html`** — pod przyciskiem `+1` dodaj:

```html
<button id="plus10">+10</button>
```

**`src/app.js`** — zmień import i dodaj obsługę przycisku:

```js
import { increment, decrement, reset, incrementBy } from "./counter.js";
```

```js
document.getElementById("plus10").addEventListener("click", () => { value = incrementBy(value, 10); render(); });
```

### 9.3. Sprawdź lokalnie

```bash
node --test tests/counter.test.js   # teraz 5 testów, wszystkie zielone
```

---

## Krok 10. Commit, push i merge request

```bash
git add .
git commit -m "Dodanie przycisku +10 wraz z testem"
git push origin feature/przycisk-plus-10
```

GitLab wypisze w terminalu link do utworzenia merge requesta — kliknij go (albo: projekt → **Merge requests → New merge request**).

W formularzu MR:

1. **Title:** `Dodanie przycisku +10`
2. **Description:** krótko co i po co, np. `Nowa funkcja incrementBy + test + przycisk w UI.`
3. Zaznacz ✅ **Delete source branch when merge request is accepted**.
4. Kliknij **Create merge request**.

> ✅ **Checkpoint:** w widoku MR **automatycznie wystartował pipeline** (zakładka *Pipelines* w MR) — to zasługa reguły `merge_request_event` z naszego `workflow`. Uruchomią się joby `testy` i `build`; job `pages` **nie** — deploy jest tylko z `main`.

---

## Krok 11. Review, merge i automatyczny deploy

1. Podeślij link do MR-a **dwóm recenzentom dodanym w Kroku 8**. Recenzent przegląda zakładkę **Changes**, może dodać komentarz do linii, a następnie klika **Approve**.
2. Gdy: ✅ 2 × Approve oraz ✅ zielony pipeline → przycisk **Merge** staje się aktywny. Kliknij **Merge**.
3. Merge do `main` uruchamia **nowy pipeline** — tym razem z jobem `pages` (build → testy → deploy).
4. Poczekaj na zielony pipeline i odśwież stronę Pages.

> ✅ **Checkpoint końcowy:** na stronie `https://…gitlab.io/licznik-app/` widzisz przycisk **+10** i działa. 🎉

---

## Dla szybkich — zadania dodatkowe ⭐

- Dodaj job `lint` (etap `test`), który sprawdzi formatowanie: `npx prettier --check "src/**/*.js"`.
- Zepsuj celowo test na nowej gałęzi i zobacz, jak MR blokuje merge. Napraw i obserwuj auto-odblokowanie.
- Dodaj do `README.md` badge z statusem pipeline'u (Settings → CI/CD → General pipelines → *Pipeline status*).
- Zmień `decrement` tak, by pozwalał na wartości ujemne — **najpierw** zmień test (TDD), potem kod.
