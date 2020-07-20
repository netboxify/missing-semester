# Upravljanje podacima

<a href="http://www.youtube.com/watch?feature=player_embedded&v=sz_dsktIjt4
" target="_blank"><img src="" 
alt="Lecture 4: Data Wrangling (2020)" width="240" height="180" border="10" /></a>

Da li ste ikada željeli da uzmete podatke u jednom formatu i pretvorite ih u drugi format? Naravno da jeste! To je, generalno gledano, tema ove lekcije. Tačnije, upravljanje podacima, bilo u tekstualnom ili binarnom formatu, sve dok ne dođete do onoga što želite.

Već smo vidjeli neke osnove upravljanja podacima u prethodnim lekcijama. Manje-više kada god koristite `|` operator, izvodite neku vrstu upravljanja podacima. Razmotrite komandu kao što je `journalctl | grep -i intel`. Pronalazi sve system log unose koji pominju Intel (nije osjetljivo na velika slova). Možda to ne posmatrate kao upravljanje podacima, ali kada idete iz jednog formata (vaš čitav system log) do formata koji je mnogo korisniji za vas (samo intel log unosi). Većina stvari koje se tiču upravljanja podacima se odnosi na to da znate koje alate imate na raspolaganju, i kako da ih kombinujete. 

Krenimo od početka. Da bi upravljali podacima, potrebne su nam dvije stvari: podaci kojima upravljamo, i nešto što ćemo sa njima da uradimo. Logs su obično dobar primjer, jer često želite da ispitate stvari koje su vezane za njih, a kompletno čitanje nije izvodljivo. Hajde da shvatimo ko pokušava da se uloguje u moj server posmatranjem mog server log-a: 

```shell
ssh myserver journalctl
```

To je previše stvari. Ograničimo to na ssh stvari:

```shell 
ssh myserver journalctl | grep sshd
```

Primjetite da koristimo pajp za striming __udaljene__ datoteke kroz `grep` na našem lokalnom računaru! `ssh` je magičan, govorićemo više o njemu u sledećoj lekciji koja se tiče okruženja komandne linije. Ipak, ovo je i dalje mnogo više stvari nego što smo željeli. I jako je teško za čitati. Bolje da odradimo:

```shell
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less
```
Zašto dodatno citiranje? Pa, naši logs mogu biti veoma veliki, suvišno je da sve to strimujemo na naš računar i onda da ih filtriramo. Umjesto toga, možemo odraditi filtriranje na udaljenom serveru, a zatim proslijediti podatke lokalno. `less` nam pruža "pager" koji nam omogućuje da skrolujemo gore i dolje kroz dugi output. Da bi mogli da uštedimo dodatni promet dok ispravljamo našu komandnu liniju, možemo umetnuti trenutni filtrirane zapise u datoteku, tako da ne moramo da pristupamo networku dok razvijamo: 

```shell
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log
```

Ovdje postoji još mnogo buke. Postoji još __mnogo__ načina da se toga riješite, ali hajde da pogledamo jedan od najmoćnijih alata koji su vam na raspolaganju: `sed`.

`sed` je "strim editor" koji je napravljen na osnovama starog `ed` editora. U njemu, vi u osnovi dajete kratke komande vezane za modifikaciju datoteke, umjesto da manipulišete sa njenim sadržajem direktno(iako i to možete da uradite). Postoji jako puno komandi, ali jedna od najčešćih je `s`: zamjena. Na primer, možemo napisati:

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

Ovo što smo upravo napisali je jednostavni __regularni izraz__; moćan konstrukt koji vam pruža mogućnost da uparujete tekst sa obrascima. Komanda `s` je napisana u formi: `s/REGEX/SUBSTITUTION/`, gdje je `REGEX` regularni izraz koji želite da pretražite, i `SUBSTITUTION` je tekst sa kojim želite da zamijenite tekst koji se poklapa. 

