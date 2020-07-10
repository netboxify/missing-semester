# Pregled kursa + shell

<a href="http://www.youtube.com/watch?feature=player_embedded&v=Z56Jmr9Z34Q
" target="_blank"><img src="" 
alt="Lecture 1: Course Overview + The Shell (2020)" width="240" height="180" border="10" /></a>

# Motivacija

Kao inženjeri računarskih nauka, znamo da su računari odlična pomoć u zadacima koji se ponavljaju. Ipak, veoma često, zaboravljamo da se to jednako odnosi na našu upotrebu računara koliko i na radnje koje želimo da naši računari obave. Na raspolaganju nam je širok spektar alata koji nam pružaju mogućnost da budemo produktivniji i riješimo složenije probleme tokom rada na bilo kom problemu koji se tiče računara. Ipak, mnogi od nas koriste samo mali dio tih alata, znamo samo kako da se izborimo sa manjim problemima, i da kopi-pestujemo komandne sa interneta kada 'zaglavimo'.

Cilj ove lekcije jeste da navedeni problem riješi.

Želimo da Vas naučimo kako da najbolje iskoristite alate koje znate, upoznamo Vas sa novim alatima, i nadamo se da postaknemo uzbuđenje kod Vas za samostalno istraživanje (a možda i izradu više alata). To je ono za šta vjerujemo da je nedostajući semestar u većini nastavnih planova CS-a.

# Struktura časa

