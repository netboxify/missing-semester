# Uklanjanje grešaka i profilisanje

<a href="http://www.youtube.com/watch?feature=player_embedded&v=l812pUnKxME
" target="_blank"><img src="" 
alt="Lecture 7: Debugging and Profiling (2020)" width="240" height="180" border="10" /></a>

Zlatno pravilo u programiranju jeste da kod ne radi ono što očekujete od njega, već ono što mo kažete da uradi. Premostiti taj problem ponekad može biti veoma krupan korak. U ovoj lekciji ćemo preći korisne tehnike za nošenje sa kodom koji ima bagove i kojem fale resursi: debugging i profilisanje. 

## Uklanjanje grešaka

### Printf uklanjanje grešaka i logovanje

"Najefikasnije sredstvo za uklanjanje grešaka je i dalje pažljivo razmišljanje, zajedno sa dobro postavljenim izjavama o ispisu" - Brian Kernighan, __Unix for Beginners.__

Prvi pristup da bi se uklonile greške iz programa i dodavanje print izjave okolo dijela u kojem imate problem, i to ponavljate dok ne dobijete dovoljno informacija da shvatite ko je krivac za dati problem.

Drugi pristup je upotreba logovanja u vašem programu, umjesto ad hoc print izjava. Logovanje je bolje u odnosu na print izjave iz više razloga: 

- Možete logovati fajlove, sockets-e, ili čak i udaljene servere umjesto standardnog output-a.
- Logovanje podržava ozbiljne nivoe (kao što su  INFO, DEBUG, WARN, ERROR, i sl.), koji vam omogućuju da prema njima filtrirate output.
- Za nove probleme, postoji dobra šansa da će vaši logovi sadržati dovoljno informacija da shvatite šta nije u redu.

