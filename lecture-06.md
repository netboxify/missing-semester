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

### Model podataka, kao pseudokod

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

Konačno, možemo definisati šta je (okvirno) Git skladište: to su `objekti` podataka i `reference`.

Na disku, Git skladišti objekte i reference: to je sve vezano za model podataka Git-a. Sve `git` komande se odnose na neku manipulaciju commit DAG dodavanjem objekata i dodavanjem/ažuriranjem referenci.

Kada god upisujete bilo koju komandu, razmislite kakvu manipulaciju pravi u graph strukturi podataka. Suprotno tome, ukoliko želite da napravite posebnu vrstu izmjene DAG commit-a, npr. "Odbaci promjene koje nisu commit-ovane i podesi 'master' pokazivač da ukazuje na `5d83f9e`", vjerovatno postoji komanda da se to uradi(npr. u ovom slučaju, `git checkout master; git reset --hard 5d83f9e`).

## Scensko područje

Ovo je još jedan koncept koji je ortogonalan modelu podataka, ali je dio interfejsa za kreiranje commit-a.

Jedan način na koji bi mogli da zamislite implementiranje snimaka kao što je gore navedeno jeste da imate "kreiraj snimak" komandu koja kreira novi snimak na osnovu __trenutnog stanja__ radnog direktorijuma. Neki alati kontrole verzije rade na ovaj način, ali ne i Git. Mi želimo čiste snimke, i možda ne bi uvijek bilo idealno da pravimo snimke u odnosu na trenutno stanje. Na primer, zamislite scenario gdje ste implementirali dvije odvojene funkcije, i želite da kreirate dva različita commit-a, gdje prvi predstavlja prvu funkciju, a drugi predstavlja drugu funkciju. Ili zamislite scenario gdje imate debugging print izjave koje su dodate po čitavom kodu, zajedno sa bugfix-om; Vi želite da commitujete bugfix i da otpišete sve print izjave.

Git se uklapa u takve scenarije dozvoljavajući vam da odredite koje bi izmjene trebalo uključiti u sledećem snimku kroz mehanizam koji se naziva "staging area".

## Git komandna linija interfejsa

