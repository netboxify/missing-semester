Zlatno pravilo u programiranju jeste da kod ne radi ono što očekujete od njega, već ono što mu kažete da uradi. Premostiti taj problem ponekad može biti veoma krupan korak. U ovoj lekciji ćemo preći korisne tehnike za nošenje sa kodom koji ima bagove i kojem fale resursi: uklanjanje grešaka i profilisanje. 

## Uklanjanje grešaka

### Printf uklanjanje grešaka i logovanje

"Najefikasnije sredstvo za uklanjanje grešaka je i dalje pažljivo razmišljanje, zajedno sa dobro postavljenim izjavama o ispisu" - Brian Kernighan, __Unix for Beginners.__

Prvi pristup da bi se uklonile greške iz programa je dodavanje print izjave okolo dijela u kojem imate problem, i ponavljanje toga dok ne dobijete dovoljno informacija da shvatite ko je krivac za dati problem.

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

Srećom, većina programa piše svoje logove negdje u vašem sistemu. U UNIX sistemima, uobičajeno je da programi pišu svoje logove u `var/log`. Na primer, [NGINX](https://www.nginx.com/) webserveri ih čuvaju u `/var/log/nginx`. U skorije vrijeme, sistemi su počeli da koriste **system log**, što je sve više mjesto gdje vaše log poruke idu. Većina (ali ne i svi) Linux sistemi koriste `systemd`, sistem daemon koji kontroliše mnoge stvari u vašem sistemu kao npr. koje usluge su omogućene i pokrenute. `systemd` stavlja logove u `/var/log/journal` u posebni format i možete koristiti [journalctl](https://www.man7.org/linux/man-pages/man1/journalctl.1.html) komandu da prikažete poruke. Slično, na macOS takođe postoji `/var/log/system.log` ali povećan broj alata koristi system log, koji može biti prikazan sa [log show](https://www.manpagez.com/man/1/log/). Na većini UNIX sistema takođe možete koristiti [dmesg](https://www.man7.org/linux/man-pages/man1/dmesg.1.html) komandu da pristupite logu kernela.

Za logovanje u sistem logs možete koristiti [logger](https://www.man7.org/linux/man-pages/man1/logger.1.html) shell program. Evo primjera korišćenja `logger`-a i kako da provjerite da li je unos stigao do sistem logova. Štaviše, većina programskih jezika ima vezivanje logginga za sistem log.

```shell
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

Kao što smo vidjeli u lekciji o upravljanju podacima, logs mogu biti veoma opširni i oni zahtijevaju određeni nivo filtriranja i procesiranja da bi dobili informacije koje želite. Ukoliko upadnete u teško filtriranje kroz `journalctl` i `log show` razmislite da koristite njihove flagove, koji mogu izršiti prvi korak u filtriranju njihovog outputa. Postoje još neki alati kao što je [lnav](http://lnav.org/), koji omogućuju poboljšanu prezentaciju i navigaciju kroz log fajlove.

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

Hajde da prođemo kroz primjer koristeći `pdb` da popravimo sledeći Python kod koji ima bagove. (Pogledajte video lekciju).

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

Imajte u vidu da pošto je Python interpretiran jezik mi možemo koristiti `pdb` shell da izvršimo komande i izvršimo instrukcije. [ipdb](https://pypi.org/project/ipdb/) je unapređeni `pdb` koji koristi [IPython](https://ipython.org/) REPL omogućujući dovršavanje tabova, isticanje sintakse, bolje tragove, i bolju introspekciju dok zadržava isti interfejs kao i `pdb` modul.

Za još niži nivo programiranja vjerovatno ćete željeti da pogledate [gdb](https://www.gnu.org/software/gdb/) (i kvalitet njegovih izmjena [pwndbg](https://github.com/pwndbg/pwndbg)) i [lldb](https://lldb.llvm.org). Oni su optimizovani za debaging za jezike kao što je C, ali će vam dozvoliti isptivanje bilo kojeg procesa i dobićete trenutno stanje mašine: registre, stack, brojač programa i sl.

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

- Izvorni kod - Pregledajte HTML/CSS/JS izvorni kod bilo kojeg sajta.
- Uživo modifikacije HTML/CSS/JS - Mijenjate sadržaj sajtova, stilova i ponašanja da bi testirali (možete i sami da vidite da snimci sajtova nisu validni dokazi).
- Javascript shell - Izvršite komande u JS REPL.
- Network - Analizirajte vremensku liniju zahtjeva.
- Skladište - Pregledajte kolačiće i skladište lokalne aplikacije.

### Statička analiza

Za neke probleme ne morate da izvršavate bilo kakav kod. Na primer, samo pažljivim gledanjem u dio vašeg koda možete shvatiti da varijabla petlje zasjenjuje već postojeću varijablu ili naziv funkcije; ili da program čita varijablu prije njenog definisanja. Ovo je gdje alati [statičke analize](https://en.wikipedia.org/wiki/Static_program_analysis) dolaze do izražaja. Programi statičke analize uzimaju izvorni kod kao input i analiziraju ga koristeći pravila kodiranja da bi procijenili njegovu ispravnost.

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

Čak iako se vaš kod funkcionalno ponaša kao što i očekujete, to možda neće biti dovoljno dobro ukoliko zauzima čitav CPU ili memoriju u tom procesu. Na časovima algoritama se često uči big __O__ notation ali ne i kako pronaći hot spots u vašim programima. Kako je [prerana optimizacija korijen svakog zla](http://wiki.c2.com/?PrematureOptimization), trebali bi da naučite o profilisanju i alatima za praćenje. Oni će vam pomoći da shvatite koji dijelovi vašeg programa zauzimaju najviše vremena i/ili resursa tako da se možete fokusirati na optimizaciju tih dijelova.

### Tajming

Slično kao u slučaju uklanjanja grešaka, u mnogim scenarijima može biti sasvim dovoljno da ispišete vrijeme koje je vašem kodu bilo potrebno između dvije tačke. Evo primjera u Pythonu koristeći [time](https://docs.python.org/3/library/time.html) modul.

```python
import time, random
n = random.randint(1, 10) * 100

# Get current time
start = time.time()

# Do some work
print("Sleeping for {} ms".format(n))
time.sleep(n/1000)

# Compute time between start and now
print(time.time() - start)

# Output
# Sleeping for 500 ms
# 0.5713930130004883
```

Ipak, vrijeme sata može biti pogrešno uzimajući u obzir da vaš računar možda obavlja i druge procese u isto vrijeme ili čeka na to da se određeni događaj dogodi. Uobičajeno je za alate da prave razliku između __Real__, __User__ i __Sys__ vremena. Generalno, __User__ + __Sys__ vam govore koliko je vremena vaš proces zapravo proveo u CPU (detaljnije objašnjenje je [ovdje](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)).

- __Real__ - Sat pokazuje proteklo vrijeme od početka do kraja programa, uključujući vrijeme koje su zauzeli drugi procesi i vrijeme dok je bio blokiran(npr. čekajući na I/O ili network)
- __User__ - Količinu vremena provedenog u CPU izvršavajući kod korisnika
- __Sys__ Količinu vremena provedenog u CPU izvršavajući kernel kod

Na primer, pokušajte da izvršite komandu koja obavlja HTTP zahtjev i dodajte prefix [time](https://www.man7.org/linux/man-pages/man1/time.1.html). Ukoliko vam je konekcija spora možda ćete dobiti output kao što je navedeno ispod. Ovdje je trebalo preko 2 sekunde da se zahtjev obradi ali je proces trajao 15ms CPU korisničkog vremena i 12ms kernel CPU vremena.

```shell
$ time curl https://missing.csail.mit.edu &> /dev/null`
real    0m2.561s
user    0m0.015s
sys     0m0.012s
```

### Profajleri

### CPU

Većinu vremena kada ljudi ukazuju na __profajlere__ oni zapravo misle na __CPU profajlere__, koji su i najčešći. Postoji dva glavna tipa CPU profajlera: __tracing__ i __sampling__ profajleri. Tracing profajleri vode evidenciju svakog poziva funkcije koji vaš program napravi dok sampling profajleri periodično probaju vaš program (često svake milisekunde) i prave zabilješku stack programa. Oni koriste ove zabilješke da predstave zbirne statistike u vezi sa tim na šta je vaš program proveo najviše vremena. [Ovdje](https://jvns.ca/blog/2017/12/17/how-do-ruby---python-profilers-work-/) je dobar uvodni članak ukoliko želite više detalja na ovu temu.

Većina programskih jezika ima neku vrstu profajlera komandne linije koje možete koristiti da bi analizirali vaš kod. Oni su često integrisani sa punopravnim radnim okruženjem ali za ovu lekciju mi ćemo se fokusirati na same alate komandne linije.

U Pythonu možemo koristiti `cProfile` modul da profilišemo vrijeme za poziv funkcije.
Evo jednostavnog primjera koji implementira osnovni grep u Pythonu:

```python
#!/usr/bin/env python

import sys, re

def grep(pattern, file):
    with open(file, 'r') as f:
        print(file)
        for i, line in enumerate(f.readlines()):
            pattern = re.compile(pattern)
            match = pattern.search(line)
            if match is not None:
                print("{}: {}".format(i, line), end="")

if __name__ == '__main__':
    times = int(sys.argv[1])
    pattern = sys.argv[2]
    for i in range(times):
        for file in sys.argv[3:]:
            grep(pattern, file)
```

Možemo profilsati ovaj kod koristeći sledeću komandu. Analiziranjem outputa možemo vidjeti da IO zauzima najviše vremena i da kompajliranje tog regexa takođe troši srednju količinu vremena. Budući da se regex kompajlira samo jednom, možemo ga faktorisati izvan for-a.

```python
$ python -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py

[omitted program output]

 ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     8000    0.266    0.000    0.292    0.000 {built-in method io.open}
     8000    0.153    0.000    0.894    0.000 grep.py:5(grep)
    17000    0.101    0.000    0.101    0.000 {built-in method builtins.print}
     8000    0.100    0.000    0.129    0.000 {method 'readlines' of '_io._IOBase' objects}
    93000    0.097    0.000    0.111    0.000 re.py:286(_compile)
    93000    0.069    0.000    0.069    0.000 {method 'search' of '_sre.SRE_Pattern' objects}
    93000    0.030    0.000    0.141    0.000 re.py:231(compile)
    17000    0.019    0.000    0.029    0.000 codecs.py:318(decode)
        1    0.017    0.017    0.911    0.911 grep.py:3(<module>)

[omitted lines]
```

Napomena za Pythonov `cProfile` profajler (i mnoge drugi profajlere koji se toga tiču) jeste da oni prikazuju vrijeme za pojedinačan poziv funkcije. To može postati neintuitivno veoma brzo, posebno ako koristite biblioteke treće strane u vašem kodu budući da se unutrašnji pozivi funkcija takođe računaju. Intuitivniji način prikazivanja informacija o profilisanju jeste da uključe vrijeme koje je potrošeno za pojedinačnu liniju koda, što je ono što __line profilers__ rade.

Na primer, sledeći dio Python koda izvršava zahtjev na sajtu i parsira odgovor da bi dobio sve URL-ove na stranici: 

```python
#!/usr/bin/env python
import requests
from bs4 import BeautifulSoup

# This is a decorator that tells line_profiler
# that we want to analyze this function
@profile
def get_urls():
    response = requests.get('https://missing.csail.mit.edu')
    s = BeautifulSoup(response.content, 'lxml')
    urls = []
    for url in s.find_all('a'):
        urls.append(url['href'])

if __name__ == '__main__':
    get_urls()
```

Ako bi koristili Pythonov `cProfile` profajler dobili bi preko 2500 linija outputa, čak i sa sortiranjem bi bilo teško da shvatimo gdje se zapravo troši vrijeme. Brzo izvršavanje sa [line profiler](https://github.com/rkern/line_profiler) pokazuje vrijeme potrošeno po liniji.

```python
$ kernprof -l -v a.py
Wrote profile results to urls.py.lprof
Timer unit: 1e-06 s

Total time: 0.636188 s
File: a.py
Function: get_urls at line 5

Line #  Hits         Time  Per Hit   % Time  Line Contents
==============================================================
 5                                           @profile
 6                                           def get_urls():
 7         1     613909.0 613909.0     96.5      response = requests.get('https://missing.csail.mit.edu')
 8         1      21559.0  21559.0      3.4      s = BeautifulSoup(response.content, 'lxml')
 9         1          2.0      2.0      0.0      urls = []
10        25        685.0     27.4      0.1      for url in s.find_all('a'):
11        24         33.0      1.4      0.0          urls.append(url['href'])
```

### Memorija

U jezicima kao što su C ili C++ curenje memorije može prouzrokovati problem da se nikada ne oslobodi memorija koja se više ne koristi. Za pomoć u procesu ispravljanja grešaka u memoriji možete koristiti alate kao što je [Valgrind](https://valgrind.org/) koji će vam pomoći da identifikujete curenje memorije.

U jezicima sa sakupljačima smeća kao što je Python i dalje je korisno koristiti profajler memorije jer dokle god imate pokazivače na objekte u memoriji njih čistač neće očistiti. Evo primjera programa i njegovog povezanog outputa kada ga pokrećete sa [memory-profiler](https://pypi.org/project/memory-profiler/) (primjetite dekorator kao u `line-profiler`).

```python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```

```python
$ python -m memory_profiler example.py
Line #    Mem usage  Increment   Line Contents
==============================================
     3                           @profile
     4      5.97 MB    0.00 MB   def my_func():
     5     13.61 MB    7.64 MB       a = [1] * (10 ** 6)
     6    166.20 MB  152.59 MB       b = [2] * (2 * 10 ** 7)
     7     13.61 MB -152.59 MB       del b
     8     13.61 MB    0.00 MB       return a
```

### Profilisanje događaja 

Kao što je bio slučaj sa `strace` za ispravljanje grešaka, možda ćete željeti da ignorišete specifičnosti koda koji izvršavate i tretirate ga kao crnu kutiju u toku profilisanja. [perf](https://www.man7.org/linux/man-pages/man1/perf.1.html) komanda sažima CPU razlike i ne izvještava o vremenu ili memoriji, umjesto toga izvještava sistemske događaje koji su vezani za vaš program. Na primer `perf` može lako izvjestiti o lošem keš lokalitetu, velikoj količini grešaka ili nepostojećih stranica. Evo pregleda komandi: 

- `perf list` - Navodi događaje koji mogu biti praćeni sa perf-om
- `perf stat COMMAND ARG1 ARG2` - Uzima broj različitih događaja koji su povezani sa procesom ili komandom
- `perf record COMMAND ARG1 ARG2` - Zapisuje izvršavanje komande i čuva statističke podatke u datoteci koja se zove `perf.data`
- `perf report` - Formatira i ispisuje podatke sačuvane u `perf.data`

### Vizualizacija

Output profajlera za stvarne programe će sadržati veliku količinu informacija zbog kompleksnosti softverskog projekta. Ljudi su vizuelna bića i prilično su loši u čitanju velike količine brojeva koji bi trebali da imaju smisao. Dodatno, postoji mnogo alata za prikazivanje outputa profajlera na lakši način.

Jedan čest način da bi se prikazalo profilisanje CPU informacija za uzeti profajler jeste korišćenje [Flame Graph](http://www.brendangregg.com/flamegraphs.html) koji će prikazati hijerarhiju poziva funkcije preko Y ose i korišćeno vrijeme proporcionalno X osi. Takođe su interaktivni, dopuštaju vam da zumirate specifične dijelove programa i da dobijete njihove stack tragove (pokušajte da kliknete na sliku ispod).

![img1][img1]

[img1]: http://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg

Grafikoni poziva ili grafikoni kontrolnog toka prikazuju odnose između potprograma unutar programa uključujući funkcije kao nodove i pozive funkcije između njih kao direktne ivice. U kombinaciji sa podacima za profilisanje kao što su broj poziva i trajanje vremena, grafikoni poziva mogu biti veoma korisni za interpretiranje toka programa. U Pythonu možemo koristiti [pycallgraph](http://pycallgraph.slowchop.com/en/master/) biblioteku da bi ih generisali.

![img2][img2]

[img2]: https://upload.wikimedia.org/wikipedia/commons/2/2f/A_Call_Graph_generated_by_pycallgraph.png

### Praćenje resursa

Nekada, prvi korak ka analiziranju performansi vašeg programa je razumijevanje koja je njegova stvarna potrošnja resursa. Programi se često izvršavaju sporo kada su ograničeni resursi npr. bez dovoljno memorije ili na sporoj network konekciji. Postoji mnoštvo alata komandne linije za ispitivanje i prikazivanje različitih sistemskih resursa kao što je upotreba CPU, upotreba memorije, upotreba diska i tako dalje.

- **Generalno praćenje** - Vjerovatno je najpopularniji [htop](https://hisham.hm/htop/index.php), koji je poboljšana verzija [top](https://www.man7.org/linux/man-pages/man1/top.1.html). `htop` predstavlja različite statistike za trenutni proces koji je pokrenut na sistemu. `htop` ima mnoštvo opcija i tasterskih prečica, a neke korisne su: `<F6>` za sortiranje procesa, `t` da bi se pokazala hijerarhija drveta i `h` za prebacivanje niti. Takođe pogledajte [glances](https://nicolargo.github.io/glances/) za sličnu implementaciju sa odličnim UI. Za dobijanje agregatnih mjera kroz sve procese, [dstat](http://dag.wiee.rs/home-made/dstat/) je još jedan sjajan alat koji računa metričke resurse u realnom vremenu za više različitih podsistema kao što su I/O, networking, CPU upotreba, prebacivanje konteksta itd.
- **I/O operacije** - [iotop](https://www.man7.org/linux/man-pages/man8/iotop.8.html) prikazuje uživo I/O upotrebu informacija i pogodan za provjeru da li proces radi teške I/O disk operacije
- **Korišćenje diska** - [df](https://www.man7.org/linux/man-pages/man1/df.1.html) prikazuje metriku po particiji i [du](https://man7.org/linux/man-pages/man1/du.1.html) prikazuje korišćenje diska po fajlu za trenutni direktorijum. U ovim alatima `-h` flag govori programu da ispiše format koji je čitljiv za ljude. Interaktivnija verzija `du`-a je [ncdu](https://dev.yorhel.nl/ncdu) koji vam dopušta da se krećete kroz foldere i brišete fajlove i foldere u toku kretanja.
- **Korišćenje memorije** - [free](https://www.man7.org/linux/man-pages/man1/free.1.html) prikazuje totalnu količinu slobodne i korišćene memorije u sistemu. Memorija se takođe prikazuje u alatima kao što je `htop`.
- **Otvoreni Fajlovi** - [lsof](https://www.man7.org/linux/man-pages/man8/lsof.8.html) prikazuje informacije o fajlovima koji su otvoreni od strane procesa. Može biti veoma korisno za provjeru koji proces ima otvoreno specifičan fajl.
- **Network konekcija i konfiguracija** - [ss](https://www.man7.org/linux/man-pages/man8/ss.8.html) vam dopušta da pratite dolazeće i odlazeće network statističke pakete kao i interfejs statistike. Uobičajen način korišćenja `ss` jeste otkrivanje koji proces koristi dati port u mašini. Za prikazivanje routinga, network uređaja i interfejsa možete koristiti [ip](https://man7.org/linux/man-pages/man8/ip.8.html). Imajte u vidu da su `netstat` i `ifconfig` prevaziđeni u korist drugih alata.
- **Korišćenje networka** - [nethogs](https://github.com/raboof/nethogs) i [iftop](http://www.ex-parrot.com/pdw/iftop/) su dobri interaktivni CLI alati za praćenje korišćenja networka.

Ukolko želite da testirate ove alate možete vještački nametnuti opterećenja mašini koristeći [stress](https://linux.die.net/man/1/stress) komandu.

### Specijalizovani alati

Ponekad, mjerenje crne kutije je sve što vam je potrebno da bi utvrdili koji softver da koristite. Alati kao što su [hyperfine](https://github.com/sharkdp/hyperfine) vam dopuštaju brzo mjerenje komandne linije programa. Na primer, u lekciji o shell alatima i skriptingu preporučivali smo `fd` u odnosu na `find`. Možemo koristiti `hyperfine` da bi ih uporedili u zadacima koje često izvršavamo. Npr. u primjeru ispod `fd` je bio 20 puta brži nego `find` na mojoj mašini.

```shell
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

Kao što je bio slučaj sa ispravljanjem grešaka, pretraživači takođe dolaze sa odličnim setom alata za profilisanje učitavanja stranice, dopuštajući vam da shvatite gdje se vrijeme troši (učitavanje, renderovanje, skripting itd.). Više informacija na [Firefox](https://developer.mozilla.org/en-US/docs/Mozilla/Performance/Profiling_with_the_Built-in_Profiler) i [Chrome](https://developers.google.com/web/tools/chrome-devtools/rendering-tools).

## Vježbe

### Ispravljanje grešaka

1. Koristite `journalctl` na Linuxu ili `log show` na macOS da bi dobili komande od strane super korisnika u poslednjem danu. Ukoliko ih nema, možete izvršiti neke bezopasne komande kao što je `sudo ls` i provjeriti ponovo.
2. Uradite [ovaj](https://github.com/spiside/pdb-tutorial) `pdb` tutorijal da bi se upoznali sa komandama. Za dublji tutorijal pročitajte [ovo](https://realpython.com/python-debugging-pdb/).
3. Instalirajte [spellcheck](https://www.shellcheck.net/) i pokušajte da provjerite sledeću skriptu. Šta nije u redu sa ovim kodom? Popravite ga. Instalirajte linter plugin u vašem editoru tako da možete automatski da dobijete upozorenja.

```shell
#!/bin/sh
## Example: a typical script with several problems
for f in $(ls *.m3u)
do
  grep -qi hq.*mp3 $f \
    && echo -e 'Playlist $f contains a HQ file in mp3 format'
done
```

4. (Napredno) Pročitajte o [reversible debugging](https://undo.io/resources/reverse-debugging-whitepaper/) i uzmite jednostavan primjer kao što je [rr](https://rr-project.org/) ili [RevPDB](https://morepypy.blogspot.com/2016/07/reverse-debugging-for-python.html).

### Profilisanje

5. [ovdje](https://missing.csail.mit.edu/static/files/sorts.py) su neke sortirajuće implementacije algoritama. Koristite [cProfile](https://docs.python.org/3/library/profile.html) i [line_profiler](https://github.com/rkern/line_profiler) da bi uporedili vrijeme izvoženja sortiranja i brzog sortiranja. Šta je usko grlo svakog od algoritama? Zatim koristite `memory_profiler` da bi provjerili upotrebu memorije, zašto je umetnuto sortiranje bolje? Provjerite sada inplace verziju brzog sortiranja. Izazov: Koristite `pref` da bi pogledali brojanje ciklusa i keširanje udara i propusta svakog od algoritama.

6. Evo jednog (iskrivljenog) Python koda za računanje Fibonačijevih brojeva korišćenjem funkcije za svaki broj.

```python
#!/usr/bin/env python
def fib0(): return 0

def fib1(): return 1

s = """def fib{}(): return fib{}() + fib{}()"""

if __name__ == '__main__':

    for n in range(2, 10):
        exec(s.format(n, n-1, n-2))
    # from functools import lru_cache
    # for n in range(10):
    #     exec("fib{} = lru_cache(1)(fib{})".format(n, n))
    print(eval("fib9()"))
```
Stavite kod u fajl i podesite da je izvršan. Instalirajte [pycallgraph](http://pycallgraph.slowchop.com/en/master/). Pokrenite kod kakav jeste sa `pycallgraph graphviz -- ./fib.py` i provjerite `pycallgraph.png` fajl. Koliko je puta `fib0` pozvan? Možemo uraditi bolje od toga optimizovanjem funkcija. Maknite iz komentara linije koje se nalaze u komentaru i regenerišite slike. Koliko puta mi pozivamo `fibN` funkciju sada?

7. Čest problem je što je port koji želite da slušate već zauzet od strane drugog procesa. Hajde da naučimo kako da saznamo ID procesa. Prvo izvršite `python -m http.server 4444` da bi pokrenuli minimalistički web server slušajući na portu `4444`. Na odvojenim terminalima pokrenite ` lsof | grep LISTEN` da bi ispisali sve procese i portove koji se slušaju. Pronađite ID tog procesa i prekinite ga komandom `kill <PID>`.

8. Ograničavanje resursa za procese može biti još jedan zgodan alat koji posjedujete. Pokušajte pokretanje `stress -c 3` i vizualizujte potrošnju CPU-a sa `htop`. Sada izvršite `taskset --cpu-list 0,2 stress -c 3` i vizualizujte ga. Da li je `stress` zauzeo tri CPU-a? Zbog čega nije? Pročitajte [man taskset](https://www.man7.org/linux/man-pages/man1/taskset.1.html). Izazov: ostvarite isto koristeći [cpgroups](https://www.man7.org/linux/man-pages/man7/cgroups.7.html). Pokušajte da ograničite korišćenje memorije `stress -m`.

9. (Napredno) komanda `curl ipinfo.io` izvršava HTTP zahtjev i povlači informacije o vašoj javnoj IP adresi. Otvorite [Wireshark](https://www.wireshark.org/) pokušajte da prođete kroz request i reply pakete koje `curl` šalje i prima. (Nagovještaj: koristite `http` filter da bi samo posmatrali HTTP pakete).
