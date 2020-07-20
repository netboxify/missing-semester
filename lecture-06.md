# Kontrola verzije (Git)

<a href="http://www.youtube.com/watch?feature=player_embedded&v=2sjqTHE0zok
" target="_blank"><img src="" 
alt="Lecture 6: Version Control (git) (2020)" width="240" height="180" border="10" /></a>

Sistemi za kontrolu verzije (VCSs) su alati koji se koriste za praćenje promjena u izvornom kodu (Ili drugih kolekcija fajlova i foldera). Kao što i samo ime kaže, ovi alati pomažu pri održavanju istorije promjena; štaviše, oni olakšavaju saradnju. VCSs prate promjene u folderu i njegovom sadržaju u nizu snimaka, gdje svaki snimak enkapsulira čitavo stanje fajlova/foldera unutar direktorijuma najvišeg nivoa. VSCs takođe održavaju meta podatke kao npr. ko je kreirao svaki snimak, poruke povezane sa svakim snimkom i tako dalje.

Zašto je kontrola verzije korisna? Čak i kada radite sami, dopušta vam da pogledate stari snimak projekta, čuva log u kome piše zbog čega su određene izmjene napravljene, rad na paralelnim granama developmenta, i još mnogo toga. Kada radite sa drugima, to je neprocjenjiv alat koji vam pruža mogućnost da vidite koje su izmjene drugi ljudi napravili, kao i rešavanja konfilkata u istovremenom developmentu.

Moderni VSC vam takođe pružaju mogućnost da lako (obično automatski) odgovorite na pitanja kao što su:

- Ko je napisao ovaj modul? 
- Kada je posebna linija ili poseban fajl uređen? Od strane koga? Zašto je editovano? 
- Tokom poslednjih 1000 revizija, kada/zašto je određeni test jedinice prestao sa radom? 