Da bi izbjegli dupliranje informacija, nećemo objašnjavati komande ispod u detalje. Pogledajte veoma preporučeni [Pro Git](https://git-scm.com/book/en/v2) za više informacija, ili pogledajte lekciju iz videa.

### Osnove

- `git help <command>`: git pomoć za git komandu
- `git init`: kreira novi git repo, sa podacima skladištenim u `.git` direktorijumu.
- `git status`: govori vam šta se dešava
- `git add <filename>`: dodaje fajlove u scensko područje
- `git commit`: kreira novi commit
    - Pišite [dobre git commit poruke](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
    - Još više razloga da pišete [dobre git commit poruke](https://chris.beams.io/posts/git-commit/)
- `git log`: pokazuje spljošteni log istorije
- `git log --all --graph --decorate`: Vizualizuje istoriju kao DAG
- `git diff <filename>`: pokazuje promjene koje ste napravili koje su relativne u odnosu na scensko područje
- `git diff <revision> <filename>`: Prikazuje promjene u fajlovima između snimaka
- `git checkout <revision>`: Ažurira HEAD i trenutnu granu

### Grananje i spajanje

- `git branch`: pokazuje grane
- `git branch <name> `: kreira granu
- `git checkout -b <name>`: kreira granu i prebacuje se na nju
    - isto kao i ` git branch <name>; git checkout <name>`
- `git merge <revision> `: spaja se u trenutnu granu
- `git mergetool `: koristi fensi tool za pomoć pri rešavanju konflikata u spajanju
- `git rebase `: postavlja set zakrpa na novu bazu
 
 ### Udaljeni
 
- `git remote`: izlistava remote 
- `git remote add <name> <url>`: dodaje remote 
- `git push <remote> <local branch>:<remote branch>`: šalje objekte na remote, i ažurira remote referencu
- `git branch --set-upstream-to=<remote>/<remote branch>`: podešava vezu između lokalne i remote grane
- `git fetch`: dohvata objekte/reference iz remote-a 
- `git pull`: isto što i `git fetch; git merge` 
- `git clone`: preuzima skladište iz remote-a 

### Undo

- `git commit --amend`: uređuje commit sadržaj/poruku
- `git reset HEAD <file>`: uklanja fajl iz scenskog područja
- `git checkout -- <file>`: odbacuje izmjene

## Napredni Git

- `git config`: Git je [veoma podesiv](https://git-scm.com/docs/git-config) 
- `git clone --depth=1`: plitak clone, bez čitave istorije verzije 
- `git add -p`: interaktivno scensko područje 
- `git rebase -i`: interaktivan rebasing 
- `git blame`: pokazuje ko je poslednji editovao koju liniju 
- `git stash`: trenutno uklanja izmjene u radnom direktorijumu 
- `git bisect`:  binarno pretražuje istoriju (npr. za regresiju)
- `.gitignore`: [označava](https://git-scm.com/docs/gitignore) namjerno fajlove koji se ne prate da budu ignorisani

## Ostalo

- **GUIs**: Postoji mnogo [GUI klijenata](https://git-scm.com/downloads/guis) za Git. Umjesto njih, mi koristimo komandnu liniju interfejsa.
- **Shell integracija**: Veoma je pogodno imati git status kao dio vašeg shell prompta([zsh](https://github.com/olivierverdier/zsh-git-prompt), [bash](https://github.com/magicmonty/bash-git-prompt)). Obično uključuju framework kao što je [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh).
- **Editor integracija**: slično ovome iznad, pogodne integracije sa mnogo funkcija. [fugitive.vim](https://github.com/tpope/vim-fugitive) je standardan za Vim.
- **Radni tokovi**: Naučili smo vas model podataka, plus neke osnovne komande: nismo vam rekli koju praksu da pratite kada radite na velikim projektima (i postoji [mnogo](https://nvie.com/posts/a-successful-git-branching-model/) [različitih](https://www.endoflineblog.com/gitflow-considered-harmful) [pristupa](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow))
- **GitHub**: Git nije GitHub. GitHub ima specifičan način doprinošenja koda drugim projektima, koji se naziva [pull request](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).
- **Drugi Git provajderi**: GitHub nije specijalan: Postoji mnogo Git repository hostova, kao što su [GitLab](https://about.gitlab.com/) i [BitBucket](https://bitbucket.org/).

## Resursi

- [Pro Git](https://git-scm.com/book/en/v2) se **veoma preporučuje za čitanje**. Prolazak kroz poglavlja 1-5 će vas naučiti najveći dio toga što vam je potrebno da bi tečno koristili Git, sada kada razumijete model podataka. Dalja poglavlja imaju neke zanimljive, napredne materijale. 
- [Oh Shit, Git!?!](https://ohshitgit.com/) je kratak vodič o tome kako da se oporavite od nekih najčešćih Git grešaka.
- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/) je kratko objašnjenje Git modela podataka, sa manje pseudokoda i više fenski dijagrama nego što ih ima u ovim zabilješkama lekcija.
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/) je detaljno objašnjenje Git implementacije detalja iznad samo modela podataka, za radoznale.
- [How to explain git in simple words](https://smusamashah.github.io/explain-git-in-simple-words/)
- [Learn Git Branching](https://learngitbranching.js.org/) je igrica koja se zasniva na browser-u i koja vas uči Git.

## Vježbe

1. Ukoliko nemate prethodnog iskustva sa Git-om, ili pokušajte da pročitate prvih nekoliko poglavlja [Pro Git](https://git-scm.com/book/en/v2) ili prođite tutorijal kao što je [Learn Git Branching](https://learngitbranching.js.org/). Kako prolazite kroz to, povezujte Git komande sa modelom podataka.
2. Klonirajte [repository for the class website](https://github.com/missing-semester/missing-semester).
  - Pretražite istorju verzije vizualizujući je kao grafik.
  - Ko je poslednja osoba koja je podesila `README.md`? (Nagovještaj: koristite `git log` sa argumentom)
  - Koja je commit poruka koja je povezana sa poslednjim izmjenama `collections:` linija `_config.yml`? (Nagovještaj: koristite `git blame` i `git show`)
3. Jedna česta greška u toku učenja Git-a jeste commit velikih fajlova kojima ne bi trebao da upravlja Git ili dodavanje osjetljivih informacija. Pokušajte da dodate fajl u repository, napravite neke commit-ove i zatim obrišite taj fajl iz istorije (možda ćete željeti da pogledate [ovo](https://docs.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository)).
4. Klonirajte neki repository iz GitHub-a, i podesite neki od postojećih fajlova. Šta se dešava kada uradite `git stash`? Šta vidite kada pokrenete `git log --all --oneline`? Pokrenite `git stash pop` da bi poništili on o što ste uradili sa `git stash`. U kom scenariju bi ovo moglo biti korisno?
5. Kao sa mnogim alatima komandne linije, Git pruža configuration fajl (ili dotfile) koji se naziva ` ~/.gitconfig`. Kreirajte pseudonim u ` ~/.gitconfig` tako da kada pokrenete `git graph`, dobijate output `git log --all --graph --decorate --oneline`.
6. Možete definisati globalne obrasce za ignorisanje u `~/.gitignore_global` nakon pokretanja `git config --global core.excludesfile ~/.gitignore_global`. Uradite ovo, i podesite vaš globalni gitignore fajl da ignoriše OS-specifične ili editor-specifične privremene fajlove, kao što je `.DS_Store`.
7. Forkujte [repository for the class website](https://github.com/missing-semester/missing-semester), pronađite grešku u kucanju ili napravite neko drugo poboljšanje, i podnesite pull request na GitHub.
