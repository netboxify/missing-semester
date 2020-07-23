# Metaprogramiranje

<a href="http://www.youtube.com/watch?feature=player_embedded&v=_Ms1Z4xfqv4
" target="_blank"><img src="" 
alt="Lecture 8: Metaprogramming (2020)" width="240" height="180" border="10" /></a>

Šta mislimo kada kažemo "metaprogramiranje"? Pa, to je bio najbolji kolektivni termin koji smo mogli smisliti za skup stvari koje su više vezane za __procese__  nego za pisanje koda ili efikasniji rad. U ovoj lekciji ćemo pogledati sisteme za izgradnju i testiranje vašeg koda, i upravljenje zavisnostima. Ovo se može činiti da ima ograničenu važnost u vašim redovnim danima kao studentima, ali u trenutku kada imate interakciju sa većom bazom koda kroz stručnu praksu ili jednom kada uđete u "pravi svijet", na ovo ćete svuda naići. Trebali bi da imamo u vidu da "metaprogramiranje" takođe može značiti [programi koji upravljaju programima](https://en.wikipedia.org/wiki/Metaprogramming), ipak ovo nije baš definicija koju koristimo za svrhu ove lekcije. 

## Sistem građenja

Ako pišete rad u LaTeXu, koje komande morate da pokrenete da biste proizveli vaš rad? Šta je sa onima koje ste koristili za vaše refrentne vrijednosti, nacrtate ih, a onda ubacite nacrt u rad? Ili da kompajlirate kod koji vam je omogućen u vašoj lekciji a zatim izvršite testiranje?

Za većinu projekata bilo da sadrže kod ili ne, postoji "proces gradnje". Neki dio operacija koje morate da obavite da biste prešli od vašeg inputa do vašeg outputa. Često, taj proces može imati više koraka i mnogo grana. Pokrenite ovo da bi generisali nacrt, ovo da bi generisali rezultate, i nešto drugo da bi dobili konačan rad. Kao sa mnogim stvarima koje smo vidjeli u ovim lekcijama, niste prvi koji nailazite na ovu iritantnost, ali srećom postoji mnogo alata koji će vam pomoći.

Oni se obično nazivaju "sistemi građenja", i ima ih veoma __mnogo__. Koji ćete koristiti zavisi od zadatka, izbora jezika i veličine projekta. U njihovoj srži, oni su veoma slični. Definišete broj __zavisnosti__, broj __targeta__, i __pravila__ za odlazak od jednog do drugog. Kažete sistemu građenja da želite određenu metu, i njegov posao je da pronađe sve prelazne zavisnosti te mete, i zatim primijeni ta pravila za stvaranje posrednih meta sve dok se ne stvori konačna meta. Idealno, sistemi građenja rade ovo bez nepotrebnog izvršavanja pravila za mete čije se zavisnosti nisu promijenile i čiji je rezultat dostupan iz prethodnog građenja.

`make` je jedan od najčešćih sistema za građenje, i vidjećete da je instaliran na gotovo svakom UNIX-based računaru. Ima svoje mane, ali radi dosta dobro za jednostavne-srednje-komplikovane projekte. Kada pokrenete `make`, on konsultuje fajl koji se zove `Makefile` u trenutnom direktorijumu. Sve mete, njihove zavisnosti, i pravila su definisana u tom fajlu. Hajde da pogledamo primjer jednog:

```python
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

Svaka direktiva u ovoj datoteci predstavlja pravilo za stvaranje lijeve strane koristeći desnu stranu. Ili, možda drugačije, stvari koje su nazvane na desnoj strani su zavisnosti, a na lijevoj strani su mete. Razdvojeni blok je niz programa koji proizvode metu iz tih zavisnosti. U `make`, prva direktiva takođe definiše defaultni cilj. Ukoliko pokrenete `make` bez argumenata, ovo je meta koja će se izgraditi. Alternativno, možete pokrenuti nešto kao `make plot-data.png`, i umjesto toga će izgraditi metu.

`%` pravilo je "obrazac", i podudariće se sa istim stringom i sa lijeve i sa desne strane. Na primer, ako se zahtijeva meta `plot-foo.png`, `make` će potražiti zavisnosti `foo.dat` i `plot.py`. Sada hajde da pogledamo šta se dešava ukoliko pokrenemo `make` sa praznim izvornim direktorijumom.

```shell
$ make
make: *** No rule to make target 'paper.tex', needed by 'paper.pdf'.  Stop.
```
`make` nam kao pomoć govori da kako bi mogli graditi `paper.pdf`, njemu je potreban `paper.tex`, i on nema pravilo koje može da slijedi da bi napravio taj fajl. Hajde da probamo da ga napravimo!

```shell
$ touch paper.tex
$ make
make: *** No rule to make target 'plot-data.png', needed by 'paper.pdf'.  Stop.
```

Hmm, interesantno, __postoji__ pravilo da bi se izgradio `plot-data.png`, ali to je pravilo obrasca. Pošto izvorni fajlovi ne postoje (`foo.dat`), `make` jednostavno izjavljuje da ne može napraviti taj fajl. Hajde da pokušamo da kreiramo sve fajlove:

```python
$ cat paper.tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\includegraphics[scale=0.65]{plot-data.png}
\end{document}
$ cat plot.py
#!/usr/bin/env python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', type=argparse.FileType('r'))
parser.add_argument('-o')
args = parser.parse_args()

data = np.loadtxt(args.i)
plt.plot(data[:, 0], data[:, 1])
plt.savefig(args.o)
$ cat data.dat
1 1
2 2
3 3
4 4
5 8
```

Sada, šta se dešava ukoliko pokrenemo `make`?

```shell
$ make
./plot.py -i data.dat -o plot-data.png
pdflatex paper.tex
... lots of output ...
```

Pogledajte, napravio je PDF za nas! Šta ukoliko pokrenemo `make` ponovo?

```shell
$ make
make: 'paper.pdf' is up to date.
```

Nije uradio ništa! Zašto? Pa, zato što nije bilo potrebe. Provjerio da li su sve mete iz prošlog građenja ažurirane uzimajući u obzir njihove liste zavisnosti. Ovo možemo testirati podešavanjem `paper.tex` i onda ponovnim pokretanjem `make`: 

```shell
$ vim paper.tex
$ make
pdflatex paper.tex
...
```

Imajte u vidu da `make` nije ponovo pokrenuo `plot.py` zato što to nije bilo potrebno; 
nije se ništa od `plot-data.png` zavisnosti promijenilo!

## Upravljanje zavisnostima

Na višem macro nivou, vaši softverski projekti će vjerovatno imati zavisnosti koje su i sami projekti. Možete zavisiti od instaliranih programa (kao što je `python`), paketa sistema (kao što je `openssl`), ili biblioteke u okviru vašeg programskog jezika (kao što je `matplotlib`). Ovih dana, većina zavisnosti će biti dostupna kroz `repository` koji hostuje veliki broj takvih zavisnosti na jednom mjestu, i pruža pogodan mehanizam za njihovu instalaciju. Neki primjeri uključuju Ubuntu paket skladište za Ubuntu sistemske pakete, kojima pristupate pomoću `apt` alata, RubyGems za Ruby biblioteke, PyPi za Python biblioteke, ili Arch korisničko skladište za Arch Linux pakete doprinešene od strane korisnika.

Kako tačan mehanizam za interakciju sa ovim skladištima veoma varira od jednog do drugog i od alata do alata, nećemo ulaziti u detalje nekog specifičnog skladišta u ovoj lekciji. Pokrićemo neke od najčešćih terminologija koju svi oni koriste. Prva od njih su __verzije__. Većina projekata od kojih drugi projekti zavise prave __verziju broja__ sa svakim izdanjem. Obično nešto kao 8.1.3 ili 64.1.20192004. Oni su često, ali ne i uvijek, numerisani. Brojevi verzije se koriste u mnoge svrhe, a jedna od najvažnijih jeste da se osigura da softver nastavi sa radom. Zamislite, na primer, da sam izbacio novu verziju moje biblioteke gdje sam preimenovao ime određene funkcije. Ukoliko neko pokuša da izgradi neki softver koji zavisi od moje biblioteke nakon što ja izbacim ažuriranje, izgradnja bi mogla da propadne jer se poziva funkcija koja više ne postoji! Verzije pokušavaju da riješe ovaj problem zato što se za projekat kaže da zavisi od određene verzije, iako postoji više verzija. Na taj način, čak iako se biblioteka promijeni, softver koji zavisi od nje nastavlja da se gradi koristeći staru verziju biblioteke.

Ovo takođe nije idealno! Šta ukoliko postoji bezbjednosno ažuriranje koje __ne__ mijenja javni interfejs moje biblioteke (njen "API"), i svaki program koji je zavistan od stare verzije bi trebalo da odmah počne da ga koristi? Ovdje različite grupe brojeva u verziji dolaze do izražaja. Tačno značenje svake varira između projekata, ali jedan relativno čest standard jeste [semantičko verziranje](https://semver.org/). Sa semantičkim verziranjem, svaki broj verzije je u formi: major.minor.patch. Pravila su:

- Ako novo izdanje ne mijenja API, povećajte patch verziju. 
- Ukoliko __dodajete__ vašem API u povratno-kompatibilnom načinu, povećajte minor verziju.
- Ukoliko mijenjate API u nepovratno-kompatibilnom načinu, povećajte major verziju.

Ovo već omogućuje neke velike prednosti. Sada, ukoliko moj projekat zavisi od tvog projekta, __trebalo bi__ da je bezbjedno koristiti poslednje izdanje sa istom major verzijom kao što je bila ona koju sam koristio tokom gradnje, dokle god je minor verzija makar ono što je bila tada. Drugim riječima, ukoliko zavisim od vaše biblioteke na verziji `1.3.7`, __trebalo bi__ da je u redu da gradim sa `1.3.8`, `1.6.1`, ili čak `1.3.0`. Verzija `2.2.4` vjerovatno ne bi bila u redu za korišćenje, jer se major verzija povećala. Možemo vidjeti primjer semantičkog verziranja u Pythonovoj verziji brojeva. Mnogi od vas su vjerovatno svjesni da Python 2 i Python 3 kod nisu dobri za miješanje, a to je razlog što se ažurirala __major__ verzija. Slično, kod napisan za Python 3.5 se možda može dobro izvršavati i na Python 3.7, ali vjerovatno ne i na 3.4.

Kada se radi sa upravljenjem sistemom zavisnosti, možete naići na pojam __lock fajlova__. Lock fajlovi su jednostavno fajlovi koji izlistaju tačnu verziju od koje __trenutno__ zavisite za svaku zavisnost. Obično, morate eksplicitno da pokrenete ažuriranje programa da bi ažurirali na novije verzije vaših zavisnosti. Postoji mnogo razloga za ovim, kao što je izbjegavanje nepotrebne rekompilacije, imate reproduktivno građenje, ili se ne ažurirate automatski na najnoviju verziju (koja može biti pokvarena). Ekstremna verzija ovakve vrste lockinga zavisnosti je, gdje kopirate sav kod svojih zavisnosti u svoj sopstveni projekat. To vam daje totalnu kontrolu za bilo koje promjene, ali to takođe znači da sami morati eksplicitno da ubacite bilo koje ažuriranje za održavanje tokom vremena.

## Sistemi kontinuirane integracije

Kako radite na sve većim i većim projektima, vidjećete da obično postoji još dodatnih zadataka koje morate da uradite kada god napravite izmjene. Možda ćete trebati da ažurirate novu verziju dokumentacije, negdje da otpremite kompajliranu verziju, objavite kod na pypi, izvršite testiranje, i razne druge stvari. Možda želite svaki put kada neko pošalje pull zahtjev na GitHub, da provjerite stilove njihovog koda i izvršite neke referentne vrijednosti? Kada ovakve stvari moraju da nastanu, vrijeme je da se pogledaju sistemi kontinuirane integracije.

Kontinuirana integracija, ili CI, je izraz za "stvari koje se pokreću kada god se vaš kod promijeni", i postoji mnogo kompanija koje omogućuju različite vrste CI-a, obično besplatno za open-source projekte. Neke od najvećih su Travis CI, Azure Pipelines, i GitHub Actions. Ugrubo, oni svi rade na isti način: dodajete fajl u skladište koji opisuje šta bi trebalo da se uradi kada se razne stvari dese tom skladištu. Najčešći pristpup je pravilo kao što je "napravi testove, kada neko doda kod". Kada se događaj trigeruje, CI provajder pokrene virtualnu mašinu (ili više njih), pokreće komande po vašem "receptu", i obično negdje zabilježi rezultate. Možete ga podesiti da vas obavijesti ukoliko testiranje ne prođe, ili da se mali bedž pojavi na vašem skladištu dokle god test prolazi.

Kao primjer CI sistema, sajt naših lekcija je podešen koristeći GitHub Pages. Pages je CI akcija koja pokreće Jekyll blog softver na svaki push na `master`-u i pravi gradnju sajta dostupnim na određenom GitHub domenu. Ovo nam veoma olakšava ažuriranje sajta. Samo napravimo naše izmjene lokalno, commitujemo ih koristeći git, i onda uradimo push. CI vodi računa o svemu ostalom.

### Kratko o testiranju

Većina velikih softvera dolazi sa "testnim paketom". Možda su vam već poznati opšti koncepti testiranja, ali smo razmišljali da brzo pomenemo neke pristupe testiranju i terminologiju testiranja na koju ćete naići:

- Testni paket: kolektivni izraz za sve testove
- Testiranje jedinice: "mikro-test" testira specifičnu funkciju u izolaciji
- Test integracije: "makro-test" koji pokreće veći dio sistema da bi provjerio različite funkcije ili komponente kako rade __zajedno__.
- Test regresije: test koji implementira poseban obrazac koji je __ranije__ prouzrokovao bag, da bi se osiguralo da se bag ne pojavi ponovo.
- Moking: mijenjanje funkcije, modula ili tipa sa lažnom implementacijom da bi se izbjeglo testiranje nepovezane funkcionalnosti. Na primer, mogli biste da "mokirate network" ili "mokirate disk".

## Vježbe

1. Većina mejkfajlova pružaju metu koja se naziva `clean`. Ovdje nije namjera da se proizvede fajl sa nazivom `clean`, već da se očisti bilo koji fajl koji se može re-izgraditi sa make. Mislite o ovome kao "undo" za sve korake gradnje. Implementirajte `clean` metu za `paper.pdf` `Makefile` iznad. Moraćete da napravite metu [phony](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html). Možda ćete vidjeti da je [git-ls-files](https://git-scm.com/docs/git-ls-files) komanda korisna. Veliki broj veoma čestih make meta su na listi [ovdje](https://www.gnu.org/software/make/manual/html_node/Standard-Targets.html#Standard-Targets).
2. Pogledajte razne načine da biste odredili zahtjeve verzije za zavisnosti u [Rust's build system](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html). Većina skladišta paketa podržava sličnu sintaksu. Za svaki (caret, tilde, wildcard, poređenje i dupliranje), pokušajte da nađete slučaj upotrebe u kojem ta određena vrsta zahtjeva ima smisla.
3. Sam Git se može ponašati kao određena vrsta CI sistema. U `.git/hooks` unutar bilo kojeg git skladišta, naći ćete (trenutno neaktivne) fajlove koji se pokreću kao skripte kada se određena akcija desi. Napišite [pre-commit](https://git-scm.com/docs/githooks#_pre_commit) hook koji pokreće `make paper.pdf` i odbija commit ukoliko `make` komanda ne uspije. Ovo će spriječiti svaki commit da za paper napravi verziju koja se ne može izgraditi.
4. Podesite jednostavnu stranicu koja se sama objavljuje koristeći [GitHub Pages](https://pages.github.com/). Dodajte [GitHub Action](https://github.com/features/actions) skladištu da bi pokrenuli `shellcheck` na bilo koji shell fajl u tom skladištu (ovo je [jedan način da to uradite](https://github.com/marketplace/actions/shellcheck)). Provjerite da li radi!
5. [Napravite vaš](https://docs.github.com/en/actions/creating-actions) GitHub action da bi pokrenuli [proselint](http://proselint.com/) ili [write-good](https://github.com/btford/write-good) na sve `.md` fajlove u skladištu. Omogućite ga u vašem skladištu, i provjerite da li radi sa pull zahtjevom koji ima grešku pri pisanju u njemu.




