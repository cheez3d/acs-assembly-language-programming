[elf]: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format

[rc4]: https://en.wikipedia.org/wiki/RC4
[file-extract-file-py]: flag-extract-file.py
[flags-extract-server-py]: flags-extract-server.py
[pyelftools]: https://github.com/eliben/pyelftools

[shellcode]: https://en.wikipedia.org/wiki/Shellcode
[ip]: https://en.wikipedia.org/wiki/Program_counter
[aslr]: https://en.wikipedia.org/wiki/Address_space_layout_randomization
[demo]: doc/demo.png

# Exploit [ELF][elf]s, not elves
Acest subdirector conține **Tema 3** din cadrul cursului _Introducere în organizarea calculatoarelor și limbaj de asamblare_ ce presupune exploatarea binarelor vulnerabile la atacuri de tip _buffer overflow_.

## Exercițiul 1
Adresa funcției vulnerabile este `0x0804867a`.

Aceasta este a 6-a funcție în cadrul apelurilor de funcții de citire din `main`. Funcțiea este vulnerabilă deoarece spre deosebire de toate celelalte funcții de citire, în cadrul acesteia se specifică pentru apelul la `read` o dimensiune mai mare a _buffer_-ului față de câți octeți au fost alocați pe stivă: se alocă 352 de octeți pe stivă, iar la apelul la `read` se specifică o dimensiune a _buffer_-ului de 476 de octeți => se poate exploata o vulnerabilitate de buffer overflow.

## Exercițiul 2
Având în vedere faptul că funcția găsită la exercițiul anterior apelează o altă funcție ce este dată ca argument din `main` (o mică digresiune: adresa pasată din `main` nu indică la o zonă executabilă cu cod, pentru că zona respectivă de memorie nu există => de aceea obținem `Segmentation fault` la rulare), putem folosi vulnerabilitatea găsită la exercițiul anterior pentru a suprascrie adresa funcției ce va fi apelată, pentru a apela funcția `print_flag`, ce are adresa `0x080485b1`.

_Payload_-ul este îndeajuns de mare încât să umple cele 5 _buffer_-e în care se citește până se ajunge la a 6-a funcție ce este vulnerabilă. Odată ajunși la a 6-a funcție de citire, este de ajuns ca textul citit să fie într-atât de mare încât să ajungă unde se află argumentul ce trebuie suprascris (adresa funcției apelate).

_Flag_-ul găsit: `NICE_FLAG{e89772a7ae2f17a00c01ca6c3adf9908}`

### Notă
Pentru acest exercițiu am analizat mai în detaliu și algoritmul de criptare a _flag_-ului descoperind că este vorba despre [RC4][rc4] și am scris două _script_-uri Python, [flag-extract-file.py][file-extract-file-py] care extrag direct _flag_-urile din orice binar specificat ca argument al _script_-ului în linia de comandă,  și [flags-extract-server.py][flags-extract-server-py] care extrage _flag_-uirle direct de pe server, fără a mai fi necesară trecerea prin pasul intermediar de descărcare a arhivei cu binare. _Script_-ul depinde de biblioteca [pyelftools][pyelftools] pentru a putea extrage conținutul secțiunii `.data` dintr-un binar [ELF][elf]. Această bibliotecă se poate instala folosind comanda `python -m pip install --user pyelftools`.

Exemplu de utilizare a [flag-extract-file.py][file-extract-file-py]:
```shell
$ ./flag-extract-file.py nice
NICE_FLAG{e89772a7ae2f17a00c01ca6c3adf9908}
$ ./flag-extract-file.py naughty
NAUGHTY_FLAG{b967a85d2552bf0ec728c07fdac66f24}
```

## Exercițiul 3

Adresa funcției vulnerabile este `0x08048682`.

Procedura este similară cu cea de la exercițiul anterior, doar că acum trebuie să și punem pe stivă anumite valori hexazecimale pe 4 octeți la anumite poziții pentru ca programul să nu apeleze `exit` până se ajunge la a 4-a funcție de citire care este vulnerabilă.

_Flag_-ul găsit: `NAUGHTY_FLAG{b967a85d2552bf0ec728c07fdac66f24}`

### Notă
Același algoritm de decriptare a _flag_-ului este folosit și la acest exercițiu, deci putem reutiliza _script_-ul de la exercițiul anterior pentru a extrage direct _flag_-ul din orice binar `naughty` dat ca
argument în linia de comandă pentru _script_.

## Exercițiul 4
Principiul este asemănător ca la exercițiile anterioare, doar că de această dată, în loc să apelăm o funcție deja existentă în executabil, vom crea propriul [shellcode][shellcode] care va deschide un _shell_. [Shellcode]-ul este următorul:
```x86asm
xor eax, eax
mov al, 0x0b      ; sys_execve
lea ebx, [esp+30] ; ebx va indica la șirul "/bin/sh" de mai jos
xor ecx, ecx      ; argv poate fi NULL pe Linux (vezi mai jos)
xor edx, edx      ; envp poate fi NULL pe Linux (vezi mai jos)
int 0x80          ; 
db "/bin/sh", 0   ; șirul de caractere este în continuarea shellcodeului
```

### Extras din `man 2 execve`
> On Linux, `argv` and `envp` can be specified as `NULL`. In both cases, this has the same effect as specifying the argument as a pointer to a list containing a single null pointer.

Folosind [acest site](https://defuse.ca/online-x86-assembler.htm) codul va fi asamblat ca mai jos:
```python
"\x31\xC0\xB0\x0B\x8D\x5C\x24\x1E\x31\xC9\x31\xD2\xCD\x80/bin/sh\x00"
```

Vom integra acest _shellcode_ în payload-ul deja existent de la exercițiul anterior. Acest _shellcode_ va fi pus pe stivă, iar cum executabilul este marcat ca având stiva executabilă vom putea executa instrucțiunile odată ce [IP][ip]-ul ajunge la ele. Am determinat adresa unde se va găsi _shellcode_-ul (cu [ASLR][aslr] dezactivat) folosind `gdb`. Adresa este `0xffffd61e`. Vom folosi această adresă în loc de adresa funcției `print_flag` deja existene în executabil (am observat că, chiar și cu [ASLR][aslr] dezactivat această adresă se schimbă la _reboot_, deci cel mai probabil _payload_-ul trebuie modificat manual de fiecare dată cu adresa potrivită; menționez că am nu am făcut tema pe mașina virtuală, ci direct pe laptop).

### Notă
Pentru ca _shell_-ul să nu se închidă imediat ce a fost deschis trebuie să livrăm _payload_-ul ca mai jos (pentru ca `stdin„ sa rămână deschis):
```shell
$ cat naughty_shellcode - | ./naughty
```

![demo]

Toate _payload_-urile au fost create folosind Python.
