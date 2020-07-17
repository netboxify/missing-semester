# Okruženje komandne linije

<a href="http://www.youtube.com/watch?feature=player_embedded&v=e8BO_dYxk5c
" target="_blank"><img src="" 
alt="Lecture 5: Command-line Environment (2020)" width="240" height="180" border="10" /></a>

U ovoj lekciji ćemo proći kroz nekoliko načina pomoću kojih možete da poboljšate vaš rad u korišćenju shell-a. Sa shell-om radimo već neko vrijeme, ali smo se najviše fokusirali na izvršavanje različitih komandi. Sada ćemo vidjeti kako da pokrenemo nekoliko procesa u isto vrijeme dok ih pratimo, kako da zaustavimo ili pauziramo specifični proces i kako da podesimo proces da radi u pozadini.

Takođe ćemo naučiti različite načine da pobljšate vaš shell i druge alate, definisanjem pseudonima i konfigurisanjem istih pomoću dotfiles. I jedno i drugo će vam uštedjeti vrijeme, npr. korišćenjem istih konfiguracija za sve vaše mašine bez potrebe da pišete duge komande. Vidjećemo i kako da radimo sa udaljenim mašinama koristeći SSH.

## Kontrola posla

U nekim slučajevima ćete morati da prekinete posao u toku izvršavanja, npr. ukoliko je komandi potrebno previše vremena da bi se izvršila (kao npr. `nalaženje` veoma velike strukture direktorijuma za pretraživanje). Većinu vremena, možete odraditi `Ctrl-C` i komanda će se zaustaviti. Ali, kako ovo zapravo radi i zašto ponekad ne uspijeva da prekne proces? 

### Ubijanje procesa 

Vaš shell koristi UNIX komunikacioni mehanizam koji se zove __signal__ za prosleđivanje informacija procesu. Kada proces primi signal, on zaustavlja svoje izvršavanje, bavi se signalom i potencijalno mijenja tok izvršenja na osnovu informacija koje je signal isporučio. Iz ovih razloga, signali su ono što __zaustavlja softver__.

U našem slučaju, kada kucamo `Ctrl-C` ovo traži shell-u da isporuči `SIGINT` signal procesu. 

Evo minimalističkog primjera Python programa koji snimi `SIGINT` signal i ignoriše ga, ne zaustavljajući se. Umjesto toga, da bi zaustavili ovaj program možemo koristiti `SIGQUIT` signal, kucanjem `Ctrl-\`.

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

Evo šta se dešava ukoliko unesemo `SIGINT` dva puta u program, praćen sa `SIGQUIT`. Imajte u vidu da `^` označava kako se `Ctrl` prikazuje u terminalu prilikom unosa.

```python
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

