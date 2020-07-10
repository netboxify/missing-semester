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

Kada pokrećemo skriptu, često ćete željete da pružite argumente koji su slični. Bash ima način da se ovo olakša, širenjem izraza

When launching scripts, you will often want to provide arguments that are similar. Bash has ways of making this easier, expanding expressions by carrying out filename expansion. These techniques are often referred to as shell globbing.
