# Metaprogramiranje

<a href="http://www.youtube.com/watch?feature=player_embedded&v=_Ms1Z4xfqv4
" target="_blank"><img src="" 
alt="Lecture 8: Metaprogramming (2020)" width="240" height="180" border="10" /></a>

Šta mislimo kada kažemo "metaprogramiranje"? Pa, to je bio najbolje kolektivni termin koji smo mogli smisliti za skup stvari koje su više vezane za __procese__  nego za pisanje koda ili efikasniji rad. U ovoj lekciji ćemo pogledati sisteme za izgradnju i testiranje vašeg koda, i upravljenje zavisnostima. Ovo se može činiti da ima ograničenu važnost u vašim redovnim danima kao studentima, ali u trenutku kada imate interakciju sa većom bazom koda kroz stručnu praksu ili jednom kada uđete u "pravi svijet", na ovo ćete svuda naići. Trebali bi da imamo u vidu da "metaprogramiranje" takođe može značiti [programi koji upravljaju programima](https://en.wikipedia.org/wiki/Metaprogramming), ipak ovo nije baš definicija koju koristimo za svrhu ove lekcije. 

## Sistem građenja

Ako pišete rad u LaTeXu, koje komande morate da pokrenete da biste proizveli vaš rad? Šta je sa onima koje ste koristili za vaše refrentne vrijednosti, nacrtate ih, a onda ubacite nacrt u rad? Ili da kompajlirate kod koji vam je omogućen u vašoj lekciji a zatim izvršite testiranje?

Za većinu projekata bilo da sadrže kod ili ne, postoji "proces gradnje". Neki dio operacija koje morate da obavite da biste prešli od vašeg inputa do vašeg outputa. Često, taj proces može imati više koraka i mnogo grana. Pokrenito ovo da bi generisali nacrt, ovo da bi generisali rezultate, i nešto drugo da bi dobili konačan rad. Kao sa mnogim stvarima koje smo vidjeli u ovim lekcijama, niste prvi koji nailazite na ovu iritantnost, ali srećom postoji mnogo alata koji će vam pomoći.

Oni se obično nazivaju "sistemi građenja", ima ih veoma __mnogo__. Koji ćete koristiti zavisi od zadatka, izbora jezika i veličine projekta. U njihovoj srži, oni su veoma slični. Definišete broj __zavisnosti__, broj __targeta__, i __pravila__ za odlazak od jednog do drugog. Kažete sistemu građenja da želite određenu metu, i njegov posao je da pronađe sve prelazne zavisnosti te mete, i zatim primijeniti ta pravila za stvaranje posrednih meta sve dok se ne stvori konačna meta. Idealno, sistemi građenja rade ovo bez nepotrebnog izvršavanja pravila za mete čije se zavisnosti nisu promijenile i čiji je rezultat dostupan iz prethodnog građenja.

`make` je jedan od najčešćih sistema za građenje, i vidjećete da je instaliran na gotovo svakom UNIX-based računaru. Ima svoje mane, ali radi dosta dobro za jednostavne-srednje-komplikovane projekte. Kada pokrenete `make`, on konsultuje fajl koji se zove `Makefile` u trenutnom direktorijumu. Sve mete, njihove zavisnosti, i pravila su definisana u tom fajlu. Hajde da pogledam primjer jednog:

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