Dok postoje i drugi VSCs, **Git** je suštinksi standard za kontrolu verzije. Ovaj [XKCD comic](https://xkcd.com/1597/) odražava reputaciju Git-a.

![img1][img1]

[img1]: https://imgs.xkcd.com/comics/git.png

Budući da je Git-ov interfejs nepropusna apstrakcija, učenje Git-a odozgo nadolje (krećući sa njegovim interfejsom / komandnom linijom interfejsa) može dovesti do mnogo zabune. Moguće je zapamtiti dosta komandi i zamišljajući ih kao neku vrstu magije, i prateći pristup stripa odozgo kada god nešto krene po zlu. 

Dok Git priznato ima ružan interfejs, dizajn i ideje koje leže ispod njega su prelijepe. Dok se ružni interfejs može __zapamtiti__, predivan dizajn se može __razumjeti__. Iz ovoh razloga, mi dajemo objašnjenje Git-a od dna ka vrhu, startujući sa njegovim modelom podataka i kasnije pokrivajući interfejs komandne linije. Jednom kada je model podataka shvaćen, komande mogu biti bolje shvaćene, u smislu toga kako one manipulišu modelom podataka.

## Model podataka Git-a

Postoji mnogo ad-hoc pristupa koje bi mogli da primijenite na kontrolu verzije. Git ima dobro osmišljen model koja omogućava sve dobre funkcije verzije kontrole, kao što je održavanje istorije, podržavanje grana, i omogućavanje saradnje.

### Snimci

Git modelira istoriju kolekcije fajlova i foldera unutar nekog najvišeg nivoa direktorijuma u vidu serije snimaka. U Git terminologiji, fajl se naziva "blob" i on je samo gomila bajtova. Direktorijum se naziva "drvo", i on mapira imena blob-u ili drvećima (tako da direktorijumu mogu sadržati druge direktorijume). Snimak je drvo najvećeg nivoa koji se prati. Na primer, mogli bismo imati drvo kao što slijedi:

```git
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

Drvo najvišeg nivoa sadrži dva elementa, drvo "foo" (koje unutar sebe sadrži jedan element, blob "bar.txt"), i blob "baz.txt".

### Modeliranje istorije: Povezani snimci

Kako bi sistem kontrole verzije trebao da povezuje snimke? Jedan jednostavan model bi bio da imamo linearnu istoriju. Istorija bi bila lista snimaka u vremenskom redosledu. Iz mnogih razloga, Git ne koristi jednostavan model kao što je ovaj.

U Git-u, istorija je usmjereni aciklički grafik (DAG) snimaka. Ovo može zvučati kao fensi riječ iz matematike, ali nemojte biti zastrašeni. Ovo samo znači da se svaki snimak u Git-u odnosi na skup "roditelja", snimaka koji su mu prethodili. U pitanju je skup roditelja prije nego jedan roditelj (kao što bi bio slučaj sa linearnom istorijom) jer snimak može poticati od više roditelja, na primer zbog kombinovanja (spajanja) dvije pararelne grane developmenta.

Git naziva ove snimke "commit"s. Vizualizaciju commit istorije može izgledati ovako: 

```git
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

U ASCII artu iznad, `о` se odnosi na pojedinačan commit (snimak). Strelica ukazuje na roditelja svakog commita (to je relacija "dolazi prije" a ne "dolazi poslije). Nakon trećeg commit-a, istorija se grana na dvije različite grane. Ovo se može odnositi, na primer, na dvije različite funkcije koje se razvijaju paralelno, nezavisne jedna od druge. U budućnosti, ove grane mogu biti spojene da bi se kreirao novi snimak koji inkorporira obje funkcije, stvarajući novu istoriju koja izgleda ovako, sa novo-kreiranim commit-om spajanja koji je označen zvijezdicom:

```git
o <-- o <-- o <-- o <---- *o*
            ^            /
             \          v
              --- o <-- o
```

Commits u Git-u su nepromenljivi. Ovo ne znači da greške ne mogu biti ispravljene, ipak; izmjene u commit istoriji zapravo kreiraju potpuno novi commit, i reference (vidi dolje) se ažuriraju tako da ukazuju na nove. 

Model podataka, kao pseudokod

Može biti poučno da vidite model podataka Git-a napisan dolje u pseudokodu:

```git
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

To je čist, jednostavan model istorije.

### Objekti i adresiranje sadržaja 

"Objekat" je blob, drvo, ili commit:

```git
type object = blob | tree | commit
```

U Git bazi podataka, svi objekti su sadržajno adresirani na osnovu njihovo [SHA-1 hash](https://en.wikipedia.org/wiki/SHA-1).

```git
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blobs, drveća, i commiti ujedinjeni na ovaj način: svi su oni objekt. Kada oni referenciraju druge objekte, oni ih u stvari ne __sadrže__ na njihovoj on-disk reprezentaciji, ali imaju referencu ka njima na osnovu hash-a.

Na primer, drva za primjer strukture direktorijuma [iznad](https://missing.csail.mit.edu/2020/version-control/#snapshots) (vizualizovan koristeći ` git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`), izgleda ovako:

```git
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

Samo drvo sadrži pokazivače na njegov sadržaj, `baz.txt` (blob) i `foo` (drvo). Ukoliko pogledamo sadržaje koji su adresirani hash-om koji odgovara baz.txt sa `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`, dobijamo sledeće: 

```git
git is wonderful
```

### Reference 

Sada, svi snimci mogu biti identifikovani na osnovu njihovog SHA-1 hash-a. To nije pogodno, jer ljudi nisu dobri u pamćenju stringova od 40 heksadecimalnih karaktera.

Git-ovo rešenje ovoga problema su ljudski razumljiva imena za SHA-1 hashove, koja se nazivaju "reference". Reference su pokazivači na commit-e. Za razliku od objekata, koji su nepromenljivi, reference su promenljive (mogu biti ažurirani da ukazuju na novi commit). Na primer, `master` referenca obično ukazuje na poslednji commit u glavnoj grani developmenta. 

```git
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

Sa ovim, Git može koristiti ljudski razumljiva imena kao što su "master" da ukaže na određeni snimak u istoriji, umjesto dugog heksadecimalnog stringa.

Jedan detalj jeste da često želimo da znamo "gdje se trenutno nalazimo" u istoriji, tako da kada napravimo novi snimak, da znamo na šta se tačno odnosi (kako smo postavili __roditelj__ polje u commit-u).U Git-u, to "gdje se trenutno nalazimo" je specijalna referenca koja se naziva "HEAD".

### Skladište

