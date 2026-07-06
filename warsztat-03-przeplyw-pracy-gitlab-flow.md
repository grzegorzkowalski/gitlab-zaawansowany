# Warsztat 3 · Przepływ pracy w GitLab — zastosowanie GitLab Flow

**Cel:** przechodzisz pełny cykl GitLab Flow: **issue → branch → commit → merge request → review → pipeline → merge → automatyczny deploy** — dodając do licznika przycisk **−10**.

**Czas:** ok. 25 min · **Wymagania:** ukończony Warsztat 2 (projekt w `firma-xx/produkty`, recenzenci z dostępem grupowym).

---

## Krok 1. Zacznij od issue

W GitLab Flow każda zmiana zaczyna się od zadania — nie od kodu.

1. Projekt → **Plan → Issues → New issue**.
2. **Title:** `Dodanie przycisku -10`
3. **Description:**

```markdown
Licznik ma przycisk +10, ale brakuje symetrycznego -10.

## Kryteria akceptacji
- [ ] funkcja `decrementBy` w `counter.js` (nie schodzi poniżej zera)
- [ ] test jednostkowy
- [ ] przycisk w interfejsie
```

4. Kliknij **Create issue** i **zapamiętaj numer** (np. `#1` — dalej pisz swój numer).

> ✅ **Checkpoint:** issue istnieje, kryteria akceptacji renderują się jako checklista.

---

## Krok 2. Utwórz gałąź prosto z issue

1. W widoku issue kliknij strzałkę przy **Create merge request** → wybierz **Create branch** → **Create branch**.
2. GitLab utworzył gałąź nazwaną numerem zadania (np. `1-dodanie-przycisku-10`). Pobierz ją lokalnie:

```bash
git fetch origin
git checkout 1-dodanie-przycisku-10   # użyj nazwy ze swojego projektu
```

> ✅ **Checkpoint:** `git branch` pokazuje, że jesteś na gałęzi z numerem issue.

---

## Krok 3. Wprowadź zmianę

**`src/counter.js`** — dopisz na końcu:

```js
export function decrementBy(value, step) {
  return Math.max(0, value - step);
}
```

**`tests/counter.test.js`** — rozszerz import w pierwszej linii o `decrementBy`:

```js
import { increment, decrement, reset, incrementBy, decrementBy } from "../src/counter.js";
```

…i dopisz test na końcu:

```js
test("decrementBy zmniejsza o krok, ale nie poniżej zera", () => {
  assert.equal(decrementBy(42, 10), 32);
  assert.equal(decrementBy(5, 10), 0);
});
```

**`src/index.html`** — obok przycisku `−1` dodaj:

```html
<button id="minus10">−10</button>
```

**`src/app.js`** — rozszerz import o `decrementBy`:

```js
import { increment, decrement, reset, incrementBy, decrementBy } from "./counter.js";
```

…i dodaj obsługę przycisku:

```js
document.getElementById("minus10").addEventListener("click", () => { value = decrementBy(value, 10); render(); });
```

Sprawdź lokalnie:

```bash
node --test tests/counter.test.js
```

> ✅ **Checkpoint:** 6 testów, wszystkie zielone.

---

## Krok 4. Commit z odwołaniem do issue

Numer zadania w komunikacie commita (`#1`) łączy zmianę z issue.

```bash
git add .
git commit -m "Dodanie przycisku -10 z testem (#1)"
git push origin 1-dodanie-przycisku-10
```

> ✅ **Checkpoint:** push przeszedł, GitLab wypisał w terminalu link do utworzenia merge requesta.

---

## Krok 5. Merge request z automatycznym zamknięciem issue

1. Kliknij link z terminala (albo: **Merge requests → New merge request**).
2. **Title:** `Dodanie przycisku -10`
3. **Description:**

```
Nowa funkcja decrementBy + test + przycisk w UI.

Closes #1
```

4. Zaznacz ✅ **Delete source branch when merge request is accepted** i utwórz MR.
5. Otwórz w MR zakładkę **Pipelines** — pipeline wystartował automatycznie; wykonują się joby `testy` i `build`, ale **nie** `pages` (deploy tylko z `main` — dokładnie tak zapisaliśmy to w regułach w Warsztacie 1).

> ✅ **Checkpoint:** MR istnieje, w opisie widać podlinkowane `Closes #1`, pipeline MR jest zielony.

---

## Krok 6. Review i merge

1. Podeślij link do MR **jednemu recenzentowi**. Recenzent: zakładka **Changes** → dodaje przynajmniej **jeden komentarz do linii kodu** → klika **Approve**.
2. Odpowiedz na komentarz i oznacz wątek jako rozwiązany (**Resolve thread**).
3. Gdy: ✅ approve i ✅ zielony pipeline — kliknij **Merge**.

> ✅ **Checkpoint:** MR zmergowany, gałąź źródłowa usunięta automatycznie.

---

## Krok 7. Obejrzyj, co zadziałało samo

1. **Plan → Issues** → Twoje issue jest **zamknięte** — zamknęło się samo dzięki `Closes #1`.
2. Otwórz zamknięte issue — na dole widać powiązany MR i commity: pełna historia zmiany w jednym miejscu.
3. Merge do `main` uruchomił nowy pipeline — tym razem z jobem `pages`. Po zielonym pipelinie odśwież stronę Pages.

> ✅ **Checkpoint końcowy:** issue zamknięte automatycznie, a na stronie Pages działa przycisk **−10**. Właśnie wykonałeś pełny cykl GitLab Flow. 🎉
