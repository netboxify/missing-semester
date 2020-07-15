# Editori (Vim)

<a href="http://www.youtube.com/watch?feature=player_embedded&v=a6Q8Na575qc
" target="_blank"><img src="" 
alt="Lecture 3: Editors(Vim) (2020)" width="240" height="180" border="10" /></a>

Pisanje riječi na engleskom i pisanje koda su dvije potpuno različite stvari. Dok programirate, provodite više vremena mijenjajući datoteke, čitajući, krećući se i uređujući kod u poređenju sa pisanjem teksta. Ima smisla da postoje različite vrste programa za pisanje Engleskih riječi u odnosu na kod (npr. Microsoft Word vs. Visual Studio Code).

Kao programeri, koristimo najveći dio našeg vremene uređujući kod, tako da je isplativo uložiti vaše vrijeme da bi postali dobri u korišćenju editora koji zadovoljava vaše potrebe. Evo kako se uči novi editor: 

- Počnite sa tutorijalom (npr. ova lekcija, plus resursi na koje vas uputimo)
- Koristite editor za sve potrebe uređivanja teksta (čak iako vas to u startu usporava)
- Istražujte stvari u hodu: ukoliko vam se čini da postoji bolji način da se nešto odradi, on vjerovatno i postoji.

Ukoliko budete slijedili, i u potpunosti se posvetite korišćenju novog programa za svaku svrhu uređivanja teksta, vremenski okvir za učenje sofisticiranog tekst editora izgleda ovako. Za sat ili dva, naučićete osnovne funkcije editora kao što je otvaranje i uređivanje datoteka, sačuvaj/izađi i navigacija. Nakon 20 sati, trebali bi da budete brzi kao što ste bili sa vašim starim editorom. Nakon toga, počinju privilegije: Imaćete dovoljno znanja i memorije mišića tako da će vam korišćenje novog editora uštedjeti na vremenu. Moderni text editori su moćni i fensi alati, tako da učenje nikada ne prestaje: bićete sve brži kako učite

## Koji editor da naučite?

