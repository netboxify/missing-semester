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
 
 