Lekcije se sastoje od 11 lekcija koje traju 1 sat, a svaka se tiče [određene teme](https://missing.csail.mit.edu/2020/). Lekcije su u velikoj mjeri nezavisne, a kako se semestar odvija, pretpostavićemo da ste upoznati sa sadržajem iz ranijih predavanja. Imamo zabilješke online, ali će biti dosta sadržaja koji će biti pokriven u lekcijama (npr. u formi demo-a) koji možda neće biti u zabilješkama. Lekcije ćemo snimati, i postaviti ih online. 

Pokušaćemo da pokrijemo dosta toga tokom kursa koji se sadrži od samo 11 lekcija koje traju 1 sat, tako da će lekcije biti prilično 'guste'. Da bi Vam pružili malo vremena da se upoznate sa sadržajem u Vašem sopstvenom ritmu, svaka lekcija sadrži set vježbi koje Vas vode kroz ključne tačke u lekciji. Nakon svake lekcije, ugostićemo Vas u kancelariji, gdje ćemo odgovoriti na sva potencijalna pitanja. Ako prisustvuje lekciji online, možete nam poslati pitanje na missing-semester@mit.edu

Uzimajući u obzir vrijeme kojim smo ograničeni, nećemo biti u mogućnosti da pređemo sve alate na istom nivou detalja kao što bio slučaj sa punim časovima. Gdje god je to moguće, pokušaćemo da Vas usmjerimo ka resursima sa dodatnim informacijama vezanim za alat ili temu, ali ukoliko Vam nešto posebno zapadne za oko, ne oklijevajte da nas kontaktirate i pitate za smjernice.

# Tema 1: Shell

## Šta je shell?

Računari danas imaju širok spektar interfejsa za zadavanje komandi, fensi grafički korisnički interfejs, glasovni interfejs, čak su i AR/VR svuda. Ovo su dobra rešenja za 80% slučajeva, ali su često suštinski ograničeni u pogledu toga što Vam omogućuju da radite - ne možete da pritisnete dugme koje nije tu ili da date glasovnu komandu koja nije programirana. Da bi iskusili sve mogućnosti koje alati Vašeg računara pružaju, moramo da koristimo stari način i upotrebimo tekstualni interfejs: Shell.

Gotovo sve platforme kojima imate pristup imaju shell u jednoj formi ili drugoj, a mnogi od njih imaju više shell-ova između kojih možete da birate. Dok postoje razlike u detaljima, oni su suštinski isti: dopuštaju vam da pokrenete programe, date inpute, i inspektuje njihov rezultat u polu-struktuiranom načinu.

U ovoj lekciji, fokusiraćemo se na Bourne Again SHell, ili skraćeno 'bash'. Ovo je jedan on najčešće korišćenih shell-ova, i njegova sintaksa je slična onome što ćete vidjete u mnogim drugom shell-ovima. Da bi otvorili shell prompt (gdje možete da kucate komande), prvo vam je potreban terminal. Vaš uredjaj je vjerovatno došao sa jednim pre-instaliranim, ili možete relativno lako da instalirate jedan.

In this lecture, we will focus on the Bourne Again SHell, or “bash” for short. This is one of the most widely used shells, and its syntax is similar to what you will see in many other shells. To open a shell prompt (where you can type commands), you first need a terminal. Your device probably shipped with one installed, or you can install one fairly easily

## Korišćenje shell-a

Kada pokrenete vaš terminal, vidjećete _prompt_ koji obično izgleda ovako:

```console
missing:~$
```

Ovo je glavni tekstualni interfejs shell-a. Govori vam da ste na mašini `missing` i da vaš "trenutni radni direktorijum", ili gdje se trenutno nalazite, je ~ (skraćeno za "home"). $ vam govori da niste root korisnik (više o tome kasnije). U ovom prompt-u možete pisati komande, koje će zatim biti interpretirane od strane shell-a. Najjednostavnija komanda da bi izvršili program je: 

```console
missing:~$ date
Fri 10 Jan 2020 11:49:31 AM EST
missing:~$
```

Ovdje, mi izršavamo date program, koji (možda na iznenadjenje) ispisuje trenutni datum i vrijeme. Shell vas zatim pita za drugu komandu za izvršavanje. Takođe, možemo izvršiti program sa argumentima:

```console
missing:~$ echo hello
hello
```

U ovom slučaju smo zadali shell-u da izvrši program echo sa argumentom hello. Echo program jednostavno ispisuje zadate argumente. Shell analizira komandu dijeleći je na bijeli prostor, a zatim izvršava program naznačen prvom riječju, posmatrajući svaku sledeću riječ kao argument kojem program može da pristupi. Ukoliko želite da omogućite argument koji sadrži razmake ili druge specijalne karaktere (npr. direktorijum po imenu "My photos"), možete citirati argument sa `'` ili `""` `("My Photos")`, ili izbjeći samo relevantne karaktere sa `\` `(My\ Photos)`.

Ali kako shell zna kako da pronađe datum ili echo program? Pa, shell je programsko okruženje, isto kao i Python ili Ruby, tako da ima varijable, kondicionale, petlje, funkcije (sledeća lekcija!). Kada pokrenete komandu u vašem shellu, vi zapravo pišete male dijelove koda koji shell interpretira. Ukoliko je shell upitan da izvrši komandu koja se ne poklapa sa bilo kojom od njegovih programskih ključnih riječi, on konsultuje _environment variable_ zvano $PATH koje navodi u kojem direktorijumu bi shell trebao da traži program kada mu je zadata komanda:

```console
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
Kada pokrenemo `echo` komandu, shell vidi da bi trebao da izvrši program `echo`, i onda pretražuje kroz `:`-odvojenu listu direktorijuma u `$PATH`-u za fajl sa tim imenom. Kada ga pronađe, pokreće ga (uz pretpostavku da je fajl izvršnog tipa, više o tome kasnije). Možemo pronaći koji fajl se izvršava za zadato ime programa koristeći `which` program. Takođe možemo kompletno izbjeći `$PATH` zadavanjem putanje fajlu koji želimo da izvršimo.

## Kretanje kroz shell

Putanja u shell-u je ograničena lista direktorijuma; odvojena `/` na Linux i macOS i `\` na Windows-u. Na Linux i macOS, putanja `/` je 'root' sistema datoteka, ispod koje se nalaze svi drugi direktorijumi i fajlovi, dok na Windows/u postoji postoji jedan root za svaku disk particiju (npr., `C:\\`). Polazimo od pretpostavke da koristite Linux fajl sistem u ovim lekcijama. Putanja koja počinje  sa `/` je apsolutna putanja. Svaka druga putanja je relativna putanja. Relativne putanje su relativne u odnosu na trenutni radni direktorijum, koji možemo vidjeti koristeći `pwd` komandu, i promijeniti je sa `cd` komandom. U putanji, `.` se odnosi na trenutni direktorijum, a `..` na parent direktorjum.

```console
missing:~$ pwd
/home/missing
missing:~$ cd /home
missing:/home$ pwd
/home
missing:/home$ cd ..
missing:/$ pwd
/
missing:/$ cd ./home
missing:/home$ pwd
/home
missing:/home$ cd missing
missing:~$ pwd
/home/missing
missing:~$ ../../bin/echo hello
hello
```

Primjećujete da nas naš shell prompt izvještava šta je naš trenutni direktorijum bio. Možete podesiti vaš prompt da vam pokazuje razne vrste korisnih informacija, koje ćemo preći u kasnijim lekcijama.

Uopšteno, kada pokrenemo program, on će raditi u trenutnom direktorijumu, osim ako mu ne kažemo drugačije. Na primer, obično će tamo pretražiti datoteke i kreirati nove datoteke ukoliko je to potrebno.

Da bi vidjeli postojeće fajlove u zadatom direktorijumu, koristimo komandu `ls` :

```console
missing:~$ ls
missing:~$ cd ..
missing:/home$ ls
missing
missing:/home$ cd ..
missing:/$ ls
bin
boot
dev
etc
home
...
```

Osim ako je direktorijum pružen kao prvi argument, `ls` će ispisati sadržaj trenutnog direktorijuma. Većina komandi prihvata flags i opcije (flags sa vrijednostima) koje počinju sa `-` da bi modifikovali njihovo ponašanje. Obično, pokretanje programa sa `h` ili `--help` flag-om(`/?` na Windowsu) će ispisati neki pomoćni tekst koji će vam reći koji flags i opcije su dostupne. Na primer `ls --help` će nam reći:


```console
-l                         use a long listing format
```

```console
missing:~$ ls -l /home
drwxr-xr-x 1 missing  users  4096 Jun 15  2019 missing
```

Ovo nam daje dosta novih informacija o svakom fajlu ili trenutnom direktorijumu. Prvo, `d` na početku linije nam govori da je `missing` direktorijum. Zatim slijedi grupa od tri karaktera (`rwd`). Ovo ukazuje na to koje dozvole vlasnk fajla (`missing`), vlasnička grupa (`users`) i svako drugi ima u pomenutom item-u. `-` Ukazuje da trenutni nalogodavac nema dato odobrenje. Jedino je vlasnik ovlašćen da mijenja (`w`) direktorijum `missing` (npr. dodaje i brise fajlove). Da bi pristupio direktorijumu, user mora imati "search" (koji je oznacen kao "execute": `x`) dozvolu na tom direktorijumu (i njegovom parent-u). Da bi izlistao njegov sadržaj, korisnik mora imati read (`r`) dozvolu na tom direktorijumu. Za fajlove, dozvole su onakve kako i pretpostavljate. Primjećujete da gotovo svi fajlovi u `/bin` imaju `x` dozvolu za poslednju od grupa "svi ostali", tako da svako može izvršiti ove programe.

Još jedan pogodni program, koji bi u ovom trenutku bilo dobro da znate jeste `mv` (da promenite naziv fajla ili da ga premjestite), `cp` (da kopirate fajl), i `mkdir`(da napravite novi direktorijum).

Ukoliko budete imali potrebu za više informacija koje su vezane za argumente programa, inpute, outpute, ili kako program generalno funkcioniše, pokušajte sa `man` programom. On uzima kao argument ime programa, i prikazuje vam uputstvo za navedeni program. Pritisnite `q` da bi izašli.

```console
missing:~$ man ls
```

## Povezivanje programa

U shell-u programi imaju dva glavna toka koji su povezani sa njima: ulazni i izlazni tok. Kada program pokuša da čita input, čita se iz ulaznog toka, a kada nešto štampa, štampa u svom izlaznom toku. Obično se i input i output programa nalaze u vašem terminalu. Odnosno, vaša tastatura je vaš input, a vaš monitor je vaš output. Ipak, i te tokove možemo povezati. 

Najjednostavnija forma preusmjeravanja je `< file` i `> file`. Ovo vam pruža mogućnost da povežete input i output toka programa u fajlu: 

```console
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
```

Takođe možete da koristite `>>` da dodate fajlu. Ovakva vrsta redirekcije inputa/outputa je najkorisnije u upotrebi `pipes`-a. Operator `|` vam dopušta `chain` programa na način što je output jednog u stvari input od drugog. 

```console
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```

Ućićemo u mnogo više detalja o tome kako da se najbolje iskoristi pipes u lekciji o obradi podataka.

## Svestran i moćan alat

Na većini UNIX-like sistema, jedan korisnik je poseban: `root` korisnik. Možda ste ga vidjeli u gore navednim datotekama. Root korisnik je izvan (gotovo) svih ograničenja, i u mogućnosti je da kreira, pregleda, ažurira i briše bilo koji fajl u sistemu. Obično se nećete prijavljivati u vašem sistemu kao root korisnik, jer se lako desi da slučajno pokvarite nešto u sistemu. Umjesto toga, koristićete `sudo` komandu. Kao što sami naziv govori, dopušta vam da uradite (do) nešto kao "su" (skraćenica za "super user", ili "root"). Kada primite grešku o zabrani dozvole, to je obično zbog toga što nešto morate da odradite kao "root". Ipak, dva puta provjerite da li je baš ovo način na koji želite da odradite nešto.

Da bi bili root, postoji jedna stvar koju bi trebalo da uradite, a to je pišete u `sysfs` fajl sistem postavljen kao `/sys.sysfs` otkrivajući broj kerner parametara kao fajlova, tako da lako možete da rekonfigurišete kernel u hodu bez specijalnih alata. **Imajte u vidu da sysyfs ne postoji na Windows-u ili macOS-u.**

Na primer, osvijetljenje vašeg laptop ekrana se otkriva kroz fajl koji se zove `brightness` 

`/sys/class/backlight`

Upisivajući vrijednost u tom fajlu, možemo promijeniti osvijetljenje ekrana. Vaš prvi instikt bi vam mogao reći da uradite nešto ovako: 

```console
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```

Ova greška vas može iznenaditi. Nakon svega, pokrenuli smo komandu sa `sudo`! Ovo je jedna vrlo važna stvar koju treba znati o shell-u. Operatori kao što su `|`, `>`, i `<` su obrađeni od strane shell-a, a ne od strane individualnih programa. Echo i prijatelji ne znaju ništa o `|`. Oni samo čitaju iz njihovih inputa i ispisuju u njihove outpute, koji god to bili. U slučaju iznad, shell (koji je potvrđen samo kao vaš korisnik), pokušava da otvori brightness fajl da bi pisao u njemu, prije nego što je podešen kao sudo echo output, ali je odvraćen od toga jer se shell ne pokreće kao root. Koristeći ovo saznanje, možemo pronaći način da ovo odradimo: 

```console
$ echo 3 | sudo tee brightness
```

Kako je `tee` program koji ćemo koristiti za otvaranje `/sys` fajlova za pisanje, i pokreće se kao root, sve dozvole su odobrene. Možete kontrolisati razne korisne stvari kroz `sys`, kao što su stanja različitih sistema LED-a (vaša putanja može biti različita):

```console
$ echo 1 | sudo tee /sys/class/leds/input6::scrolllock/brightness
```
