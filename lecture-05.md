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

```console
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

