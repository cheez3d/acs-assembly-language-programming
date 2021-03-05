[tema2-asm]: tema2.asm

[cdecl]: https://en.wikipedia.org/wiki/X86_calling_conventions#cdecl

[lut]: https://en.wikipedia.org/wiki/Lookup_table
[ascii]: https://en.wikipedia.org/wiki/ASCII
[morse]: https://en.wikipedia.org/wiki/Morse_code
[lsb]: https://en.wikipedia.org/wiki/Bit_numbering#Least_significant_bit_in_digital_steganography

# Stegano
Acest subdirector conține **Tema 2** din cadrul cursului _Introducere în organizarea calculatoarelor și limbaj de asamblare_ ce presupune implementarea anumitor funcționalități steganografice și prelucrarea de imagini.

## Structura codului sursă
Întreaga temă este rezolvată în fișierul [tema2.asm][tema2-asm]. Unde este cazul, există comentarii menite să ușureze înțelegerea codului.

## Observații
* Am modificat scheletul pentru a optimiza codul, astfel că succesiunea de instrucțiuni `cmp` folosite pentru a sări la codul _task_-ului corespunzător a fost înlocuită cu un _indirect jump_ care se folosește de un _jump table_ unde sunt stocate adresele de unde incepe codul pentru fiecare _task_.
* Am corectat mesajul de _usage_ afișat în cazul în care sunt oferiți parametri necorespunzători programului.
* Am adăugat mai multe verificări pentru a determina dacă parametri specificați în linia de comandă sunt corecți (inclusiv verificare dacă fișierul imagine specificat există).
* Nu am respectat numele funcțiilor specificate în enunțul temei pentru fiecare _task_, deoarece pentru a evita _label_-uri duplicate, prefixez în cadrul unei funcții _label_-urile din interiorul acesteia cu numele funcției (e.g. în funcția `blur`, _label_-ul `copy_first_line` devine `blur_copy_first_line`). Cum unele nume erau destul de lungi (e.g. `bruteforce_singlebyte_xor`), ar fi rezultat niște _label_-uri mult prea lungi. Astfel, funcțiile corespunzătoare _task_-urilor sunt următoarele:
  | _Task_ | Denumire veche | Denumire nouă |
  | - | - | - |
  | 1 | `bruteforce_singlebyte_xor` | `xor_dec` |
  | 2 | `xor_enc` | `xor_enc` |
  | 3 | `morse_encrypt` | `morse_enc` |
  | 4 | `lsb_encode` | `lsb_enc` |
  | 5 | `lsb_decode` | `lsb_dec` |
  | 6 | `blur` | `blur` |
* Nu am respectat antetele funcțiilor specificate în enunt
  (și deci convenția [cdecl][cdecl] de apelare), eu recurgând la pasarea parametrilor direct în registre (mi s-a părut un pas inutil în plus folosirea convenției
  de apel [cdecl][cdecl] având în vedere că toate funcțiile din cadrul temei sunt scrise în limbaj de asamblare, și deci am control total asupra lor). Faptul că parametri sunt dați direct în registre, nemaifiind necesară trecerea prin stivă, sporește și performanța. În codul sursă există comentarii care specifică registrele folosite pentru pasarea parametrilor pentru fiecare funcție în parte.

## Detalii de implementare ([tema2.asm][tema2-asm])
Pentru a ajunge la codul aferent _task_-ului specificat ca argument folosim un _jump table_. De acolo inițializăm toți parametrii necesari pentru funcția/funcțiile ce vor fi apelate pentru rezolvarea _task_-ului respectiv. Mai jos sunt descriși pașii care se urmează în vederea rezolvării fiecărui _task_:
| _Task_ | Pași |
| - | - |
| 1 | 1. căutam cuvântul cheie pe fiecare linie, încercând fiecare cheie de un octet pentru asta<br/>2. în momentul găsirii cuvântului cheie, ne întoarcem la începutul liniei și decriptăm întegral mesajul, afișându-l alături de cheie și de numărul liniei la care acesta se află |
| 2 | 1. mai întâi rulăm funcția primului _task_ pentru a obține informațiile necesare în rezolvarea acestui task<br/>2. inserăm noul mesaj criptat cu cheia veche la o linie după mesajul decriptat deja existent și criptăm cu noua cheie, pixel cu pixel |
| 3 | 1. s-a folosit un [lookup table][lut] ce se bazează pe codificarea [ASCII][ascii] pentru a ajunge la reprezentarea în [Morse][morse] a unui caracter<br/>2. parcurgem mesajul de codificat caracter cu caracter și inserăm codificarea mesajului in [Morse][morse] în locul pixelilor de la poziția specificată |
| 4 |  1. parcurgem fiecare caracter al mesajului de inserat în imagine bit cu bit, inserând câte un bit în [LSB][lsb]-ul pixelului curent |
| 5 | 1. parcurgem câte 8 pixeli din imagine pentru extragerea unui caracter pe care îl afișăm (deci afișăm mesajul caracter cu caracter, pe măsură ce este decodificat) până când întâlnim terminatorul de șir |
| 6 | 1. facem media aritmetică a pixelului curent cu vecinii săi, având grijă să facem asta cu vechile valori pentru vecinul stâng și cel de deasupra care sunt modificați pe parcursul modificării imaginii (imaginea se parcurge de la stânga la dreapta, de sus în jos)

## Note
* În fișierul sursă se găsesc comentarii explicative pentru modul de implementare al fiecărui _task_.
* Am incercat să optimizez codul cât mai mult cu putință (e.g. să folosesc exclusiv registre cât de mult posibil) și de asemenea să nu scriu cod duplicat (e.g. vezi _label_-urile `task3`, `task4`, `task5` care folosesc toate același cod de _setup_ înainte de apelarea funcției corespunzătoare _task_-ului curent).
