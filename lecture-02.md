# Shell alati i scripting

<a href="http://www.youtube.com/watch?feature=player_embedded&v=kgII-YWo3Zw
" target="_blank"><img src="" 
alt="Lecture 2: Shell Tools and Scripting (2020)" width="240" height="180" border="10" /></a>

U ovoj lekciji, predstavićemo neke osnove koje se tiču korišćenja bash-a kao scripting jezika zajedno sa više shell alata koji pokrivaju nekoliko najčešćih zadataka koje ćete konstantno izvoditi kroz komandnu liniju.

## Shell scripting

Do sada smo vidjeli kako da izvršimo komande u shell-u i kako da ih spojimo. Ipak, u mnogim slučajevima željećete da izvršite seriju komandi i iskoristite kontrolu niza izraza kao što su uslovni iskazi ili petlje.

Shell skripte su sledeći korak u kompleksnosti. Većina shell-ova imaju svoje skripting jezike sa varijablama, kontrolom toka i ličnom sintaksom. Ono što čini shell skripting različitim od ostalih skripting programskih jezika jeste da je optimizovan za izvršavanje zadataka koji se tiču shell-a. Takođe, kreiranje komandnih pajplajna, čuvanje rezultata u fajlovima, i čitanja kroz standardni input su primitive u shell skriptingu, koje je lakše koristiti u odnosu na skripting jezike koji su generalne prirode. Za ovu sekciju mi ćemo se fokusirati na bash skripting, budući da je on najčešći.

Da bi dodali varijble u bash-u, koristite sintaksu `foo=bar` i pristupite vrijednosti varijable sa `$foo`. Imajte u vidu da `foo = bar` neće raditi, s obzirom na to da je interpretirano kao pozivanje foo programa sa argumentima `=` i `bar`. Generalno u shell skriptingu razmak će izvršiti dijeljenje argumenata. Ovo ponašanje može biti zbunjujuće iz prve, pa uvijek imajte to u vidu.

Stringovi u bash-u mogu biti definisani sa `'` i `"` navodnicima, ali oni nisu jednaki. Stringovi označeni sa `'` su literalni stringovi i neće zamijeniti vrijednost varijable, dok `"` navodnici hoće.

```shell
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```
Kao i sa većinom programskih jezika, bash podržava kontrolu toka sa tehnikama koje uključuju `if`, `case`, `while` i `for`. Slično, `bash` ima funkcije koje primaju argumente i dozvoljava vršenje operacija sa njima. Evo primjera funkcija koja kreira direktorijum i vrši cd u njega.

```shell
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

Ovdje je `$1` prvi argument skripti/funkciji. Za razliku od drugih skripting jezika, bash koristi širok spektar specijalnih varijabli koje ukazuju na argumente, error kodove, i druge relevantne varijable. Ispod je lista nekih od njih. Obimnija lista se može naći [ovdje](https://www.tldp.org/LDP/abs/html/special-chars.html)

- `$0` = Naziv skritpte
- `$1` do `$9` - Argumenti skripte. `$1` je prvi argument i tako dalje.
- `$@` - Svi argumenti
- `$#` - Broj argumenata
- `$?` - Vraća kod od prethodne komande
- `$$` - Indetifikacioni broj procesa (IBP) za trenutnu skriptu
- `!!` - Čitava poslednja komanda, uključujući argumente. Često se koristi da bi se izvršila komanda koja je prethodno podbacila zbog dozvola koje nedostaju; Možete brzo re-izvršiti komandu sa sudo koristeći `sudo !!`
- `$_` - Poslednji argument od poslednje komande. Ako ste u interaktivnom shell-u, možete takođe brzo dobiti ovu vrijednost kucanjem `Esc` i zatim `.`

Komande će često vratiti output koristeći `STDOUT`, greške kroz `STDERR`, i Return Code da bi izvijestile o greškama na način koji je više skript-friendly. Kod koji se vraća ili exit status je način na koji skript/komande moraju da komuniciraju o tome kako je izvršavanje proteklo. Vrijednost 0 obično znači da je sve proteklo dobro; Bilo šta što je drugačije od 0 znači da je nastupila greška.

