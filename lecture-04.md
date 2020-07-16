# Upravljanje podacima

<a href="http://www.youtube.com/watch?feature=player_embedded&v=sz_dsktIjt4
" target="_blank"><img src="" 
alt="Lecture 4: Data Wrangling (2020)" width="240" height="180" border="10" /></a>

Da li ste ikada željeli da uzmete podatke u jednom formatu i pretvorite ih u drugi format? Naravno da jeste! To je, generalno gledano, tema ove lekcije. Tačnije, upravljanje podacima, bilo u tekstualnom ili binarnom formatu, sve dok ne dođete do onoga što želite.

Već smo vidjeli neke osnove upravljanja podacima u prethodnim lekcijama. Manje-više kada god koristite `|` operator, izvodite neku vrstu upravljanja podacima. Razmotrite komandu kao što je `journalctl | grep -i intel`. Pronalazi sve system log unose koji pominju Intel (nije osjetljivo na velika slova). Možda to ne posmatrate kao upravljanje podacima, ali kada idete iz jednog formata (vaš čitav system log) do formata koji je mnogo korisniji za vas (samo intel log unosi). Većina stvari koje se tiču upravljanja podacima se odnosi na to da znate koje alate imate na raspolaganju, i kako da ih kombinujete. 

Krenimo od početka. Da bi upravljali podacima, potrebne su nam dvije stvari: podaci kojima upravljamo, i nešto što ćemo sa njima da uradimo. Logs su obično dobar primjer, jer često želite da ispitate stvari koje su vezane za njih, a kompletno čitanje nije izvodljivo. Hajde da shvatimo ko pokušava da se uloguje u moj server posmatranjem mog server log-a: 

```console
ssh myserver journalctl
```

To je previše stvari. Ograničimo to na ssh stvari:

```console 
ssh myserver journalctl | grep sshd
```

Primjetite da koristimo pajp za striming __udaljene__ datoteke kroy `grep` na našem lokalnom računaru! `ssh` je magičan, govorićemo više o njemu u sledećoj lekciji koja se tiče koja se tiče okruženja komandne linije. Ipak, ovo je i dalje mnogo više stvari nego što smo željeli. I jako je teško za čitati. Bolje da odradimo:

```console
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less
```
Zašto dodatno citiranje? Pa, naši logs mogu biti veoma veliki, suvišno je da sve to strimujemo na naš računar i onda da ih filtriramo. Umjesto toga, možemo odraditi filtriranje na udaljenom serveru, a zatim proslijediti podatke lokalno. `less` nam pruža "pager" koji nam omogućuje da skrolujemo gore i dolje kroz dugi output. Da bi mogli da uštedimo dodatni promet dok ispravljamo našu komandnu liniju, možemo umetnuti trenutni filtrirane zapise u datoteku, tako da ne moramo da pristupamo networku dok razvijamo: 

```console
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log
```

Ovdje postoji još mnogo buke. Postoji još __mnogo__ načina da se toga riješite, ali hajde da pogledamo jedan od najmoćnijih alata koji su vam na raspolagnaju: `sed`.

`sed` je "strim editor" koji je napravljen na osnovama starog `ed` editora. U njemu, vi u osnovi dajete kratke komande vezane za modifikaciju datoteke, umjesto da manipulišete sa njenim sadržajem direktno(iako i to možete da uradite). Postoji jako puno komandi, ali jedna od najčešćih je `s`: zamjena. Na primer, možemo napisati:

```console
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

Ovo što smo upravo napisali je jednostavni __regularni izraz__; moćan konstrukt koji vam pruža mogućnost da uparujete tekst sa obrascima. Komanda `s` je napisana u formi: `s/REGEX/SUBSTITUTION/`, gdje je `REGEX` regularni izraz koji želite da pretražite, i `SUBSTITUTION` je tekst sa kojim želite da zamijenite tekst koji se poklapa. 

(Možda možete prepoznati ovu sintaksu iz "Pronađi i zamijeni" sekcije iz naših Vim [Zabilješke lekcije](https://missing.csail.mit.edu/2020/editors/#advanced-vim)! Zaista, Vim koristi sintaksu da pronađe i zamijeni koja je slična sa `sed` komandom zamjene. Učenje jednog alata će vam često pomoći u tome da budete mnogo bolji i sta ostalim alatima.)

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

Gledajući ponovo na `/.*Disconnected from /`, vidimo da se poklapa sa bilo kojim tekstom koji počinje sa bilo kojim brojem karaktera, praćenim literalnim stringom "Disconnected from". To je upravo ono što želimo. Ali budite oprezni, regularni izrazi umiju da budu nezgodni. Šta ukoloko je neko pokušao da se uloguje sa korisničkim imenom "Disconnected from"? Imali bi: 

```console
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

Sa čim bi završili? Pa, `*` i `+` su, uobičajeno "pohlepni". Oni će se podudariti sa najviše teksta koliko mogu. Tako, da bi gore završili sa samo

```console
46.97.239.16 port 55920 [preauth]
```

To može biti ono što nismo željeli. U nekim implementacijama regularnih izraza, možete samo dodati `*` ili `+` sa `?` da bi učinili da ne budu pohlepni, ali nažalost `sed` to ne podržava. Mogli bismo se okrenuti perl-ovom modu komandne linije, koji __podržava__ takav konstrukt: 

```console
perl -pe 's/.*?Disconnected from //'
```

Držaćemo se `sed`-a do kraja ovoga, jer to mnogo češći alat za ovakvu vrstu poslova. `sed` takođe može raditi druge pogodne stvari, kao što je ispisivanje redova praćene zadatim poklapanjem, izvršiti više zamjena u toku jednog poziva, pretražiti stvari itd. Ali nećemo previše tih stvari pokriti ovdje. `sed` je, u stvari, posebna tema za sebe, ali često postoje i bolji alati.

U redu, takođe imamo sufix kojeg bi da se riješimo. Kako bi mogli to da uradimo? Malo je nezgodno izvršiti poklapanje teksta koji prati korisničko ime, posebno ukoliko ime ima razmake i slično. Ono što moramo da uradimo jeste da se podudari __čitava__ linija: 

```console
| sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```
