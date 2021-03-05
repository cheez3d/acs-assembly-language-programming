[tema1-asm]: tema1.asm

[ascii]: https://en.wikipedia.org/wiki/ASCII

# Arbore sintactic abstract
Acest subdirector conține **Tema 1** din cadrul cursului _Introducere în organizarea calculatoarelor și limbaj de asamblare_ ce presupune evaluarea unui arbore sintactic abstract.

## Structura codului sursă
Întreaga temă este rezolvată în fișierul [tema1.asm][tema1-asm]. Unde este cazul, există comentarii menite să ușureze înțelegerea codului.

## Detalii de implementare ([tema1.asm][tema1-asm])
Implementarea este una recursivă și funcționează
în felul următor:
* primește un nod ca argument al funcției recursive
  * daca nodul este un număr (subarborele care are rădăcina în acest nod a fost evaluat complet), acesta este întors funcției apelante
  * daca nodul nu este încă un număr (subarborele care are rădăcina în acest nod nu a fost evaluat încă), atunci se evaluează valorile fiilor stâng și drept, iar apoi se aplică operația din nodul curent pe valorile celor doi fii și se întoarce rezultatul funcției aplenate

## Notă
Pentru aplicarea operației potrivite într-un mod cât mai elegant (fără o multitudine de instrucțiuni `cmp` și `jmp`) s-a folosit o tabelă de salturi ce exploatează faptul că în [ASCII][ascii] caracterele `+`, `-`, `*` și `/` sunt apropiate unul de celălalt (distanța dintre oricare două caractere este de maxim 1).
