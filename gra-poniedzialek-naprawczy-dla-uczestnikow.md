# 🧹 „Poniedziałek naprawczy" — misja: uratuj repozytorium

**Czas:** 20-30 min · **Forma:** solo · **Punkty:** 100 (+bonus)

Każda podpowiedź ▸ kosztuje **5 pkt** — otwierajcie tylko w ostateczności.

---

## 📜 Fabuła

> Darek Stażysta poszedł na dwutygodniowy urlop. Zostawił po sobie repozytorium cennika wycieczek biura **„Wakacje z Gitem"** — a w nim: commity o treści „asdfgh", swoje prywatne notatki, śmieci generowane przez skrypty, niedokończone eksperymenty na gałęziach i tag o nazwie `final-final-2`.
>
> W piątek wchodzi promocja letnia. **Macie jeden poniedziałek, żeby doprowadzić to repozytorium do porządku i przygotować wydanie v2.0.0.**

Sklonujcie repozytorium (adres poda prowadzący):

```bash
git clone https://github.com/grzegorzkowalski/poniedzialek-naprawczy.git
cd poniedzialek-naprawczy
git fetch --all --tags
```

Rozejrzyjcie się: `git log --all --graph --oneline --decorate` — to Wasza mapa terenu. Postępy notujcie w **protokole sprzątania** (szablon na końcu).

---

## Misja 1 · Audyt jakości commitów *(10 pkt)*

Przejrzyjcie historię gałęzi `main`. Znajdźcie **trzy najgorsze commity** i wpiszcie do protokołu: co jest z nimi nie tak i **czego nie da się przez nie zrobić** (np. znaleźć zmiany po opisie, zrozumieć historię, bezpiecznie cofnąć).

Ustalcie i zapiszcie **3 własne zasady dobrego commita** — będą Was obowiązywać do końca gry (a prowadzący będzie je egzekwował!). Dla inspiracji:

- opis mówi **co i po co**, nie „zmiany";
- jeden commit = jedna logiczna zmiana;
- po przeczytaniu samego `git log --oneline` da się opowiedzieć historię projektu.

<details><summary>▸ podpowiedź (−5 pkt)</summary>

`git log --oneline` pokaże listę; `git show <hash>` — co commit naprawdę zmienia. Porównajcie opisy commitów Ani i Darka.
</details>

---

## Misja 2 · Gaśnica: `git restore` *(15 pkt)*

Zanim zaczniecie zmieniać na poważnie — przećwiczcie ratowanie się z wpadek. **Celowo** zróbcie dwie katastrofy i wyjdźcie z nich bez śladu:

1. **Katastrofa A:** nadpiszcie cały `data.js` byle czym (`echo "ups" > data.js`). Plik zniszczony, zmiana **niedodana** do stagingu. Przywróćcie go jedną komendą — bez klonowania od nowa.
2. **Katastrofa B:** zmieńcie dowolną cenę w `data.js` i **dodajcie do stagingu** (`git add`). Teraz: najpierw wycofajcie plik **ze stagingu** (zmiana ma zostać w pliku!), obejrzyjcie `git status`, a potem wycofajcie też zmianę z pliku.

Do protokołu: **czym różni się `git restore <plik>` od `git restore --staged <plik>`?** (jedno zdanie własnymi słowami).

<details><summary>▸ podpowiedź (−5 pkt)</summary>

`git restore data.js` cofa plik do wersji z ostatniego commita; `git restore --staged data.js` tylko wyjmuje ze stagingu. `git status` podpowiada obie komendy wprost — czytajcie jego komunikaty!
</details>

---

## Misja 3 · Maszyna czasu: `git reset` + porządny commit *(20 pkt)*

Szef prosi o zmianę ceny rejsu. Zróbcie to najpierw **tak jak Darek**, a potem naprawcie historię:

1. Zmieńcie w `data.js` cenę **Rejsu po Odrze** na `149.99` → commit z opisem `test`.
2. Dopiszcie w `data.js` na końcu linię `// tmp` → drugi commit z opisem `asdf`.
3. Spójrzcie na `git log --oneline` — właśnie dołożyliście dwa „darkowe" commity. **Cofnijcie oba naraz**, tak żeby zmiany **zostały** w plikach/stagingu (nic nie może zginąć!).
4. Usuńcie śmieciową linię `// tmp`, a zmianę ceny zacommitujcie **jednym porządnym commitem** zgodnym z Waszymi zasadami z Misji 1 (np. dlaczego cena rośnie?).

Do protokołu: hash i opis finalnego commita + odpowiedź: **co by się stało, gdybyście zamiast `--soft` użyli `--hard`?** (Nie sprawdzajcie na tym repo! Możecie sprawdzić na boku.)

<details><summary>▸ podpowiedź (−5 pkt)</summary>

`git reset --soft HEAD~2` — cofa wskaźnik gałęzi o 2 commity, zostawiając zmiany w stagingu. `--hard` skasowałby też zmiany w plikach — bezpowrotnie (prawie…).
</details>

---

## Misja 4 · Kwarantanna: `.gitignore` *(15 pkt)*

W repo jest skrypt `generuj-raport.sh` — uruchomcie go **dwa razy** i sprawdźcie `git status`. Śmieci? Śmieci.

1. Sprawcie, żeby `raport.log` oraz katalog `tmp/` **nigdy nie trafiały** do repozytorium. `git status` ma być czysty mimo ich obecności na dysku.
2. Sprawa delikatniejsza: Darek zacommitował plik `notatki.txt` (zajrzyjcie do środka… właśnie dlatego prywatne pliki nie wchodzą do repo). Dopisanie go do `.gitignore` **nie wystarczy** — plik jest już śledzony. Sprawcie, żeby: zniknął z repozytorium (i z `git status`), ale **został na dysku** Darka.
3. Zacommitujcie `.gitignore` + usunięcie notatek — porządnym opisem.

