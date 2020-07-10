# Shell alati i scripting

<a href="http://www.youtube.com/watch?feature=player_embedded&v=kgII-YWo3Zw
" target="_blank"><img src="" 
alt="Lecture 1: Course Overview + The Shell (2020)" width="240" height="180" border="10" /></a>

U ovoj lekciji, predstavicemo neke osnove koje se tiču korišćenja bash-a kao scripting jezika zajedno sa više shell alata koji pokrivaju nekoliko najčešćih zadataka koje ćete konstantno izvoditi kroz komandnu liniju.

## Shell scripting

Do sada smo vidjeli kako da izvršimo komande u sell-u i kako da ih spojimo. Ipak, u mnogim slučajevima željećete da izvršite seriju komandi i iskoristite kontrolu niza izraza kao što su uslovni iskazi ili petlje.

Shell skripte su sledeće korak u kompleksnosti. Većina shell-ova imaju svoje skripting jezike sa varijablama, kontrolom toka i ličnom sintaksom. Ono što čini shell skripting različitim od ostalih skripting programskih jezika jeste da je optimizovan za izvršavanje zadataka koji se tiču shell-a. Takođe, kreiranje komandnih pajplajna, čuvanje rezultata u fajlovima, i čitanja kroz standardni input su primitive u shell skriptingu, koje je lakše koristiti u odnosu na skripting jezike koji su generalne prirode. Za ovu sekciju mi ćemo se fokusirati na bash skripting, budući da je on najčešći.

Da bi dodali varijble u bash-u, koristite sintaksu `foo=bar` i pristupite vrijednosti varijable sa `$foo`. Imajte u vidu da `foo = bar` neće raditi, s obzirom na to da je interpretirano kao pozivanje foo programa sa argumentima `=` i `bar`. Generalno u shell skriptingu razmak će izvršiti dijeljenje argumenata. Ovo ponašanje može biti zbunjujuće iz prve, pa uvijek imajte to u vidu.

Stringovi u bash-u mogu biti definisani sa `'` i `"` navodnicima, ali oni nisu jednaki. Stringovi označeni sa `'` su literalni stringovi i neće zamijeniti vrijednost varijable, dok `"` navodnici hoće.

```console
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```
Kao i sa većinom programskih jezika, bash podržava kontrolu toka sa tehnikama koje uključuju `if`, `case`, `while` and `for`. Slično, `bash` ima funkscije koje primaju argumente i dozvoljava vršenje operacija sa njima. Evo primjera funkcija koja kreira direktorijum i vrši cd u njega.

```console
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

Ovdje je `$1` prvi argument skripti/funkciji. Za razliku od drugih skripting jezika, bash koristi širok spektar specijalnih varijabli koje ukazuju na argumente, error kodove, i druge relevantne varijable. Ispod je lista nekih od njih. Obimnija lista se može naći [ovdje](https://www.tldp.org/LDP/abs/html/special-chars.html)

- `$0` = Naziv skritpte
- `$1` do `$9` - Argumenti skripte. `$1` je prvi argument i tako da dalje.
- `$@` - Svi argumenti
- `$#` - Broj argumenata
- `$?` - Vraća kod od prethodne komande
- `$$` - Indetifikacioni broj procesa (IBP) za trenutnu skriptu
- `!!` - Čitava poslednja komanda, uključujući argumente. Često se koristi da bi se izvršila komanda koja je prethodno podbacila zbog dozvola koje nedostaju; Možete brzo re-izvršiti komandu sa sudo koristeći `sudo !!`
- `$_` - Poslednji argument od poslednje komande. Ako ste u interaktivnom shell-u, možete takođe brzo dobiti ovu vrijednost kucanjem `Esc` i zatim `.`

Komande će često vratiti output koristeći `STDOUT`, greške kroz `STDERR`, i Return Code da bi izvijestile o greškama na način koji je više skript-friendly. Kod koji se vraća ili exit status je način na koji skript/komande moraju da komuniciraju o tome kako je izvršavanje proteklo. Vrijednost 0 obično znači da je sve proteklo dobro; Bilo šta što je drugačije od 0 znači da je nastupila greška.

Exit kodovi se mogu koristiti da bi se uslovno izvršavala komanda koristeći `&&` (i operator) i `||` (ili operator). Komande takođe mogu biti razdvojene u okviru iste linije koristće tačku zarez `;`. `Ispravni program` će uvijek imati 0 kao return kod, a komanda koja `nije uspjela` će uvijek imati 1 kao return kod. Hajde da vidimo neke primjere.