Exit kodovi se mogu koristiti da bi se uslovno izvršavala komanda koristeći `&&` (i operator) i `||` (ili operator), gdje su i jedan i drugi [short-circuiting](https://en.wikipedia.org/wiki/Short-circuit_evaluation) operatori. Komande takođe mogu biti razdvojene u okviru iste linije koristće tačku zarez `;`. `true` program će uvijek imati 0 kao return kod, a komanda koja je `false` će uvijek imati 1 kao return kod. Hajde da vidimo neke primjere.

```shell
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

true ; echo "This will always run"
# This will always run

false ; echo "This will always run"
# This will always run
```
Još jedan poznati obrazac je kada želite da dobijete output komande kao varijablu. Ovo može biti odrađeno zamjenom komandi. Kada god postavite `$( CMD )` izvršiće `CMD`, uzeti output komande i zamijeniti je na mjestu. Na primer, ukoliko uradite `for file in $(ls)`, shell će prvo pozvati `ls` i onda izvršiti iteraciju kroz te vrijednosti. Sličan način, koji je manje poznat jeste proces zamjene, `<( CMD )` će izvršiti `CMD` i postaviti output u trenutni fajl u zamijeniti `<()` sa nazivom toga fajla. Ovo je korisno kada komande očekuju vrijednosti da im budu proslijeđene od strane fajla umjesto od strane `STDIN`. Na primer, `diff <(ls foo) <(ls bar)` će pokazati razlike između fajlova u direktorijumu foo i bar.

Kako je ovdje prikazano jako puno informacija, hajde da vidimo primjer koji prikazuje primjenu ovih stvari. Izvršiće iteraciju kroz argumente koje smo obezbijedili, 
`grep` za string `foobar`, i dodaće ih fajlu kao komentar ukoliko nije pronađen.

```shell
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```
U poređenju smo testirali da li `$?` nije bilo jednako 0. Bash sprovodi mnoga poređenja ovakve vrste - možete pronaći detaljnu listu u man stranici za [test](https://www.man7.org/linux/man-pages/man1/test.1.html). Kada se izršava poređenje u bash-u, pokušajte da više koristite duple zagrade `[[ ]]` u odnosu na obične zagrade `[ ]`. Šanse da napravite greške su slabije, iako nisu prenosive na `sh`. Detaljnije objašnjenje može biti pronađeno [ovdje](mywiki.wooledge.org/BashFAQ/031).

Kada pokrećemo skriptu, često ćete željete da pružite argumente koji su slični. Bash ima način da se ovo olakša, širenjem izraza provođenjem expanzije imena fajla. Ove tehnike se česte nazivaju shell _globbing_.

- Wildcards - Kada god želite da izvedete neku vrstu wildcard podudaranja, možete koristiti `?` i `*` da bi se podudarili sa jednim ili više karaktera. Na primer, dati fajlovi `foo`, `foo1`, `foo2`, `foo10` i `bar`, i komanda `rm foo?` će izbrisati `foo1` i `foo2`, dok će `rm foo*` izbrisati sve osim `bar`.

- Vitičaste zagrade {} - Kada god imate zajednički substring u seriji komandi, možete koristiti vitičaste zagrade za bash da bi proširili ovo automatski. Ovo je veoma korisno kada pomjerate ili konvertujete fajlove.

```shell
convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Will expand to
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files


mkdir foo bar
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
# Outputs
# < x
# ---
# > y
```

Pisanje `bash` skripti može biti izazovno i neintuitivno. Postoje alati kao što je [spellcheck](https://github.com/koalaman/shellcheck) koji će vam pomoći da pronađete greške u vašim sh/bash skriptama.

Imajte na umu da skripte ne moraju biti napisane u bash-u da bi bile pozvane iz terminala. Na primer, ovo je jednostavna Python skripta koja ispisuje argumente u obrnutom redosledu:
```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```
Kernel zna da izvrši ovu skriptu sa python interpreterom umjesto sa shell komandom zato što smo uključili [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) linije na samom vrhu skripte. Dobra praksa je da pišete shebang liniju koristeći env komandu koja će se izvršiti bez obzira koja je komanda linija u sistemu, povećavajući prenosivost vaših skripti. Da bi riješio lokaciju, env će koristiti `PATH` evironment variable koji smo predstavili u prvoj lekciji. Za ovaj primjer shebang linija će izgledati ovako: `#!/usr/bin/env python.`

Neke razlike između shell funcija i scripti koje bi trebali da imate u vidu su:

- Funkcije moraju biti u istom jeziku kao i shell, dok skripte mogu biti napisane u bilo kojem jeziku. Ovo je razlog zašto je uključivanje shebang za skripte veoma važno.
- Funkcije se učitavaju jednom kada se pročita njihova definicija. Skripte se učitavaju svaki put kada se izvršavaju. Ovo čini funkcije malo bržim za učitavanje, ali kada ih promijenite moraćete da ponovo učitate njihove definicije.
- Funkcije se izvršavaju u trenutnom shell okruženju, dok se skripte izvršavaju u njihovom posebnom procesu. Dodatno, funkcije mogu promijeniti varijable okruženja, npr. promijeniti vaš trenutni direktorijum, dok skripte ne mogu. Skripte će biti zaobiđene od vrijednosti environment varijabli koje su bile eksportovane koristeći [export](https://www.man7.org/linux/man-pages/man1/export.1p.html).
- Kao i sa bilo kojim drugim programskim jezikom, funkcije su moćan konstrukt za postizanje modularnosti, ponovne upotrebe koda, i jasnoće shell kod-a. Obično će shell skripte uključiti njihovu sopstvenu definiciju funkcije.

## Shell Tools

### Pronalaženje načina za korišćenje komandi

U ovom trenutku, vjerovatno se pitate kako da pronađete za komande u sekcijama kao što su: `ls -l, mv -i i mkdir -p`. Još šire, imajući u vidu komandu, kako saznajete šta ona radi i koje su njene opcije? Uvijek možete da koristite google, ali pošto je UNIX stariji od StackOverflow-a, postoje već ugrađeni načini da bi došli do ovih informacija.

Kao što smo vidjeli u shell lekciji, prvi pristup bi bio da pozovete komadnu sa `-h` ili `--help` flagovima. Detaljniji pristup jeste da koristite man komandu. Skraćenica za uputstvo, [man](https://www.man7.org/linux/man-pages/man1/man.1.html) pruža mogućnost stranice sa uputstvom (zvanom manpage) za komandu koju ste izabrali. Na primer, `man rm` će ispisati ponašanje rm komande zajedno sa flagovima koje prima, uključujući `-i` flag koji smo prikazali ranije. U stvari, sve linkove koje sam postavljao do sada za svaku komandu su onlajn verzije Linux manpages za komande. Čak i strane komande koje instalirate će imati manpage ukoliko ga je programer napisao i uključio u proces instalacije. Za interaktivne alate kao što su oni koji su bazirani na ncurses, može se pristupiti pomoći za programe koristeći `:help` komandi ili kucajući `?`.

Nekada manpages mogu pružiti previše detaljan opis komandi, i mogu da otežaju odluku koji flag/sintaksu će biti korišćen za standardnu upotrebu. [TLDR](https://tldr.sh/) strance su sjajno rešenje koje se fokusira na dati primjer tako da možete mnogo brže i lakše da izaberete opciju koju ćete koristiti. Lično, češće koristim stranice za [tar](https://tldr.ostera.io/tar) i [ffmpeg](https://tldr.ostera.io/ffmpeg) u odnosu na manpages.

## Traženje fajlova

Jedan od najčešćih zadataka koji se ponavljaju jeste traženje fajlova ili direktorijuma. Svi UNIX-like sistemi dolaze sa [find](https://www.man7.org/linux/man-pages/man1/find.1.html), odličnim shell alatom kojim se pretražuju fajlovi. `find` će rekurzivno tražiti fajlove koji ispunjavaju zadati kriterijum. Neki primjeri: 

```shell
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '*/test/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz'
```
Osim listinga fajlova, pretraga takođe može izvršiti akcije za fajlove koji se poklapaju sa vašim upitom. Ovo svojstvo može biti nevjerovatno korisno da bi se pojednostavilo ono što može biti jako monoton zadatak.

```shell
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec convert {} {}.jpg \
```
Uprkos sveprisutnosti pretrage, njegova sintaksa ponekad može biti nezgodna za pamćenje. Na primer, da bi se pojednostavila pretraga fajlova koji se poklapaju sa nekim obrascem, morate da izvršite `find -name '*PATTERN*' (ili -iname` ukoliko želite da obrazac za poklapanje ne bude osjetljiv na velika slova). Možete početi da pravite aliase za ove scenarije, ali dio shell filozofije je dobro pretražiti alternative. Zapamtite, jedano od najboljih svojstava shella jeste da vi samo pozivate programe, tako da možete da pronađete (ili čak i sami da napišete) zamjenu za neke. Na primer, [fd](https://github.com/sharkdp/fd) je jednostavan, brz, i user-friendly alternativa za find. Nudi neke dobre defaultne vrijednosti kao što je kolorizovani output, defaultno regex poklapanje, i Unicode podršku. Takođe ima, prema mom mišljenju, intuitivniju sintaksu. Na primer, sintaksu da se pronađe pattern `PATTERN` je `fd PATTERN`.

Većina će se složiti da su `find` i `fd` dobri, ali se možda neki od vas pitaju o efektivnosti traženja fajlova svaki put umjesto kompajliranja neke vrste index-a ili baze podataka za brzu pretragu. [Locate](https://www.man7.org/linux/man-pages/man1/locate.1.html) služi tome. `Locate` koristi bazu podataka koja se ažurira koristeći [updatedb](https://www.man7.org/linux/man-pages/man1/updatedb.1.html). U većini sistema, `updatedb` je ažuriran na dnevnom nivou preko [cron](https://www.man7.org/linux/man-pages/man8/cron.8.html). Stoga je jedan kompromis između ovo dvoje brzina protiv svježine. Pored toga `find` i slični alati mogu takođe pronaći fajlove koristeći atribute, kao što je veličina fajla, vrijeme modifikacije, dozvole fajla, dok `locate` koristi samo naziv fajla. Još dublje poređenje može biti pronađeno [ovdje](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other).

## Traženje koda

Pronalaženje datoteka po imenu je korisno, ali prilično često želite da pretražite na osnovu sadržaja datoteke. Uobičajeni scenario je da se traže sve datoteke koje sadrže neki obrazac zajedno sa onim datotekama gdje se navedeni obrazac pojavljuje. Da bi se ovo postiglo, većina UNIX-like sistema pružaju [grep](https://www.man7.org/linux/man-pages/man1/grep.1.html), generički alat za usklađivanje obrazaca iz ulaznog teksta. `grep` izuzetno vrijedan shell alat koji ćemo detaljnije obraditi u lekciji o upravljanju podacima. 

Za sada, znajte da `grep` ima mnogo flag-ova koji ga čine veoma svestranim alatom. Neke koje često koristim su `-C` za dobijanje **C**ontexta oko rezultata koji se podudaraju i `-v` za in**v**erting za preokret rezultata, odnosno ispisivanje svih linija koje se ne podudaraju sa obrascem. Na primer, `grep -C 5` će ispisati pet linija prije i nakon rezultata. Kada je u pitanju brza pretraga kroz više fajlova, željećete da koristite `-R` jer će ono rekurzivno proći kroz direktorijume, i tražiti fajlove koji se podudaraju sa zadatim stringom. 

Ali `grep -R` se može poboljšati na više načina, kao što je ignorisanje `.git` foldera, korišćenje multi CPU podrške. Razvijene su mnoge `grep` alternative, uključujući [ack](https://beyondgrep.com/), [ag](https://github.com/ggreer/the_silver_searcher) i [rg](https://github.com/BurntSushi/ripgrep). Svi od njih su odlični i pružaju otprilike iste funkcionalnosti. Za sada koristiću ripgrep (`rg`), uzimajući u obzir koliko je brz i intuitivan. Neki primjeri: 

```python
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN
```

Imajte u vidu da je sa `find`/`fd`, važno da znate da se ovakvi problemi mogu brzo riješiti korišćenjem jednim od ovih alata, dok korišćenje specifičnog alata i nije toliko bitno. 

## Traženje shell komandi 

Do sada smo vidjeli kako da pronađete datoteke i kod, ali kako budete provodili više vremena u shell-u, možda ćete željeti da pronađete specifične komande koje ste pozvali u jednom trenutku. Prva stvar koju treba da znate jeste da će vam kucanje strelice na gore vratiti poslednju komandu koju ste pozvali, ukoliko nastavite da je pritiskate polako će proći kroz vašu shell istorju. 

`history` komanda će vam pružiti mogućnost da programski pristupite istorji vašeg shell-a. Ispisaće vašu shell istorju na standardnom output-u. Ukoliko želite da pretražujete kroz nju, možete proslijediti output `grep-u` i pretražiti obrasce. `history | grep find` će ispisati komande koje sadrže substring "find".

U većini shell-ova možete koristiti `Ctrl + R` da bi izvršili pretragu unazad kroz vašu istoriju. Nakon kucanja `Ctrl + R`, možete unijeti substring za koji želite da se podudara sa komandama u vašoj istoriji. Ukolko nastavite da pritiskate kretaćete se kroz podudaranja u vašoj istoriji. Ovo takođe može biti omogućeno sa UP/DOWN strelicama u [zsh](https://github.com/zsh-users/zsh-history-substring-search). Dobar dodatak u kombinaciji sa `Ctrl + R` dolazi sa korišćenjem [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) vezivanjem. `fzf` je fuzzy pretraživač opšte namjene koji se može koristiti sa mnogim komandama. Ovdje se koristi za fuzzily podudaranje kroz vašu istoriju i prikazivanje rezultata na pogodan i vizuelno prijatan način.

Još jedan dobar trik koji se tiče istorije jesu autosugestije zasnovane na istoriji. Prvo su predstavljene od strane [fish](https://fishshell.com/) shell-a, ova funkcija dinamički dovršava vašu trenutnu shell komandu sa poslednjom komandom koju ste unijeli a koje imaju zajednički prefix. To se može omogućiti u [zsh](https://github.com/zsh-users/zsh-autosuggestions) i to je sjajan trik za vaš shell.

I na kraju, ono što treba imati u vidu, ako započnete vašu komandu sa leading space-om ona neće biti dodata u vašu istoriju. Ovo je pogodno kada pišete komande sa passwordom ili drugim osjetljivim informacijama. Ukoliko napravite grešku, pa ne dodate leading space, uvijek možete ručno ukloniti unos editovanjem `.bash_history` ili `.zhistory`.

## Navigacija kroz direktorijume 

Do sada smo pretpostavljali da se nalazite tačno tamo gdje treba da budete da bi izvršili ove akcije. Ali, šta mislite o brzom kretanju kroz direktorijume? Postoji mnogo jednostavnih načina da se ovo uradi kao što je pisanje shell alijasa ili kreiranje sym-linkova sa [ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html), ali je istina da su programeri smislili izuzetno pametno i sofisticirano rešenje do sada. 

Kao i kod teme ove lekcije, često želite da optimizujete uobičajenu upotrebu. Pronalaženje učestalih i novijih fajlova i direktorijuma se može odraditi kroz alate kao što su [fasd](https://github.com/clvv/fasd) i [autojump](https://github.com/wting/autojump). Fasd rangira fajlove i direktorijume kroz [frecency](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm) koji podrazumijeva i učestalost i rijetkost. Uobičajeno, `fasd` dodaje `z` komandu koju možete koristiti da brzo izvršite `cd` koristeći substring __frecent__ direktorijuma. Na primer, ukoliko često idete do `/home/user/files/cool_project` možete jednostavno koristiti `z cool` da tome pristupite. Koristeći autojump, ista promjena direktorijuma može biti odrađena koristeći `j cool`.

Postoje složeniji alati da biste brzo dobili pregled strukture direktorijuma: [tree](https://linux.die.net/man/1/tree), [broot](https://github.com/Canop/broot) ili čak punopravni menadzeri datoteka kao što su [nnn](https://github.com/jarun/nnn) i [ranger](https://github.com/ranger/ranger).

## Vježbe

1. Pročitajte [man ls](https://www.man7.org/linux/man-pages/man1/ls.1.html) i napišite `ls` komandu koja će ispisate fajlove na sledeće način 

- Uključuje sve fajlove, kao i skrivene fajlove
- Veličine su navedene u formatu čitljivom ljudima (npr. 454M umjesto 454279954)
- Fajlovi su posloženi do najnovijeg 
- Output je kolorizovan

Jednostavan output bi izgledao ovako

```shell
 -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
 drwxr-xr-x   5 user group  160 Jan 14 09:53 .
 -rw-r--r--   1 user group  514 Jan 14 06:42 bar
 -rw-r--r--   1 user group 106M Jan 13 12:12 foo
 drwx------+ 47 user group 1.5K Jan 12 18:08 ..
```
2. Napišite bash funkcije `marco` i `polo` koje izvršavaju sledeće. Kada god izvršite funkciju `marco`, trenutni radni direktorijum bi trebao da bude sačuvan na neki način, onda kada izvršite funkciju `polo`, bez obzira u kom se direktorijumu nalazite , `polo` bi trebao da uradi `cd` nazad u direktorijum gdje ste izvršili funkciju `marco`. Da bi lakše uklonili bugove možete napisati kod u fajlu `marco.sh` i ponovo učitali definicije u vaš shell izvršavanjem `source marco.sh`. 

3. Recimo da imate komandu koja rijetko ne uspijeva. Da biste otklonili grešku morate da snimite njen output ali može proći dosta vremena da biste ostvarili neuspjeh. Napišite bash skriptu koja pokreće sledeću skriptu dok se ne desi greška i zabilježi standardni output i ispiše greške iz datoteka i fajlova. Bonus poeni ukoliko dobijete podatak koliko je puta skripta pokrenuta prije nego se desila greška. 

```shell
 #!/usr/bin/env bash

 n=$(( RANDOM % 100 ))

 if [[ n -eq 42 ]]; then
    echo "Something went wrong"
    >&2 echo "The error was using magic numbers"
    exit 1
 fi

 echo "Everything went according to plan"
```

4. Kao što smo rekli u ovoj lekciji `find`'s `-exec` može biti veoma moćno za izvođenje operacija sa datotekama koje tražimo. Ipak, šta ukoliko želimo da uradimo nešto sa **svim** fajlovima, kao npr. da kreiramo zip fajlove? Kao što ste vidjeli do sada komande će primiti inpute iz argumenata i STDIN. Kada pajpujete komande, povezujete STDOUT I STDIN, ali neke komande kao `tar` uzimaju inpute iz argumenata. Za prevazilaženje ovog prekida postoji [xargs](https://www.man7.org/linux/man-pages/man1/xargs.1.html) komanda koja će izvršiti komandu korišćenjem STDIN-a kao argumenta. Na primer `ls | xargs rm` će izbrisati sve fajlove iz trenutnog direktorijuma. 

Vaš zadatak je da napišete komandu koja će rekurzivno pronaći sve HTML fajlove u folderu i zip-ovati ih. Imajte u vidu da bi vaša komanda trebala da radi čak ako fajlovi imaju razmake (nagovještaj: provjerite `d` flag za `xargs` )

5. (Napredno) Napišite komandu ili skriptu koja će rekurzivno pronaći poslednje podešene fajlove u direktorijumu. Dodatno, možete li razvrstati sve fajlove od najstarijeg do najnovijeg?