Dok su `SIGINT` i `SIGQUIT` obično povezani sa zahtjevima koji se odnose na terminal, više generički signal koji traži od procesa da se prekinu je `SIGTERM` signal. Da bi poslali ovaj signal možemo koristiti [kill](https://www.man7.org/linux/man-pages/man1/kill.1.html) komandu, sa sintaksom `kill -TERM <PID>`.

### Pauziranje i stavljanje procesa u pozadinu

Signali mogu raditi i druge stvari mimo zaustavljanja procesa. Na primer, `SIGSTOP` pauzira proces. U terminalu, kucanje `Ctrl-Z` će proslijediti shell-u da pošalje `SIGTSTP` signal, skraćeno za Terminal Stop (npr. verzija terminala za `SIGSTOP`).

Zatim možemo nastaviti pauzirani posao koji se nalazi u prvom planu ili u pozadini koristeći, zavisno od toga, [fg](https://www.man7.org/linux/man-pages/man1/fg.1p.html) ili [bg](https://man7.org/linux/man-pages/man1/bg.1p.html).

[Jobs](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) komanda izlistava nezavršene poslove koji su povezani sa trenutnom sesijom terminala. Možete referencirati te poslove koristeći njihov pid (možete koristiti [pgrep](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) da to saznate). Još više intuitivnije, možete referencirati proces koristeći simbol procenta i njegov broj zadatka (prikazan kao `jobs`). Da bi referencirali poslednji posao koji se odvija u pozadini, možete koristiti specijalni parametar `$!`. 

Dodatna stvar koju je potrebno znati jeste da `&` suffix u komandi će pokrenuti komandu u pozadini, vraćajući vam prompt, iako će i dalje koristiti shell STDOUT što može biti iritantno (koristite shell redirekcije u tom slučaju).

Da bi prebacili u pozadinu program koji je u toku možete odraditi `Ctrl-Z` praćen sa `bg`. Imajte u vidu da su procesi u pozadini i dalje djeca procesa vašeg terminala i da će biti ugašeni ukoliko zatvorite terminal (ovo će poslati još jedan signal, `SIGHUP`). Da bi spriječili da se to desi možete izvršiti program koristeći [nohup](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (omotač koji ignoriše `SIGHUP`), ili koristi `disown` ukoliko je proces već započet. Alternativno, možete koristiti terminal multiplexer kao što ćemo vidjeti u sledećoj sekciji. 

Ispod je jednostavna sesija koja prikazuje neke od tih koncepata. 

```shell
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs
```

Specijalni signal je `SIGKILL` budući da ne može biti snimljen od strane procesa i da će uvijek biti odmah prekinut. Ipak, može imati loše efekte sa strane kao što je ostavljanje samih children procesa. 

Možete naučiti više o ovim i drugim signalima [ovdje](https://en.wikipedia.org/wiki/Signal_(IPC)) ili kucanjem [man signal](https://www.man7.org/linux/man-pages/man7/signal.7.html) ili `kill -t`.

## Multiplexeri Terminala

Kada koristite interfejs komandne linije obično ćete željeti da pokrenete više stvari odjednom. Na primer, možda ćete željeti da pokrenete vaš editor i vaš program jedno pored drugog. Iako ovo može biti ostvareno otvaranjem novog prozora terminala, korišćenje multiplexera terminala je svestranije rešenje.

Multiplexeri terminala kao što je [tmux](https://www.man7.org/linux/man-pages/man1/tmux.1.html) vam pružaju mogućnost multiplexa terminala prozora koristeći panes i tabove tako da možete imati interakciju sa više shell sesija. Još više, multiplexeri terminala vam dopuštaju da odvojite trenutnu sesiju terminala i da se prikačite na nju u nekom kasnijem trenutku. Ovo može mnogo popraviti vaše iskustvo kada radite sa udaljenim mašinama jer izbjegava upotrebu `nohup` i sličnih trikova.

Najpopularniji multiplexer terminala ovih dana je [tmux](https://www.man7.org/linux/man-pages/man1/tmux.1.html). `tmux` je veoma podesiv i korišćenjem povezanih prečica tastera možete kreirati više tabova i panes-a i brzo se kretati kroz njih. 

`tmux` očekuje od vas da znate prečice za tastere, i sve one imaju formu `<C-b> x` što znači (1) pritisnite `Ctrl+b`, (2) otpustite `Ctrl+b`, i zatim (3) pritisnite `x`. `tmux` ima sledeću hijerarhiju objekata:

- **Sesije** - Sesija je nezavisni radni prostor sa jednim ili više prozora
    - `tmux` započinje novu sesiju 
    - `tmux new -s NAME` je započinje sa tim nazivom
    - `tmux ls` izlistava trenutnu sesiju
    - U okviru `tmux` kucanje `<C-b> d` odvaja trenutnu sesiju
    - `tmux a` pridodaje poslednju sesiju. Možete da koristite `-t` flag da označite koju
- Windows - Ekvivalent tabovima u editoru ili pretraživaču - vizuelno dovaja dio iste sesije
    - `<C-b> c` Kreira novi prozor. Da ga zatvorite možete zaustaviti shell koristeći `<C-d>`
    - `<C-b> N` Ide do __N__ th prozora. Imajte u vidu da su numerisani
    - `<C-b> p` Ide do prethodnog prozora
    - `<C-b> n` Ide do sledećeg prozora
    - `<C-b> ,` Mijenja ime trenutnog prozora
    - `<C-b> w` Izlistava trenutni prozor
- **Panes** - Kao i vim splits, panes vam dopuštaju da imate više shell-ova na istom displeju.
    - `<C-b> "` Dijeli trenutni pane horizontalno
    - `<C-b> %` Dijeli trenutni pane vertikalno
    - `<C-b> <direction>` pomjera pane u zadati __pravac__. 
    Pravac ovdje označava tastera sa strelicama.
    - `<C-b> z` Uključuje zoom za trenutni pane
    - `<C-b> [` Započinje pomicanje unazad. Zatim možete pritisnuti `<space>`da započnete selekciju i `<enter>` da kopirate tu selekciju.
    - `<C-b> <space>` Kruži kroz pane uređenje.
    
Za dalje čitanje, [ovdje](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) je kratki tutorijal na `tmux` a [ovdje](http://linuxcommand.org/lc3_adv_termmux.php) je detaljnije objašnjenje koje pokriva originalnu `screen` komandu. Možda ćete željete da se bolje upoznate sa [screen](https://www.man7.org/linux/man-pages/man1/screen.1.html), budući da dolazi instaliran na većini UNIX sistema.

## Pseudonimi

Može biti zamorno kucanje dugih komandi koje uključuju mnoge flagove ili opširne opcije. Iz ovog razloga, većina shell-ova podržava __pseudonime__. Shell pseudonim je kratka forma neke komande koju će vaš shell automatski zamijeniti za vas. Na primer, psuedonim u bash-u ima sledeću strukturu: 

```shell
alias alias_name="command_to_alias arg1 arg2"
```

Imajte u vidu da nema razmaka okolo znaka jednako `=`, jer je [alias](https://www.man7.org/linux/man-pages/man1/alias.1p.html) shell komanda koja prima jedan argument. 

Pseudonimi imaju mnoge pogodne funkcije: 

```shell
# Make shorthands for common flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gs="git status"
alias gc="git commit"
alias v="vim"

# Save you from mistyping
alias sl=ls

# Overwrite existing commands for better defaults
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# Alias can be composed
alias la="ls -A"
alias lla="la -l"

# To ignore an alias run it prepended with \
\ls
# Or disable an alias altogether with unalias
unalias la

# To get an alias definition just call it with alias
alias ll
# Will print ll='ls -lh'
```

Imajte u vidu da pseudonimi nisu trajni u shell sesiji po default-u. Da bi neki pseudonim bio trajan morate ih uključiti u shell startup datoteci, kao što je `.bashrc` ili `.zshrc` koje ćemo predstaviti u sledećoj sekciji.

## Dotfiles

Mnogi programi se podešavaju koristeći datoteke sa čistim tekstom poznate kao __dotfiles__ (jer naziv datoteke počinje sa `.`, npr. `~/.vimrc`, tako da su oni po defaultu skriveni u listingu direktorijuma `ls`).

Shell-ovi su jedan primjer programa koji se podešavaju preko takvih fajlova. Prilikom pokretanja, vaš shell će pročitati mnoge datoteke da bi se učitala njegova podešavanja. Zavisno od shell-a, bilo da pokrećete login i/ili interakciju, čitav proces može biti veoma kompleksan. [Ovo](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html) je odličan resurs na ovu temu. 

Za `bash`, uređivanje vašeg `.bashrc` ili `.bash_profile` će raditi na većini sistema. Ovdje možete uključiti komande koje želite da se pokrenu prilikom pokretanja, kao što su pseudonimi koje smo opisali ili modifikacija vašeg `PATH` okruženja varijabli. U stvari, mnogi programi će vas pitati da uključite red kao što je `export PATH="$PATH:/path/to/program/bin"` u vašoj shell konfiguracionoj datoteci, tako da njegovi binarni fajlovi mogu biti pronađeni.

Neki drugi primjeri alata koji mogu biti podešeni kroz dotfiles su: 

- `bash - ~/.bashrc, ~/.bash_profile`
- `git - ~/.gitconfig`
- `vim` - `~/.vimrc` and the `~/.vim folder`
- `ssh - ~/.ssh/config`
- `tmux - ~/.tmux.conf`

Kako da organizujete vaše dotfiles? Oni bi trebali da budu u njihovom posebnom folderu, ispod kontrole verzije, i **symlinked** na mjesto koristeći script. Ovo ima sledeće benefite:

- **Laka instalacija:** ukoliko se ulogujete na novu mašinu, potvrđivanje vaših podešavanja će trajati samo oko minuta.
- **Prenosivost:** Vaši alati će raditi na isti način svuda.
- **Sinhronizacija:** Možete da ažurirate vaše dotfiles bilo gdje i da svi oni budu sinhronizovani.
- **Praćenje promjena:** Vjerovatno ćete kroz vašu čitavu programersku karijeru održavati vaše dotfiles, i jako je dobro imati kontrolu istorije za projekte koji će duže trajati.

Šta bi trebali da stavite u dotfiles? Možete učiti o podešavanju vaših alata čitanjem online dokumentacije ili [man pages](https://en.wikipedia.org/wiki/Man_page). Drugi dobar način jeste pretraživanje interneta za blog postove o specifičnim programima, gdje će vam autori iznijeti njihvoe preference podešavanja. Johttps://github.com/jonhoo/configsš jedan način da učite o podešavanjima jeste pregledanjem dotfiles drugih ljudi: možete pronaći gomilu [dotfiles repositories](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) na Githubu - pogledajte najpopularnije [ovdje](https://github.com/mathiasbynens/dotfiles) (Savjetujemo vam da ne kopirate na slijepo podešavanja). [Ovdje](https://dotfiles.github.io/) je još jedan dobar resurs na ovu temu. 

Svi instruktori ovih lekcija imaju njihove dotfiles koji su svima dostupni na Github-u: [Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/jjgo/dotfiles).

### Prenosivost

Čest problem sa dotfiles jeste što podešavanja možda neće raditi u radu sa nekoliko mašina, npr. ukoliko imaju različite operativne sisteme ili shell-ove. Ponekad takođe želite da se podešavanja primjenjuju samo na mašini na kojoj u tom trenutku radite.

Postoje česti trikovi da vam to olakšaju. Ukoliko to datoteka sa podešavanjima podržava, koristite ekvivalent if izjava da bi na mašini primjenili specifična podešavanja. Na primer, vaš shell može imati nešto kao što je:

```shell
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# Check before using shell-specific features
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# You can also make it machine-specific
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

Ukoliko fajl sa podešavanjima to podržava, koristite includes. Na primer, `~/.gitconfig` može imati podešavanje:

```shell
[include]
    path = ~/.gitconfig_local
```

Onda na svakoj mašini, `~/.gitconfig_local` može sadržati specifična podešavanja za mašinu.

Ova ideja je takođe korisna ako želite da različiti programi dijele ista podešavanja. Na primer, ukoliko želite da i `bash` i `zsh` dijele iste pseudonime možete ih napisati u `.aliases` i imati sledeći blok u oba: 

```shell
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

## Udaljene mašine

Kod programera je postalo često da koriste udaljene servere u njihovom svakodnevnom životu. Ukoliko morate da koristite udaljene servere da bi razvili backend software ili vam je potreban server sa visokim računarskim sposobnostima, na kraju ćete koristiti Secure Shell (SSH). Kao i za većina alata koje smo prešli, SSH je veoma podesiv i vrijedan je učenja.

Da izvršite komandu `ssh` na serveru uradite sledeće:

```shell
ssh foo@bar.mit.edu
```

Ovdje pokušavamo da ssh kao korisnik `foo` na serveru `bar.mit.edu`. Server može biti naznačen sa URL-om (kao što je `bar.mit.edu`) ili IP (nešto kao `foobar@192.168.1.42`). Kasnije ćemo vidjeti ako prilagodimo ssh config datoteku možete pristupiti samo koristeći nešto kao `ssh bar`.

### Izvršavanje komandi

Funkcija `ssh` koja se često ne obazire jeste mogućnost da se komanda izvršava direktno. `ssh foobar@server ls` će izvršiti `ls` u home datoteci foobar. To radi sa pajpom, tako da će `ssh foobar@server ls | grep PATTERN` će lokalno grepovati udaljeni output `ls`-a i `ls | ssh foobar@server grep PATTERN` će grepovati udaljeni lokani output `ls`.

### SSH ključevi

Autentikacija na bazi ključeva koristi kriptografiju javnog ključa kako bi dokazala serveru da klijent posjeduje tajni privatni ključ bez otkrivanja ključa. Na ovaj način ne morate ponovo da unosite password svaki put. Ipak, privatni ključ (često kao ` ~/.ssh/id_rsa` i još češće kao `~/.ssh/id_ed25519`) je efektivno vaš password, tako ga i tretirajte.

### Generisanje ključeva 

Da bi generisali par možete izvršiti [ssh-keygen](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html).

```shell
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

Trebali bi da izaberete lozinku, da bi izbjegli da neko ko dođe do vašeg privatnog ključa ima autorizovani pristup serverima. Koristite [ssh-agent](https://www.man7.org/linux/man-pages/man1/ssh-agent.1.html) ili [gpg-agent](https://linux.die.net/man/1/gpg-agent) tako da ne morate da kucate vašu lozinku svaki put.

Ako ste ikada podešavali pushing na GitHub koristeći SSH ključ, onda ste vjerovatno napravili korake koji su navedeni [ovdje](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh) i već imate validan par ključeva. Da bi provjerili da li imate lozinku i da je potvrdite možete izvršiti `ssh-keygen -y -f /path/to/key`.

### Autentikacija bazirana na ključevima

`ssh` će provjeriti u `.ssh/authorized_keys` da bi odredio kog klijenta bi trebao da pusti. Da bi kopirali javni ključ ovdje možete koristiti: 

```shell
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

Jednostavnije rešenje može biti postignuto sa `ssh-copy-id` tamo gdje je dostupno: 

```shell
ssh-copy-id -i .ssh/id_ed25519.pub foobar@remote
```
### Kopiranje datoteka preko SSH-a

Postoji mnogo načina da kopirate datoteke preko SSH:

- `ssh+tee`, najlakše je koristiti `ssh` izvršenje komande i STDIN input koristeći `cat localfile | ssh remote_server tee serverfile`. Sjetite se da [tee](https://www.man7.org/linux/man-pages/man1/tee.1.html) ispisuje output iz STDIN u datoteku. 
- [scp](https://www.man7.org/linux/man-pages/man1/scp.1.html) kada kopirate veliku količinu datoteka/direktorijuma, sigurna copy `scp` komanda je pogodnija jer se lakše može kretati kroz paths. Sintaksa je `scp path/to/local_file remote_host:path/to/remote_file`.
- [rsync](https://www.man7.org/linux/man-pages/man1/rsync.1.html) je poboljšanje u odnosu na `scp` zbog detektovanja indetičnih datoteka lokalno i udaljeno, i sprečavanja njihovog ponovnog kopiranja. Takođe omogućava bolju kontrolu nad symlinks, dozvolama i ima dodatne funkicje kao što su `--partial` flag koji može nastaviti iz prethodno prekinute kopije. `rsync` ima sličnu sintaksu kao i `scp`.

### Prosleđivanje port-a

U mnogim scenarijima ćete naići na software koji sluša određene portove na mašini. Kada se ovo desi na vašoj lokalnoj mašini možete kucati `localhost:PORT` ili `127.0.0.1:PORT`, ali šta radite sa udaljenim serverima čiji port nije direktno dostupan kroz network/internet?

Ovo se naziva __prosleđivanje port-a__ i dolazi u dvije varijatnte: Local Port Forwarding and Remote Port Forwarding (pogledajte slike za više detalja, slike su sa [ovog StackOverflow posta](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot).

### Local Port Forwarding

![img1][img1]

[img1]: https://i.stack.imgur.com/a28N8.png%C2%A0

### Remote Port Forwarding 

![img2][img2]

[img2]: https://i.stack.imgur.com/4iK3b.png%C2%A0