```console
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
Još jedan poznati obrazac je kada želite da dobijete output komande kao varijablu. Ovo može biti odrađeno zamjenom komandi. Kada god postvate `$( CMD )` izvršiće `CMD`, uzeti output komande i zamijeniti je na mjestu. Na primer, ukoliko uradite `for file in $(ls)`, shell će prvo pozvati `ls` i onda izvršiti iteraciju kroz te vrijednosti. Sličan način, koji je manje poznat jeste proces zamjene, `<( CMD )` će izvršiti `CMD` i postaviti output u trenutni fajl u zamijeniti `<()` sa nazivom toga fajla. Ovo je korisno kada komande očekuju vrijednosti da im budu proslijeđene od strane fajla umjesto od strane `STDIN`. Na primer, `diff <(ls foo) <(ls bar)` će pokazati razlike između fajlova u direktorijumu foo i bar.

Kako je ovdje prikazano jako puno informacija, hajde da vidimo primjer koji prikazuje primjenu ovih stvari. Izvršiće iteraciju kroz argumente koje smo obezbijedili, 
`grep` za string `foobar`, i dodaće ih fajlu kao komentar ukoliko nije pronađen.

```console
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
U poređenju smo testirali da li `$?` nije bilo jednako 0. Bash sprovodi mnoga poređenja ovakve vrste - možete pronaći detaljnu listu u man stranici za [test](https://www.man7.org/linux/man-pages/man1/test.1.html). Kada se izršava poređenje u bash-u, pokušajte da više koristite duple zagrade `[[ ]]` u odnosu na obične zagrade `[ ]`. Šanse da napravite greške su niže, iako nisu prenosive na `sh`. Detaljnije objašnjenje može biti pronađeno [ovdje](mywiki.wooledge.org/BashFAQ/031).

Kada pokrećemo skriptu, često ćete željete da pružite argumente koji su slični. Bash ima način da se ovo olakša, širenjem izraza provođenjem expanzije fajlnejma. Ove tehnike se česte nazivaju shell _globbing_.

- Wildcards - Kada god želite da izvedete neku vrstu wildcard podudaranja, možete koristiti `?` i `*` da bi se podudarili sa jednim ili više karaktera. Na primer, dati fajlovi `foo`, `foo1`, `foo2`, `foo10` i `bar`, i komanda `rm foo?` će izbrisati `foo1` i `foo2`, dok će `rm foo*` izbrisati sve osim `bar`.

-Vitičaste zagrade {} - Kada god imate zajednički substring u seriji komandi, možete koristiti vitičaste zagrade za bash da bi proširili ovo automatski. Ovo je veoma korisno kada pomjerate ili konvertujete fajlove.

```console
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

Pisanje bash bash skripti može biti izazovno i neintiutivno. Postoje alati kao što je [spellcheck](https://github.com/koalaman/shellcheck) koji će vam pomoći da pronađete greške u vašim sh/bash skriptama.

Imajte na umu da skripte ne moraju biti napisane u bash-u da bi bile pozvane iz terminala. Na primer, ovo je jednostavna Python skripta koja ispisuje argumente u obrnutom redosledu:
```console
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```
Kernel zna da izvrši ovu skriptu sa python interpreterom umjesto sa shell komandom zato što smo uključili [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix) linije na samom vrhu skripte. Dobra praksa je da pišete shebang liniju koristeći env komandu koja će se izvršiti bez obzira koja je komanda linija u sistemu, povećavajući prenosivost vaših skripti. Da bi riješio lokaciju, env će koristiti `PATH` evironment variable koji smo predstavili u prvoj lekciji. Za ovaj primjer shebang linija će izgledati ovako: `#!/usr/bin/env python.`

Neke razlike između shell funcija i scripti koje bi trebali da imate u vidu su:

- Funkcije moraju biti u istom jeziku kao i shell, dok skripte mogu biti napisane u bilo kojem jeziku. Ovo je razlog zašto je uključivanje shebang za skripte veoma važno.
- Funkcije se učitavaju jednom kada se pročita njihova definicija. Skripte se učitavaju svaki put kada se izvršavaju. Ovo čini funkcije malo bržim za učitavanje, ali kada ih promijenite moraćete da ponovo učitate njihove definicije.
- Funkcije se izvršavaju u trenutnom shell okruženju, dok se skripte izvršavaju u njihovom posebnom procesu. Dodatno, funkcije mogu promijeniti varijable okruženja, npr. promijeniti vaš trenutni direktorijum, dok skripte ne mogu. Skripte će biti zaobiđene od vrijednosti environment varijabli koje su bile eksportovane koristeći [export](https://www.man7.org/linux/man-pages/man1/export.1p.html).
- Kao i sa bilo kojim drugim programskim jezikom, funkcije su moćan konstrukt za postizanje modularnosti, ponovne upotrebe koda, i jasnoće shell kod-a. Obično će shell skripte uključiti njihovu sopstvenu definiciju funkcije.

## Shell Tools

### Pronalaženje načina za korišćenje komandi

U ovom trenutku, vjerovatno se pitate kako da pronađete za komande u sekcijama kao što su: `ls -l, mv -i i mkdir -p`. Još šire, imajući u vidu komandu, kako saznajete šta ona radi i koje su njene opcije? Uvijek možete da koristite google, ali pošto je UNIX stariji od StackOverflow-a, postoje već ugrađeni načini da bi došli do ovih informacija.

Kao što smo vidjeli u shell lekciji, prvi pristup bi bio da pozovete komadnu sa `-h` ili `--help` flagovima. Detaljniji pristup jeste da koristite man komandu. Skraćenica za uputstvo, [man](https://www.man7.org/linux/man-pages/man1/man.1.html) pruža mogućnost stranice sa uputstvom (zvanom manpage) za komandu koju ste izabrali. Na primer, `man rm` će ispisati ponašanje rm komande zajedno sa flagovima koje prima, uključujući `-i` flag koji smo prikazali ranije. U stvarim, sve linkove koje sam postavljao do sada za svaku komandu su onlajn verzije Linux manpages za komande. Čak i strane komande koje instalirate će imati manpage ukoliko ga je programer napisao i uključio u proces instalacije. Za interaktivne alate kao što su oni koji su bazirani na ncurses, može se pristupiti pomoći za programe koristeći `:help` komandi ili kucajući `?`.

Nekada manpages mogu pružiti previše detaljan opis komandi, i mogu da otežaju odluku koji flag/sintaksu će biti korišćen za standardnu upotrebu. [TLDR](https://tldr.sh/) strance su sjajno rešenje koje se fokusira na dati primjer tako da možete mnogo brže i lakše da izaberete opciju koju ćete koristiti. Lično, češće koristim stranice za [tar](https://tldr.ostera.io/tar) i [ffmpeg](https://tldr.ostera.io/ffmpeg) u odnosu na manpages.

## Traženje fajlova

Jedan od najčešćih zadataka koji se ponavljaju jeste traženje fajlova ili direktorijuma. Svi UNIX-like sistemi dolaze sa find, odličnim shell alatom kojim se pretražuju fajlovi. Find će rekurzivno tražiti fajlove koji ispunjavaju zadati kriterijum. Neki primjeri: 

One of the most common repetitive tasks that every programmer faces is finding files or directories. All UNIX-like systems come packaged with find, a great shell tool to find files. find will recursively search for files matching some criteria. Some examples: 

```console
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

```console
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec convert {} {}.jpg \
```
Uprkos sveprisutnosti pretrage, njegova sintaksa ponekad može biti nezgodna za zapamtiti. Na primer, da bi se pojednostavila pretraga fajlova koji se poklapaju sa nekim obrascem, morate da izvršite `find -name '*PATTERN*' (ili -iname` ukoliko želite da obrazac za poklapanje ne bude osjetljiv na velika slova). Možete početi da pravite aliase za ove scenarije, ali dio shell filozofije je dobro pretražiti alternative. Zapamtite, jedano od najboljih svojstava shella jeste da vi samo pozivate programe, tako da možete da pronađete (ili čak i sami da napišete) zamjenu za neke. Na primer, [fd](https://github.com/sharkdp/fd) je jednostavan, brz, i user-friendly alternativa za find. Nudi neke dobre defaultne kao što je kolorizovani output, defaultno regex poklapanje, i Unicode podršku. Takođe ima, prema mom mišljenju, intuitivniju sintaksu. Na primer, sintaksu da se pronađe pattern `PATTERN` je `fd PATTERN`.

Većina će se složiti da su `find` i `fd` dobri, ali se možda neki od vas pitaju o efektivnosti traženja fajlova svaki put umjesto kompajliranja neke vrste index-a ili baze podataka za brzu pretragu. [Locate](https://www.man7.org/linux/man-pages/man1/locate.1.html) služi tome. `Locate` koristi bazu podataka koja se ažurira koristeći [updatedb](https://www.man7.org/linux/man-pages/man1/updatedb.1.html). U većini sistema, `updatedb` je ažiriran na dnevnom nivou preko [cron](https://www.man7.org/linux/man-pages/man8/cron.8.html). Stoga je jedan kompromis između ovo dvoje brzina protiv svježine. Pored toga `find` i slični alati mogu takođe pronaći fajlove koristeći atribute, kao što je veličina fajla, vrijeme modifikacije, dozvole fajla, dok `locate` koristi samo naziv fajla. Još dublje poređenje može biti pronađeno [ovdje](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other).