Programeri imaju [oprečna mišljenja](https://en.wikipedia.org/wiki/Editor_war) u vezi sa editorima teksta. 

Koji editoru su popularni danas? Pogledajte ovaj [StackOverflow survey](https://insights.stackoverflow.com/survey/2019/#development-environments-and-tools) (može postojati i pristrasnost budući da korisnici StackOverflowa možda nisu reprezentativni za programere u cjelini). [Visual Studio Code](https://code.visualstudio.com/) je najpopularniji editor. [Vim](https://www.vim.org/) je najpopularniji editor koji se zasniva na komandnoj liniji. 

### Vim 

Svi instruktori ovih lekcija koriste Vim kao njihov editor. Vim ima bogatu istoriju. Potiče od Vi editora (1976), i razvija se i dan danas. Iza Vima stoje neke veoma uredne ideje, i iz ovog razloga, puno alata podržava Vim mod emulacije (npr. 1.4 miliona ljudi je instaliralo Vim emulator za VS code). Vim je vjerovatno vrijedan učenja čak iako na kraju ipak pređete na neki drugi editor.

Nije moguće naučiti svu funkcionalnost Vim-a za 50 minuta, tako da ćemo se fokusirati na objašnjenje filozofije Vim-a, naučiti vas osnove, pokazati vam neke napredne funkcionalnosti, i dati vam resursi da bi savladali alat.

## Filozofija Vim-a

Kada programirate, koristite najveći dio vašeg vremena čitajući/uređujući, a ne pišući. Iz ovog razloga, Vim je __modalni__ editor: ima različite modove za unošenje teksta u odnosu na manipulaciju tekstom. Vim je programatičan (Sa Vimscript i sa drugim jezicima kao što je Python), i Vim's interfejs je sam po sebi programski jezik: tasteri su komande, i ove komande su uklopljive. Vim izbjegava upotrebu miša, jer je to previše sporo, Vim čak izbjegava upotrebu dugmadi sa strelicama jer iziskuju previše pokreta.

Krajnji rezultat je editor koji se uklapa u brzinu kojom mislite.

## Režimsko uređivanje

Dizajn Vim-a se bazira na ideji da programeri provode dosta vremena čitajući, krećući se i praveći manje izmjene, u poređenju sa pisanjem dugih linija teksta. Iz ovog razloga, Vim ima više operativnih režima. 

- **Normal**: Za kretanje kroz fajl i pravljenje izmjena
- **Insert**: Za ubacivanje teksta
- **Replace**: Za zamjenu teksta
- **Visual**: (običan, linija ili blok): za selektovanje blokova teksta
- **Command-line**: Za pokretanje komandne linije

Pritisci na tastere imaju različita značenja u različitim režimima rada. Na primer, slovo `x` u insert režimu će samo umetnuti bukvalni znak x, ali u normalnom režimu će izbrisati karakter pod kursorom, a u vizuelnom režimu će izbrisati odabrano. 

U svojoj uobičajenoj konfiguraciji, Vim prikazuje trenutni režim u donjem lijevom uglu. Početni/Podrazumijevani režim je Normalni režim. Obično ćete provesti većinu vašeg vremena između Normalnog i Insert režima. 

Mijenjate režim pritiskom na `<ESC>` za prelazak sa bilo kojeg režima nazad u normalni režim. Iz normalnog režima uđite u Insert režim sa `i` Replace režim sa `R`, Visual režim sa `v`, Visual line režim sa `V`, Visual Block mode sa `<C-v>` (Ctrl-V, ponekad se piše kao `^V`), i režim komandne linije sa `:`.

Koristićete `<ESC>` dosta u toku korišćenja Vim-a, razmislite da zamijenite Caps Lock za Escape ([macOS instrukcije](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS))
 
 ## Osnove
 
 ### Ubacivanje teksta
 
 Iz Normalnog režima, pritisnite `i` za insert režim. Sada, Vim se ponaša kao bilo koji drugi text editor, dok ne pritisnete `<ESC>` da se vratite u normalni režim. Ovo, zajedno sa osnovama koje su gore objašnjene, je sve što vam je potrebno da bi počeli da koristite Vim kao tekstualni editor(iako nije izuzetno efikasno, ukoliko koristite svo vaše vrijeme editovanjem iz Insert režima).
 
### Baferi, tabovi, i windows

Vim održava set otvorenih datoteka, koji se nazivaju "baferi". Vim sesija ima broj tabova, svaki od njih ima broj windows-a. Svaki windows pokazuje jedan bafer. Za razliku od drugih programa koji su vam poznati, kao što su web pretraživači, ne postoji 1-1 korespondiranje između bafera i windows-a; Windows su samo pregledi. Dati bafer može biti otvoren u __više__ windows-a, čak i u okviru istog tab-a. Ovo može biti zgodno, na primer, da bi se prikazale dvije različite strane datoteke u isto vrijeme. 

Uobičajeno, Vim se otvara sa jednim tabom, koji sadrži jedan windows.

### Komandna linija 

Komandnoj liniji se može pristupiti kucanjem `:` u Normal režimu. Vaš kursor će preći na komandnu liniju na dnu vašeg ekrana nakon pritiskanja `:`. Ovaj mod ima mnogo funkcionalnosti, uključujući otvaranje, čuvanje, i zatvaranje datoteka, i [zatvaranje Vima](https://twitter.com/iamdevloper/status/435555976687923200).


- `:q` izađi (zatvori window)
- `:w` sačuvaj (“ispiši”)
- `:wq` sačuvaj i izađi
- `:e` {naziv datoteke} otvori datoteku za editovanje
- `:ls` prikaži otvorene bafere
- `:help` {topic} otvori pomoć
- `:help :w` otvori pomoć za `:w` komandu
- `:help w` otvori pomoć za `w` pokret

## Vim interfejs je programski jezik

Najvažnija ideja u Vim-u jeste da je Vimov intefejs u stvari programski jezik. Pritisci na tastere su komande, i ove komande __sastavljaju__. Ovo omogućava efikasno kretanje i editovanje, posebno kada komande postanu memorije mišića. 

### Kretanje 

Trebalo bi da većinu vašeg vremena provodite u Normal režimu, koristeći komande kretanja da bi se kretali kroz bafer. Kretanje kroz Vim se takođe naziva "imenica", zato što se ono odnosi na dijelove teksta. 

- Osnovno kretanje: `hjkl` (lijevo, dolje, gore, desno)
- Riječi: `w` (sledeća riječ), `b` (početak riječi), `e` (kraj riječi)
- Redovi: `0` (Početak reda), `^` (prvi non-blank karakter), `$` (kraj reda)
- Ekran: `H` (vrh ekrana), `M` (sredina ekrana), `L` (dno ekrana)
- Skrol: `Ctrl-u` (gore), `Ctrl-d` (dolje)
- File: `gg` (početak fajla), `G` (kraj fajla)
- Broj reda: `:{number}<CR>` or `{number}G` (line {number})
- Misc: `%` (odgovarajući item)
- Pronađi: `f{character}`, `t{character}`, `F{character}`, `T{character}`
       - find/to naprijed/nazad {character} na trenutnoj liniji.
       -  `,` / `;` za navigaciju sa podudaranjem
- Pretraga: `/{regex}, n / N` za navigaciju sa podudaranjem
 
 ### Selekcija 
 
 Visual režimi: 
  
- Visual
- Visual Line
- Visual Block

Mogu se koristiti ključevi za kretanje da bi se izvršila selekcija

### Editovanje 

Za sve što ste navikli da koristite miš, sada ćete raditi sa tastaturom koristeći komande za kretanje i editovanje. Ovdje Vim interfejs počinje da izgleda kao programski jezik. Vim komande za editovanje se nazivaju "glagoli", jer glagoli djeluju na imenice. 

- `i` enter Insert mode
  - ali za manipulaciju/brisanje teksta, želite da koristite nešto više od backspace-a.
- `o` / `O` ubaci liniju ispod/iznad 
- d{motion} izbriši{pokret}
  - npr. `dw` je izbriši riječ, `d$` je izbriši do kraja linije, `d0` je izbriši do početka linije
- `c{motion}` promijeni{pokret}
  - npr. `cw` je promjena riječi
  - kao `d{motion}` praćen sa `i`
- `x` izbriši karakter (jednako sa `dl`)
- `s` zamijeni karakter (jednako sa `xi`)
- Visual režim + manipulacija
  - izaberi tekst, `d` za brisanje ili `c` za promjenu. 
- `u` za undo, `<C-r>` za redo 
- `y` za kopiranje/"yank"(neke druge komande kao `d` takođe kopiraju)
- `p` za paste
- Još mnogo stvari za naučiti npr. `~` mijenja case karaktera

### Brojanje

Možete kombinovati imenice i glagole sa brojanjem, koje će izvršiti zadatku akciju više puta. 

- `3w` pomijera tri riječi naprijed
- `5j` pomijera pet linija dolje 
- `7dw` briše sedam riječi 

### Modifikatori

Možete koristiti modifikatore da promijenite značenje riječi. Neki modifikatori su `i`, koji znači "unutrašnji", ili `a`, koji znači "okolo".

- `ci(` mijenja sadržaj unutar para zagrada
- `ci[` mijenja sadržaj unutar uglastih zagrada 
- `da'`briše string sa jednostranim navodnicima, uključujući okružujuće jednostrane navodnike

## Demo 

Evo pokvarene [fizzbuzz](https://en.wikipedia.org/wiki/Fizz_buzz) implementacije

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print('fizz')
        if i % 5 == 0:
            print('fizz')
        if i % 3 and i % 5:
            print(i)

def main():
    fizz_buzz(10)
```

Popravićemo sledeće probleme:

- Main nikada nije pozvan
- Počinje sa 0 umjesto sa 1
- Ispisuje "fizz" i "buzz" na posebnim linijama za brojeve dijeljive sa 15
- Ispisuje "fizz" za brojeve dijelive sa 5 
- Koristi hard-kodiran argument 10 umjesto da primi argument komandne linije

Pogledajte video lekciju za demonstraciju. Uporedite kako se promjene vrše pomoću Vima sa načinom na koji možete izvršiti iste izmjene pomoću drugog programa. Vidite kako je u Vimu potrebno vrlo malo pritisaka tipki, što vam pruža mogućnost da uređujete brzinom kojom mislite. 

## Prilagođavanje Vima

Vim se prilagođava putem konfiguracijske datoteke u običnom tekstu u `~/.vimrc` (Sadži Vimscript komande). Vjerovatno da postoji dosta osnovnih podešavanje koje želite da uključite. 

Mi pružamo dobro dokumentovanu osnovnu konfiguraciju koju možete koristiti kao polaznu tačku. Preporučujemo je zato što podešava neko od Vimovih pokvarenih osnovnih ponašanja. **Preuzmite naš config [ovdje](https://missing.csail.mit.edu/2020/files/vimrc) i sačuvajte ga u `~/.vimrc`.**

Vim je veoma prilagodljiv, i vrijedi uložiti vrijeme u istraživanje opcija prilagođavanja. Možete vidjeti dotfiles drugih ljudi na GitHubu za inspiraciju, na primer, Vim configs vaših instruktora ([Anish](https://github.com/anishathalye/dotfiles/blob/master/vimrc), [Jon](https://github.com/jonhoo/configs/blob/master/editor/.config/nvim/init.vim)(koristi [neovim](https://neovim.io/)), [Jose](https://github.com/JJGO/dotfiles/blob/master/vim/.vimrc)).Takođe, postoji dosta dobrih blog postova na ovu temu. Pokušajte da ne radite samo copy-paste punih konfiguracija drugih ljudi, pročitajte ih, shvatite ih, i uzmite ono što vam je potrebno. 

## Ekstenzije Vima

Postoji puno plugina za ekstenziju Vima. Suprotno zastarelom savjetu koji možete naći na internetu, __nije__ vam potreban plugin menadzer za Vim (još od Vim 8.0). Umjesto toga, možete koristiti ugrađeni sistem upravljanja paketima. Jednostavno kreirajte direktorijum `~/.vim/pack/vendor/start/`, stavite plugine tu (npr. pomoću `git clone`). 

Evo nekih naših omiljenih plugina:

- [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim): fuzzy file finder
- [ack.vim](https://github.com/mileszs/ack.vim): code search  
- [nerdtree](https://github.com/preservim/nerdtree): file explorer 
- [vim-easymotion](https://github.com/easymotion/vim-easymotion): magic motions

Pokušavamo da izbjegnemo da vam damo veoma veliku listu plugina ovdje. Možete provjeriti dotfiles instruktora ([Anish](https://github.com/anishathalye/dotfiles). [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/JJGO/dotfiles)) da vidite koje druge plugine koriste. Pogledajte [Vim Awesome](https://vimawesome.com/) za više sjajnih Vim plugina. Takođe, postoji dosta blog postova na ovu temu, samo pretražite "Najbolji Vim plugini".

## Vim-mod u drugim programima

Mnogi alati podržavaju Vim emulaciju. Kvalitet varira od dobrog do odličnog; zavisno od alata, možda neće podržati fensi Vim funkcije, ali većina pokriva osnove veoma dobro. 

### Shell

Ukoliko ste Bash user, koristite `set -o vi`. Ukoliko koristite Zsh, `bindkey -v`. Za Fish, `fish_vi_key_bindings`. Dodatno, bez obzira koji shell koristite, možete `export EDITOR=vim`. Ovo je okruženje varijabli koje se koristi da bi se odlučilo koji editor se otvara kada program hoće da pokrene editor. Na primer, `git` će koristiti ovaj editor za commit poruku. 

### Readline 

Mnogi programi koriste [GNU Readline]() biblioteku za interfejs komandne linije. Readline podržava (osnovnu) Vim emulacije takođe, koja može biti omogućena dodavanjem sledeće linije u `~/.inputrc` fajl: 

```console
set editing-mode vi
```

Sa ovim podešavanjem, na primer, Python REPL će podržati Vim vezivanje.

### Others

Postoji čak i Vim keybinding ekstenzija za web [browsere](https://vim.fandom.com/wiki/Vim_key_bindings_for_web_browsers) - neki od popularnih su [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en) za Google Chrome i [Tridactyl](https://github.com/tridactyl/tridactyl) za Firefox. Možete čak dobiti Vim vezivanje u [Jupyter notebooks](https://github.com/lambdalisue/jupyter-vim-binding).

## Napredni Vim 

Evo par primjera koji će vam pokazati moć editora. Ne možemo vas naučiti sve ove stvari, ali ćete ih naučiti vremenom. Imajte na umu: kada god koristite editor i pomislite "mora da postoji bolji način da se ovo uradi", vjerovatno i postoji: pogledajte online. 

### Pretraži i zamijeni

`:s` (zamjena) komanda ([dokmentacija](https://vim.fandom.com/wiki/Search_and_replace))
- `%s/foo/bar/g` 
  - Mijenja foo sa bar globalno u fajlu
- `%s/\[.*\](\(.*\))/\1/g`
  - Mijenja naziv Markdown linka sa čistim URL-om
  
### Više prozora

- `:sp / :vsp` da podijeli prozore
- Može imati više views-a na isti buffer.

### Macros

- `q{character}` za početak snimanja macroa u registru `{character}`
- `q` za prekid snimanja
- `@{character}` replays macro
- Macro izvršenje se zaustavlja zbog greške
- `{number}@{character}` izvršava macro {number} puta
- Macroi mogu biti rekurzivni 
  - Prvo očistite macro sa `q{character}q`
  - Snimite macro, sa `@{character}` da bi pozvali macro rekurzivno
  - Primjer: Konvertujte xml u json ([file](https://missing.csail.mit.edu/2020/files/example-data.xml))
    - Niz objekata sa ključem “name” / “email”
    - Koristite Python program?
    - Koristite sed/regexes
      - `g/people/d`
      - `%s/<person>/{/g`
      - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
      - ...
    - Vim komande / macros
      - `Gdd`, `ggdd` briše prvu i poslednju liniju
      - Macro da formatira pojedinačan element (register `e`)
        - Ide na liniju sa `<name>`
        - `qe^r"f>s": "<ESC>f<C"<ESC>q`
      - Macro da formatira osobu 
        - Ide na liniju `<person>`
        - `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
      - Macro da formatira osobu i da ide na sledeću osobu 
        - Ide na linuju `<person>`
        - `qq@pjq`
      - Izvršava macro do kraja fajla
        - `999@q`
      - Ručno uklanja poslednje `,` i dodaje `[` i `]`
      
## Resursi

- `vimtutor` je tutorijal koji dolazi instaliran sa Vimom - ukoliko je Vim instaliran, trebalo bi da ste mogućnosti da pokrenete `vimtutor` iz shell-a.
- [Vim Adventures](https://vim-adventures.com/) je igra kojom se uči Vim
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/) ima razne Vim savjete
- [Vim Golf](www.vimgolf.com) je [code golf](https://en.wikipedia.org/wiki/Code_golf) ali gdje je programski jezik Vim UI
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/) (book)

## Vježbe

1. Završite `vimtutor`. Napomena: Najbolje izgleda na [80x40](https://en.wikipedia.org/wiki/VT100) (80 kolona sa 24 reda) prozora terminala.
2. Preuzmite naš [basic vimrc](https://missing.csail.mit.edu/2020/files/vimrc) i sačuvajte ga u `~/.vimrc`. Pročitajte dobro dokumentovane datoteke (koristeći Vim!) i pogledajte kako Vim izgleda i kako se ponaša malo drugačije sa novom konfiguracijom.
3. Instalirajte i podesite plugin: [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim).
  1. Kreirajte plugin direktorijum sa `mkdir -p ~/.vim/pack/vendor/start`
  2. Preuzmite plugin: `cd ~/.vim/pack/vendor/start; git clone https://github.com/ctrlpvim/ctrlp.vim`
  3. Pročitajte [dokumentaciju](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md) za plugin. Pokušajte da koristite CtrlP da locirate datoteku krećući se kroz direktorijum projekta, otvarajući Vim, i korišćenjem Vim komandne linije za početak `:CtrlP`.
  4. Izmijenite CtrlP dodavanjem [konfiguracije](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options) u vaš `~/.vimrc` da otvorite CtrlP pritiskajući Ctrl-P.
4. Da bi izvježbali Vim, ponovo odradite [Demo](https://missing.csail.mit.edu/2020/editors/#demo) iz ove lekcije na vašoj mašini.
5. Koristite Vim za svako vaše editovanje teksta u sledećem mjesecu. Kada god nešto izgleda neefikasno, ili pomislite "mora da postoji bolji način", Guglajte, vjerovatno i postoji. Ako zaglavite, dođite dok traju office hours ili nam pošaljite mejl.
6. Podesite vaše druge alata da koriste Vim vezivanje (Vidi instrukcije iznad).
7. Dalje prilagodite vaš `~/.vimrc` i instalirajte još plugina.
8. (Napredno) Konvertujte XML u JSON [Primjer fajla](https://missing.csail.mit.edu/2020/files/example-data.xml) koristeći Vim macroe. Pokušajte sami da ovo odradite, ali možete pogledati [macros](https://missing.csail.mit.edu/2020/editors/#macros) sekciju iznad ukoliko zaglavite.