(Možda možete prepoznati ovu sintaksu iz "Pronađi i zamijeni" sekcije iz naših Vim [Zabilješke lekcije](https://missing.csail.mit.edu/2020/editors/#advanced-vim)! Zaista, Vim koristi sintaksu da pronađe i zamijeni koja je slična sa `sed` komandom zamjene. Učenje jednog alata će vam često pomoći u tome da budete mnogo bolji i sa ostalim alatima.)

## Regularni izrazi

Regularni izrazi su uobičajeni i dovoljno korisni da je pametno uložiti vrijeme da bi shvatili kako funkcionišu. Počnimo posmatrajući primjer odozgo: `/.*Disconnected from /`. Regularni izrazi su obično (ali ne i uvijek) okruženi sa `/`. Većina ASCII karaktera nosi samo svoje normalno značenje, ali neki karakteri imaju "specijalno" značenje. Šta tačno koji karakter radi može varirati između različitih implementacija regularnih izraza, što može biti izvor velike frustracije.
Uobičajeni obrasci su: 

- `.` znači "bilo koji pojedinačni karakter" osim novog reda
- `*` nula ili više prethodnih se poklapa
- `+` jedan ili više prethodnih se poklapa 
- `[abc]` bilo koji od karaktera `a`, `b`, `c`
- `(RX1|RX2)` nešto što se poklapa ili sa RX1 ili sa RX2
- `^` početak reda
- `$` kraj reda

`sed` regularni izrazi su donekle čudni, i zahtjevaće da stavite `\` prije većine navedenih da bi im se dalo specijalno značenje. Ili možete proslijediti `-E`.

Gledajući ponovo na `/.*Disconnected from /`, vidimo da se poklapa sa bilo kojim tekstom koji počinje sa bilo kojim brojem karaktera, praćenim literalnim stringom "Disconnected from". To je upravo ono što želimo. Ali budite oprezni, regularni izrazi umiju da budu nezgodni. Šta ukoliko je neko pokušao da se uloguje sa korisničkim imenom "Disconnected from"? Imali bi: 

```shell
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

Sa čim bi završili? Pa, `*` i `+` su, uobičajeno "pohlepni". Oni će se podudariti sa najviše teksta koliko mogu. Tako, da bi gore završili sa samo

```shell
46.97.239.16 port 55920 [preauth]
```

To može biti ono što nismo željeli. U nekim implementacijama regularnih izraza, možete samo dodati `*` ili `+` sa `?` da bi učinili da ne budu pohlepni, ali nažalost `sed` to ne podržava. Mogli bismo se okrenuti perl-ovom modu komandne linije, koji __podržava__ takav konstrukt: 

```shell
perl -pe 's/.*?Disconnected from //'
```

Držaćemo se `sed`-a do kraja ovoga, jer je to mnogo češći alat za ovakvu vrstu poslova. `sed` takođe može raditi druge pogodne stvari, kao što je ispisivanje redova praćene zadatim poklapanjem, izvršiti više zamjena u toku jednog poziva, pretražiti stvari itd. Ali nećemo previše tih stvari pokriti ovdje. `sed` je, u stvari, posebna tema za sebe, ali često postoje i bolji alati.

U redu, takođe imamo sufix kojeg bi da se riješimo. Kako bi mogli to da uradimo? Malo je nezgodno izvršiti poklapanje teksta koji prati korisničko ime, posebno ukoliko ime ima razmake i slično. Ono što moramo da uradimo jeste da se podudari __čitava__ linija: 

```shell
| sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```
Hajde da pogledamo šta se dešava sa [regex debugger](https://regex101.com/r/qqbZqh/2). U redu, početak je isti kao i ranije. Onda, vrišimo podudaranje sa bilo kojom varijantom "korisnika" (postoje dva prefixa u logs). Onda vršimo podudaranje bilo kojeg stringa gdje je korisničko ime. Onda vršimo podudaranje bilo koje pojedinačne riječi `([^ ]+`; i dijelova karaktera koji nisu prazni). Zatim riječ "port" koja je praćena dijelovima cifara. Zatim mogući suffix `[preauth]`, i zatim kraj reda.

Primjećujete da koristeći ovu vrstu tehnike, korisničko ime "Disconnected from" nas više ne zbunjuje. Da li možete da vidite zašto?

Postoji jedan problem sa ovim, a to je da čitav log postaje prazan. Mi želimo da __zadržimo__ korisničko ime nakon svega. Za ovo, možemo koristiti "hvatanje grupa". Bilo koji tekst koji se poklapa sa regex-om okruženim zagradom se čuva u grupi koja je označena brojem. Ove su dostupne u zamjeni (a u nekim engin-ima, čak i sam obrazac) kao `\1`, `\2`, `\3`, itd. Tako da: 

```shell
| sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

Kao što vjerovatno možete da zamislite, možete dobiti __veoma__ složeni regularni izraz. Na primer, evo jednog članka o tome kako bi mogli da izvršite podudaranje sa [e-mail adresom](https://www.regular-expressions.info/email.html). Nije [lako](https://emailregex.com/). I postoji [dosta rasprave](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982). I ljudi su [pisali testove](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php). I [matrice testova](https://mathiasbynens.be/demo/url-regex). Možete čak i napisati regex da bi provjerili da li je zadati broj [prost broj](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/).

Regularni izrazi su notorno teški za pročitati, ali ih je jako pogodno znati.

## Nazad na upravljanje podacima

U redu, sada imamo

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

`sed` može uraditi razne interesantne stvari, kao ubacivanje teksta (sa `i` komandom), eksplicitno pisanje linija (sa `p` komandom), selektovanje linija po indeksu, i dosta drugih stvari. Provjerite `men sed`!

Da se vratimo na stvar. Ono što imamo sada je lista svih korisničkih imena koji su pokušali da se uloguju. Ali ovo i nije baš korisno. Hajde da pogledamo neke česte primjere: 

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort` će sortirati njegov input.  `uniq -c` će oboriti uzastopne linije koje su iste u jednu liniju, koji ima prefix u vidu broja slučajeva. Vjerovatno da to želimo da sortiramo i zadržimo najčešća korisnička imena:

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n` će sortirati numeričkim (umjesto leksikografskim) redom. `-k1,1` znači "sortiraj sa samo prvom kolonom koja je odvojena bijelim prostorom". `,n` dio kaže "Sortiraj do `n` polja, gdje je default kraj linije." U ovom __specifičnom__ primjeru, sortiranje od strane čitave linije neće značiti, ali mi smo ovdje da bi naučili. 

Ukoliko bi željeli one koje su najređe, možemo koristiti `head` umjesto `tail`.
Takođe postoji `sort -r`, koji sortira u obrnutom redosledu. 

U redu, to je bilo prilično dobro, ali šta ukoliko bi željeli da izvedemo samo korisnička imena sa listom koja je odvojena zarezom umjesto novim redom, možda za config datoteku? 

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

Počnimo sa `paste`: Dopušta vam da kombinujete linije (`-s`) sa datim razdjelnikom sa jednim karakterom (`-d`; `,` u ovom slučaju). Ali šta je posao ovog `awk`-a?

## awk - drugi editor

`awk` je programski jezik koji je veoma dobar za obradu toka teksta. Postoji __dosta toga__ što bi se moglo reći za `awk` ukoliko bi željeli da ga valjano naučite, ali kao sa dosta drugih stvari ovdje, mi ćemo preći samo osnove. 

Prvo, šta `{print $2}` radi? Pa, `awk` program uzima formu opcionog obrasca plus blok koji govori šta da se uradi ukoliko se obrazac poklopi sa datim redom. Uobičajen obrazac (koji smo koristili gore) se podudara sa svim redovima. Unutar bloka, `$0` je podešen za čitav sadržaj reda, i `$1` kroz `$n` su podešene za `n` polje tog reda, koji je odvojen `awk` separatorom polja (obično bijeli prostor, mijenja se sa `-F`). U ovom slučaju, mi govorimo da, za svaku liniju, se ispisuje sadržaj drugog polja, a dešava se to da je to korisničko ime!

Hajde da vidimo da li možemo nešto da uradimo još bolje. Prvo, primjećujete da sada imam obrazac (stvari koje idu prije `{...}`). Obrazac kaže da bi prvo polje reda trebalo da bude jednako 1 (to je brojanje od `uniq -c`), i da bi drugo polje trebalo da se podudari sa datim regularnim izrazom. A blok samo kaže da se ispiše korisničko ime. Zatim brojimo broj redova u outputu sa `wc -l`.

Ipak, `awk` je programski jezik, sjećate se? 

```shell
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN` je obrazac koji se podudara sa početkom inputa (i `END` se podudara sa krajem). Sada, blok po redu samo dodati brojač iz prvog polja (iako će to biti 1 u ovom slučaju), i onda ćemo ispisati to na kraju. U stvari, mogli bismo se otarasiti `grep` i `sed` u potpunosti, iz razloga što `awk` [može sve to da odradi](https://backreference.org/2010/02/10/idiomatic-awk/), ali ćemo ipak izbor ostaviti čitaocu. 

## Analiziranje podataka 

Možete koristiti matematiku direktno u vašem shell-u koristeći `bc`, kalkulator koji može da čita iz STDIN! Na primer, dodavanje brojeva na svakoj liniji zajedno, spajajući ih zajedno, razgraničavajući ih sa `+`:

```shell
 | paste -sd+ | bc -l
```

Ili napraviti složenije izraze: 

```shell
echo "2*($(data | paste -sd+))" | bc -l
```

Možete dobiti statistiku na više načina. [st](https://github.com/nferraz/st) je veoma uredan, ali ako već imate [R](https://www.r-project.org/): 

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```

R je još jedan (čudan) programski jezik koji je odličan za analizu podataka i [plotting](https://ggplot2.tidyverse.org/). Nećemo puno zalaziti u detalje, ali je dovoljno reći da `rezime` štampa zbirnu statisktiku za vektor, a kreirali smo vektor koji sadrži ulazni tok brojeva, tako da R daje statistiku koju smo željeli!

Ukoliko samo želite jednostavan plotting, `gnuplot` je vaš prijatelj:

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

## Upravljanje podacima da bi se napravili argumenti

Ponekad želite da upravljate podacima da bi pronašli stvari koje ćete da instalirate ili uklonite bazirajući se na nekoj dužoj listi. Upravljanje podacima o kojem smo pričali do sada + `xargs` mogu biti moćna kombinacija. 

Na primer, kao što ste vidjeli u lekciji, mogu koristiti sledeću komandu da deinstaliram stari build Rust-a iz mog sistema izvodeći stare nazive build-a koristeći alate za upravljanje podacima i prosleđujući ih kroz `xargs` deinstalatoru:

```shell
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

## Upravljanje binarnim podacima

Do sada, najviše smo pričali o upravljanju tekstualnim podacima, ali pajpovi su takođe korisni i za binarne podatke. Na primer, možemo koristiti ffmpeg da snimimo sliku iz naše kamere, konvertujemo je u grayscale, kompresujemo je, pošaljemo je na udaljenu mašinu preko SSH-a, tamo je dekompresujemo, napravimo kopiju, i onda je prikažemo. 

```shell
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

## Vježbe 

1. Pogledajte ovaj [kratki interaktivni regex tutorijal](https://regexone.com/)
2. Pronađite broj riječi (u `/usr/share/dict/words`) koje sadrže bar tri `a` i nemaju `s` na kraju. Koje su tri kombinacije najčešća poslednja dva slova te riječi? `sed` i `y` komanda, ili `tr` program vam može pomoći sa neosjetljivošću velikih slova. Koliko je tih kombinacija sa dva slova tu? I za izazov: Koje kombinacije se ne pojavljuju?
3. Da bi se uradila zamjena u mjestu veoma je izazovno uraditi nešto kao `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`. Ipak je ovo loša ideja, zašto? Da li je ovo posebno vezano za `sed`? Koristite `man sed` da bi saznali kako ovo da odradite. 
4. Pronađite vaš prosjek, medijanu, i najduže vrijeme podizanja sistema za poslednjih deset puta. Koristite `journalctl` na Linuxu i `log show` na macOS, i potražite vremenske oznake log-a blizu početka i kraja svakog pokretanja. Na Linuxu, oni mogu izgledati ovako: 

```shell
Logs begin at ...
```

i

```shell
systemd[577]: Startup finished in ...
```

Na macOS, [potražite](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):

```shell
=== system boot:
```

i 

```shell
Previous shutdown cause: 5
```

5. Potražite boot poruke koje nisu podijeljenje između vaša 3 poslednja podizanja sistema (pogledajte `journalctl` `-b` flag). Podijelite ovaj zadatak na više koraka. Prvo nađite način da dobijete log iz poslednja 3 podizanja sistema. Može postojati odgovarajući flag na alatu koji koristite da bi izvukli log podizanja sistema, ili možete koristiti `sed '0,/STRING/d'` da uklonite sve linije prije one koja se poklapa sa `STRING`. Dalje, uklonite sve dijelove reda koji uvijek varira (kao vremenski žig). Zatim, de-duplicirajte linije inputa i zadržite broj svake od njih (`uniq` je vaš prijatelj). I konačno, elminišite bilo koji red čiji je brojač 3 (budući da je bio podijeljen u svim podizanjima sistema).
6. Pronađite skup podataka online kao što je [ovaj](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm), [ili ovaj](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1) ili možda [ovaj](https://www.springboard.com/blog/free-public-data-sets-data-science-project/). Dohvatite ih koristeći `curl` i izvadite samo dvije kolone numeričkih podataka. Ukoliko povlačite HTML podatke, [pup](https://github.com/EricChiang/pup) može biti od koristi. Za JSON podatke, pokušajte sa [jq](https://stedolan.github.io/jq/). Pronađite minimum i maksimum jedne kolone u jednoj komandi, i zbir razlika između dvije kolone u drugoj.