[Ovdje](https://missing.csail.mit.edu/static/files/logger.py) je primjer koda koji loguje poruku: 

```python
$ python logger.py
# Raw output as with just prints
$ python logger.py log
# Log formatted output
$ python logger.py log ERROR
# Print only ERROR levels and above
$ python logger.py color
# Color formatted output
```

Jedan od mojih omiljenih savjeta da bi poboljšali čitljivost logova jeste da obilježite kod bojom. Do sada ste vjerovatno shvatili da vaš terminal koristi boje da učini stvari čitljivijim. Ali, kako to radi? Programi kao što su `ls` ili `grep` koriste [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code), koji su specijalizovani dijelovi karaktera koji pokazuju vašem shell-u da promijeni boju output-a. Na primer, izvršavanje `echo -e "\e[38;2;255;0;0mThis is red\e[0m"` ispisuje poruku `This is red` crvenom bojom na vašem terminalu, dokle god podržava [true color](https://gist.github.com/XVilka/8346728#terminals--true-color). Ako vaš terminal ne podržava ovo (npr. macOS Terminal.app), možete koristiti univerzalnije podržani escape code sa izborom od 16 boja, na primer `echo -e "\e[31;1mThis is red\e[0m"`. 

Sledeća skripta pokazuje kako ispisati mnogo RGB boja u vašem terminalu (opet, sve dok podržavaju true color).

```shell
#!/usr/bin/env bash
for R in $(seq 0 20 255); do
    for G in $(seq 0 20 255); do
        for B in $(seq 0 20 255); do
            printf "\e[38;2;${R};${G};${B}m█\e[0m";
        done
    done
done
```

### Logovi trećih strana

Kako počinjete da pravite šire softverske sisteme vjerovatno ćete naići na zavisnosti koje se pokreću kao poseban program. Veb serveri, baze podataka ili brokeri poruka su česti primjeri ovakvih vrsta zavisnosti. Kada imate interakciju sa ovim sistemima često je neophodno pročitati njihove logove, jer error poruka na klijent strani možda neće biti dovoljna.

Srećom, većina programa piše svoje logove negdje u vašem sistemu. U UNIX sistemima, uobičajeno je da programi pišu svoje logove u `var/log`. Na primer, [NGINX](https://www.nginx.com/) webserveri ih čuvaju u `/var/log/nginx`. U skorije vrijeme, sistemi su počeli da koriste **system log**, što je sve više mjesto gdje vaše log poruke idu. Većina (ali ne i svi) Linux sistemi koriste `systemd`, sistem daemon koji kontroliše mnoge stvari u vašem sistemu kao npr. koje usluge su omogućene i pokrenute. `systemd` stavlja logove u `/var/log/journal` u posebnom formatu i možete koristiti [journalctl](https://www.man7.org/linux/man-pages/man1/journalctl.1.html) komandu da prikažete poruke. Slično, na macOS takođe postoji `/var/log/system.log` ali povećan broj alata koristi system log, koji može biti prikazan sa [log show](https://www.manpagez.com/man/1/log/). Na većini UNIX sistema takođe možete koristiti [dmesg](https://www.man7.org/linux/man-pages/man1/dmesg.1.html) komandu da pristupite logu kernela.

Za logovanje u sistem logs možete koristiti [logger](https://www.man7.org/linux/man-pages/man1/logger.1.html) shell program. Evo primjera korišćenja `logger`-a i kako da provjerite da li je unos stigao do sistem logova. Štaviše, većina programskih jezika ima vezivanje logginga za sistem log.

```shell
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

Kao što smo vidjeli u lekciji o upravljanju podacima, logs mogu biti veoma opširni i oni zahtijevaju odreženi nivo filtriranja i procesiranja da bi dobili informacije koje želite. Ukoliko upadnete u teško filtriranje kroz `journalctl` i `log show` razmislite da koristite njihove flagove, koji mogu izršiti prvi korak u filtriranju njihovog outputa. Postoje još neki alati kao što je [lnav](http://lnav.org/), koji omogućuju poboljšanu prezentaciju i navigaciju kroz log fajlove.

### Debageri

Kada printf ispravljanje grešaka nije dovoljno trebali bi da koristite debager. Debageri su programi koji vam omogućuju interakciju sa izvršenjem programa, dozvoljavajući sledeće: 

- Zaustavljanje izvršenja programa kada dođe do određene linije.
- Prolazak kroz program sa jednom instrukcijom u isto vrijeme.
- Ispitivanje vrijednosti varijable nakon greške programa.
- Uslovno zaustavljanje izvršenja kada je određeni uslov zadovoljen.
- I još mnogo naprednih funkcija.

Mnogi programski jezici dolaze sa nekom formom debagera. U Pythonu to je Python Debugger [pdb](https://docs.python.org/3/library/pdb.html).

Evo kratkog opisa nekih od komandi koje `pdb` podržava: 

- **l**(ist) - Prikazuje 11 linija oko trenutne linije ili nastavlja prethodni listing.
- **s**(tep) - Izvršava trenutnu liniju, zaustavlja se prve moguće prilike.
- **n**(ext) - Nastavlja izvršavanje sve dok se ne dostigne sledeća linija u trenutnoj funkciji ili dok ne vrati nešto.
- **b**(reak) - Postavlja tačku prekida (zavisno od pruženih argumenata).
- **p**(rint) - Procijenjuje izraz u trenutnom kontekstu i ispisuje njegovu vrijednost.
Takođe postoji `pp` da se prikaže koristeći [pprint](https://docs.python.org/3/library/pprint.html).
- **r**(eturn) - Nastavlja izvršavanje sve dok sledeća funkcija ne vrati.
- **q**(uit) - Izlazi iz debagera.

Hajde da prođemo kroz primjer koristeći `pdb` da popravimo sledeće python kod koji ima bagove. (Pogledajte video lekciju).

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(n):
            if arr[j] > arr[j+1]:
                arr[j] = arr[j+1]
                arr[j+1] = arr[j]
    return arr

print(bubble_sort([4, 2, 1, 8, 7, 6]))
```

Imajte u vidu da pošto je Python interpretiran jezik mi možemo koristiti `pdb` shell da izvršimo komande i izvršimo instrukcije. [ipdb](https://pypi.org/project/ipdb/) je unapređeni `pdb` koji koristi [IPython](https://ipython.org/) REPL omogućujući dovršavanje tabova, isticanje sintakse, bolje tragove, i bolju introspekciju dok zadržava isti isti interfejs kao i `pdb` modul.

Za još nižeg nivoa programiranja vjerovatni ćete željeti da pogledate [gdb](https://www.gnu.org/software/gdb/) (i kvalitet njegovih izmjena [pwndbg](https://github.com/pwndbg/pwndbg)) i [lldb](https://lldb.llvm.org). Oni su optimizovani za debaging za jezike kao što je C, ali će vam dozvoliti isptivanje bilo kojeg procesa i dobiti trenutno stanje mašine: registre, stack, brojač programa i sl.

### Specijalizovani alati

Čak iako želite da ispravite greške u crnom binarnom okviru, postoje alati koji vam mogu pomoći sa tim. Kada god program mora da izvrši akciju koju samo kernel može, oni koriste [System Calls](https://en.wikipedia.org/wiki/System_call). Postoje komande koje vam dopuštaju da pratite syscalls koje vaš program napravi. U Linuxu postoji [strace](https://www.man7.org/linux/man-pages/man1/strace.1.html), a macOS i BSD imaju [dtrace](http://dtrace.org/blogs/about/). `dtrace` može biti nezgodan za korišćenje zato što koristi svoj `d` jezik, ali postoji omotač koji se naziva `dtruss` koji omogućuje interfejs koji je sličniji `strace` (više detalja [ovdje](https://8thlight.com/blog/colin-jones/2015/11/06/dtrace-even-better-than-strace-for-osx.html).

Ispod su neki primjeri korišćenja `strace` ili `dtruss` da pokažu [stat](https://www.man7.org/linux/man-pages/man2/stat.2.html) syscall tragove za izvršenje `ls`. Za dublji pregled `strace`, [ovo](https://blogs.oracle.com/linux/strace-the-sysadmins-microscope-v2) je dobro za čitanje.

```shell
# On Linux
sudo strace -e lstat ls -l > /dev/null
4
# On macOS
sudo dtruss -t lstat64_extended ls -l > /dev/null
```

Pod nekim okolnostima, možda ćete morati da pogledate network pakete da bi shvatili koji je problem u vašem programu. Alati kao što su [tcpdump](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) i [Wireshark](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) su analizatori network paketa koji vam dopuštaju da čitate sadržaj network paketa i filtrirate ih na osnovu različitog kriterijuma. 

Za web development, Chrome/Firefox alatke za programere su veoma pogodne. 
One imaju veliki broj alata, uključujući: 

- Izvorni kod - Pregledajte HTML/CSS/JS izvorni kod bilo kog sajta.
- Uživo modifikacije HTML/CSS/JS - Mijenjate sadržaj sajtova, stilova i ponašanja da bi testirali (možete i sami da vidite da snimci sajtova nisu validni dokazi).
- Javascript shell - Izvršite komande u JS REPL.
- Network - Analizirajte vremensku liniju zahtjeva.
- Skladište - Pregledajte kolačiće i skladište lokalne aplikacije.

### Statička analiza

Za neke probleme ne morate da izvršavate bilo kakav kod. Na primer, samo pažljivim gledanjem u dio vašeg koda možete shvatiti varijabla petlje zasjenjuje već postojeću varijablu ili naziv funkcije; ili da program čita varijablu prije njenog definisanja. Ovo je gdje alati [statičke analize](https://en.wikipedia.org/wiki/Static_program_analysis) dolaze do izražaja. Programi statičke analize uzimaju izvorni kod kao input i analiziraju ga koristeći pravila kodiranja da bi procijenili njegovu ispravnost.

U sledećem Python isječku postoji nekoliko grešaka. Prvo, naša varijabla petlje `foo` zasjenjuje prethodnu definiciju funkcije `foo`. Takođe smo napisali `baz` umjesto `bar` u poslednjoj liniji, tako da će u programu doći do greške nakon izvršavanja `sleep` poziva (što će trajati oko minut).

```python
import time

def foo():
    return 42

for foo in range(5):
    print(foo)
bar = 1
bar *= 0.2
time.sleep(60)
print(baz)
```

Statička analiza može identifikovati ovakvu vrstu problema. Kada pokrenemo [pyflakes](https://pypi.org/project/pyflakes/) u kodu mi dobijamo grešku koja se odnosi na oba baga. [mypy](http://mypy-lang.org/) je još jedan alat koji može otkriti problem sa provjerom tipa. Ovdje, `mypy` nas upozorava da je bar inicijalno `int` a da se kasnije pretvara u `float`. Opet, imajte u vidu da su svi ovi problemi otkriveni bez potrebe da se izvršava kod. 

U lekciji o shell alatima pokrili smo [shellcheck](https://www.shellcheck.net/), koji je sličan alat za shell skripte. 

```python
$ pyflakes foobar.py
foobar.py:6: redefinition of unused 'foo' from line 3
foobar.py:11: undefined name 'baz'

$ mypy foobar.py
foobar.py:6: error: Incompatible types in assignment (expression has type "int", variable has type "Callable[[], Any]")
foobar.py:9: error: Incompatible types in assignment (expression has type "float", variable has type "int")
foobar.py:11: error: Name 'baz' is not defined
Found 3 errors in 1 file (checked 1 source file)
```

Većina editora i radnih okruženja podržavaju prikazivanje ovog outputa u samom editoru, ističući lokacije upozorenja i grešaka. Ovo se često naziva **code linting** i takođe može biti korišćeno da se prikažu različiti tipovi problema kao što su stilska kršenja ili nesigurni konstrukti.

U vim-u, plugini [ale](https://vimawesome.com/plugin/ale) ili [syntastic](https://vimawesome.com/plugin/syntastic) će odraditi taj posao. Za Python, [pylint](https://github.com/PyCQA/pylint) i [pep8](https://pypi.org/project/pep8/) su primjeri stilskih lintera i [bandit](https://pypi.org/project/bandit/) je alat koji je dizajniran da pronađe poznate bezbjednosne probleme. Za druge jezike ljudi su kompajlirali sveobuhvatne liste korisnih alata za statičku analizu, kao što je [Awesome Static Analysis](https://github.com/analysis-tools-dev/static-analysis) (možda ćete željeti da pogledate __Writing__ sekciju) a za lintere tu je [Awesome Linters](https://github.com/caramelomartins/awesome-linters).

Dodatni alati za linting stila su kod formateri kao što je [black](https://github.com/psf/black) za Python, `gofmt` za Go, `rustfmt` za Rust ili [prettier](https://prettier.io/) za JavaScript, HTML i CSS. Ovi alati autoformatuju vaš kod tako da bude dosledan sa uobičajenim stilskim obrascima za dati programski jezik. Iako možda niste voljni da prepustite stilsku kontrolu nad vašim kodom, standardizacija formata koda će pomoći drugim ljudima u čitanju vašeg koda i pomoći će vama u čitanju koda (stilski standardizovanog) drugih ljudi.

## Profilisanje