Do protokołu: dlaczego `.gitignore` nie działa na pliki już śledzone i jaka komenda to rozwiązuje?

<details><summary>▸ podpowiedź (−5 pkt)</summary>

`.gitignore` filtruje tylko pliki **nieśledzone**. Śledzony plik wyjmiecie z indeksu przez `git rm --cached notatki.txt` (bez `--cached` skasowałby też plik z dysku!).
</details>

---

## Misja 5 · Scalenie promocji: `merge` z konfliktem *(20 pkt)*

Ania przygotowała gałąź `promocja-lato` z obniżką ceny rejsu. Ale Wy w Misji 3 też zmieniliście tę cenę…

1. Obejrzyjcie **przed scaleniem**, co gałąź zmienia względem main: `git diff main promocja-lato` (albo `git log main..promocja-lato`).
2. Scalcie `promocja-lato` do `main`. Będzie **konflikt** — i o to chodzi.
3. Otwórzcie plik konfliktowy i przeczytajcie znaczniki `<<<<<<<` / `=======` / `>>>>>>>`: która wersja jest „Wasza", a która „ich"?
4. Decyzja biznesowa: **wygrywa cena promocyjna 99.99** (promocja to promocja). Rozwiążcie konflikt, dokończcie merge, sprawdźcie `index.html` w przeglądarce — suma ma być liczbą, a rejs po 99.99.

Do protokołu: suma z przeglądarki (to Wasz **kod kontrolny**) + jedno zdanie: skąd Git *wiedział*, że jest konflikt, a inne zmiany scalił sam?

<details><summary>▸ podpowiedź (−5 pkt)</summary>

`git merge promocja-lato`; po ręcznej edycji pliku: `git add data.js` i `git commit` (albo `git merge --continue`). Konflikt = obie strony zmieniły **te same linie**.
</details>

---

## Misja 6 · Chirurgia: `cherry-pick` *(15 pkt)*

Na gałęzi `eksperymenty-darka` są trzy commity. Jeden z nich jest **wartościowy** (wymagany prawnie!), dwa to śmieci. **Nie wolno scalać całej gałęzi.**

1. Obejrzyjcie commity gałęzi (`git log eksperymenty-darka --oneline -3`, szczegóły przez `git show`).
2. Wybierzcie ten właściwy i przenieście **tylko jego** na `main` jedną komendą.
3. Dowód: otwórzcie `index.html` w przeglądarce — na dole strony ma być stopka, a tło ma **pozostać białe** (jeśli jest różowe — przenieśliście za dużo; wiecie już, jak się cofnąć 😉).

Do protokołu: hash wybranego commita + jedno zdanie: czym różni się `cherry-pick` od `merge`?

<details><summary>▸ podpowiedź (−5 pkt)</summary>

`git cherry-pick <hash>` — kopiuje pojedynczy commit na bieżącą gałąź (powstaje **nowy** commit z tą samą zmianą).
</details>

---

## Misja 7 · Wydanie: porządny tag *(5 pkt)*

W repo wisi tag `final-final-2` z opisem „chyba dziala" — obejrzyjcie go (`git show final-final-2`) jako pomnik antywzorca.

Otagujcie aktualny stan `main` jako **`v2.0.0`** — tagiem **adnotowanym**, z opisem, który za pół roku powie, *co* weszło w to wydanie (macie materiał: promocja, stopka, porządki).

Do protokołu: pełny opis Waszego taga.

<details><summary>▸ podpowiedź (−5 pkt)</summary>

`git tag -a v2.0.0 -m "..."` — flaga `-a` robi tag adnotowany (z autorem, datą i opisem); bez niej powstaje tag lekki, „goła etykieta".
</details>

---

## ⭐ Misje bonusowe (dla szybkich)

- **+5 pkt:** ostatni commit ma literówkę w opisie? A może chcecie dopisać zapomniany plik? Znajdźcie parametr `git commit`, który poprawia **ostatni** commit zamiast tworzyć nowy — i użyjcie go świadomie. Kiedy NIE wolno tego robić?
- **+5 pkt:** posprzątajcie gałęzie: usuńcie lokalnie scaloną `promocja-lato`. Spróbujcie tak samo usunąć `eksperymenty-darka` — Git zaprotestuje. Dlaczego, i jaka flaga to wymusza?
- **+5 pkt:** wypchnijcie efekt na swój fork/projekt razem z tagiem `v2.0.0` (`git push --tags`) i sprawdźcie, jak GitLab pokazuje tag w sekcji *Code → Tags*.

---

## 📋 Protokół sprzątania (skopiujcie i wypełniajcie)

```markdown
# Protokół zespołu: ______                     Punkty: __ /100 (+bonus)

M1  Trzy najgorsze commity (hash + zarzut): ________________________________
    Nasze 3 zasady dobrego commita: ________________________________________
M2  restore vs restore --staged: ___________________________________________
M3  Finalny commit (hash + opis): __________________________________________
    Co zrobiłby --hard: ____________________________________________________
M4  Czemu .gitignore nie łapie śledzonych i co pomaga: _____________________
M5  KOD KONTROLNY (suma z przeglądarki): ______ zł
    Skąd Git wie, że jest konflikt: ________________________________________
M6  Przeniesiony commit (hash): _______  cherry-pick vs merge: _____________
M7  Opis naszego taga v2.0.0: ______________________________________________
```

**Meta:** `git log --oneline` Waszego `main` czyta się jak historia projektu? To wygraliście — niezależnie od punktów.
