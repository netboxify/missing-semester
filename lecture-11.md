# Q&A

<a href="http://www.youtube.com/watch?feature=player_embedded&v=Wz50FvGG6xU
" target="_blank"><img src="" 
alt="Lecture 11: Q&A (2020)" width="240" height="180" border="10" /></a>

U poslednjoj lekciji, odgovorili smo na pitanja koja su studenti postavili:

- [Da li imate neke preporuke za učenje tema koje se tiču operativnih sistema kao što su procesi, virtuelna memorija, prekidi, menadzment memorije i slično ?](#da-li-imate-neke-preporuke-za-učenje-tema-koje-se-tiču-operativnih-sistema-kao-što-su-procesi,-virtuelna-memorija,-prekidi,-menadzment-memorije-i-slično?)
- [Koji su alati za koje mislite da bi trebali da imaju prioritet u učenju?](#koji-su-alati-za-koje-mislite-da-bi-trebali-da-imaju-prioritet-u-učenju?)
- [Kada koristim Python u odnosu na Bash skripte ili neke druge jezike?](#kada-koristim-python-u-odnosu-na-bash-skripte-ili-neke-druge-jezike-?)
- [Koja je razlika između `source script.sh` i `./script.sh` ?]()
- [Gdje su mjesta u kojima se čuvaju razni paketi i alati i kako njihovo referenciranje funkcioniše. Šta je ustvari `./bin` ili `./lib`?]()
- [Da li bih trebao da izvršavam `apt-get install` python-ili nešto drugo, ili `pip install` nekog paketa?]()
- [Koji je najlakši i najbolji alat za profilisanje da bi se unaprijedile performanse mog koda?]()
- [Koje plugine za pretraživače vi koristite?]()
- [Koji su drugi korisni alati za upravljanje podacima?]()
- [Koja je razlika između Dockera i Virtuelne mašine]()
- [Koje su prednosti i mane svakog od operativnih sistema i kako da biramo između njih (npr. izbor najboljeg Linux distra za naše potrebe)]()
- [Vim vs Emacs?]()
- [Neki trik ili savjet za Machine Learning aplikaciju?]()
- [Još neki Vim trik?]()
- [Šta je 2FA i zašto bih trebao da je koristim?]()
- [Neki komentar na razlike između veb pretraživača?]()

## Da li imate neke preporuke za učenje tema koje se tiču operativnih sistema kao što su procesi, virtuelna memorija, prekidi, menadzment memorije i sl. ?

Prvo, nije jasno da li bi stvarno trebali da budete upoznati sa navedenim temama budući da su to sve teme vezane za niži nivo. One će postati bitne ukoliko počnete da pišete kod nižeg nivoa za implementiranje ili podešavanje kernela. Drugačije, većina tema neće biti od značaja, sa izuzetkom procesa i signala koji su ukratko pokriveni u drugim lekcijama.

Neki dobri resursi za učenje o ovim temama:

-[MIT's 6.828 class](https://pdos.csail.mit.edu/6.828/) - Lekcija diplomskog nivoa za inženjering operativnih sistema. Materijal lekcija sa časa je javno dostupan.
- Moderni operativni sistemi (4 izdanje) - od autora Andrew S. Tanenbaum je dobar pregled mnogih pomenutih koncepata.
- Dizajn i implementacija FreeBSD operativnih sistema - dobar resurs o FreeBSD operativnim sistemima (napomena: ovo nije Linux).
- Drugi vodiči kao što je [pisanje operativnog sistema u Rust-u](https://os.phil-opp.com/) gdje ljudi implementiraju kernel korak po korak u raznim jezicima, najviše za svrhe predavanja.

## Koji su alati za koje mislite da bi trebali da imaju prioritet u učenju ?

Neke teme kojima vrijedi dati prioritet:

- Učiti kako da više koristite tastaturu i da manje koristite miš. Ovo se izvodi kroz prečice za tastaturu, mijenjanje interfejsa i sl.
- Naučiti dobro vaš editor. Kao programer ćete većinu vremena provesti u uređivanju fajlova, tako da se razvijanje ove vještine veoma isplati.
- Naučite kako da automatizujete/pojednostavite zadatke koji se ponavljaju u vašem radu jer će ušteda vremena biti ogromna.
- Učenje alata o kontroli verzije kao što je Git i kako ga koristiti u kombinaciji sa GitHubom da bi sarađivali na modernim softverskim projektima.

## Kada koristim Python u odnosu na Bash skripte ili neke druge jezike ?

Generalno, bash skripte su korisne kao kratke i jednostavne skirpte, kada želite da izvršite specifičnu seriju komandi. Bash ima dio čudnih dijelova koji čine da bude jako teško raditi sa većim programima ili skriptama:

- bash je lak za jednostavne slučajeve upotrebe ali može biti izuzetno težak za sve moguće inpute. Na primer, razmaci kao argumenti skripte su doveli do nebrojenih bagova u bash skriptama.
- bash nije podložan ponovnoj upotrebi koda tako da može biti teško ponovno koristiti komponente od prethodnih programa koje ste napisali. Još opštije, ne postoji koncept softverskih biblioteka u bash-u.
- bash se oslanja na mnoge magične stringove kao što su `$?` ili `$@` da bi uputio na određene vrijednosti, dok drugi jezici na njih upućuju eksplicitnije, kao npr. `exitCode` ili `sys.args`.

Tako da, za veće i kompleksnije skirpte preporučujemo da se koristi zreliji skripting jezik kao što je Python ili Ruby. Možete pronaći online veliki broj biblioteka koje su ljudi već napisali da bi riješili česte probleme u ovim jezicima. Ukoliko pronađete biblioteke koje implementiraju specifičnu funkcionalnost do koje vam je stalo u nekom jeziku, obično je najbolja stvar koju možete da uradite da koristite taj jezik.

## Koja je razlika između `source script.sh` i `./script.sh` ?

U oba slučaja `script.sh` će biti pročitan i izvršen u bash sesiji, razlika leži u tome koja sesija izvršava komandu. Za `source` komanda se izvršava u vašoj trenutnoj bash sesiji i dodatno, bilo koje promjene koje su napravljene u trenutnom okruženju, kao što je mijenjanje direktorijuma ili definisanje funkcija će ostati u trenutnoj sesiji jednom kada `source` komanda završi sa izvršavanjem. Kada pokrećete samu skriptu kao što je `./script.sh`, vaša trenutna bash sesija počinje sa novom instancom bash-a koji će pokrenuti komande u `script.sh`. Dodatno, ukoliko `script.sh` mijenja direktorijum, nova bash instanca će promijeniti direktorijum, ali 
kada završi i vrati kontrolu parent bash sesiji, parent sesija će ostati na istom mjestu. Slično, ukoliko `script.sh` definiše funkciju kojoj želite da pristupite u vašem terminalu, potreban vam je `source` za to da bi bila dostupna u vašoj trenutnoj bash sesiji. Drugačije, ukoliko je pokrenete, novi bash proces će biti onaj koji će obraditi definiciju funkcije umjesto vašeg trenutnog shell-a.

## Gdje su mjesta u kojima se čuvaju razni paketi i alati i kako njihovo referenciranje funkcioniše. Šta je ustvari `./bin` ili `./lib`?

Što se tiče programa koje izvršavate u vašem terminalu, svi se nalaze u direktorijumu koji su navedeni u vašem `PATH` okruženju varijabli i možete koristiti `which` komandu (ili `type` komandu) da bi provjerili gdje vaš shell pronalazi specifični program. Uopšteno, postoje neke konvencije u vezi toga gdje specifični tipovi podataka žive. Evo nekih o kojima smo pričali, provjerite [Sistem fajlova, standard hijerarhije](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) za obimniju listu.

- `/bin` - Osnovna binarna komanda
- `/sbin` - Osnovni binarni sistem, obično se pokreće kao root
- `/dev` - Datoteke uređaja, posebne datoteke koje su često interfejsi hardverskim uređajima
- `/etc` - Host-specifikacija za široke sistemske konfiguracijske fajlove 
- `/home` - Home direktorijumi za korisnike u sistemu
- `/lib` - Česte biblioteke za sistemske programe
- `/opt` - Opcioni softver aplikacije
- `/sys` - Sadrži informacije i konfiguracije za sistem (pokriveni u [prvoj lekciji](https://missing.csail.mit.edu/2020/course-shell/))
- `/tmp` - Privremeni fajlovi (isto kao i `/var/tmp`). Obično se brišu između dva podizanja sistema
- `/usr/` - Čitaju samo podatke korisnika
    - `/usr/bin` - Ne-suštinska binarna komanda
    - `/usr/sbin` - Ne-suštinski binarni sistem, obično se pokreće kao root
    - `/usr/local/bin` - Binarni podaci za korisničke kompajlirane programe
- `/var` - Fajlovi sa varijablama kao što su logs ili keš.

## Da li bih trebao da izvršavam `apt-get install` python-ili nešto drugo, ili `pip install` nekog paketa?

Ne postoji univerzalan odgovor na ovo pitanje. Više se odnosi na generalno pitanje da li bi trebalo da koristite paket menadzere sistema ili paket menadzere određenog jezika da bi instalirali softver. Nekoliko stvari treba uzeti u obzir:

- Uobičajeni paketi će biti dostupni kroz oba, ali oni koji su manje popularni ili koji su skorijeg datuma možda neće biti dostupni u vašem  menadzeru paketa sistema. U tom slučaju, korišćenje alata koji je vezan za specifični jezik je bolja opcija.
- Slično menadzeri paketi za specifični jezik obično imaju savremenu verziju paketa u odnosu na menadzere paketa sistema.
- Kada koristite menadzere paketa sistema, biblioteke će biti instalirane širom sistema. To znači da ukoliko su vam potrebne različite verzije biblioteka za svrhe developmenta, menadzer paketa sistema možda neće biti dovoljan. Za ovaj scenario, većina programskih jezika pruža neku vrstu izolovanog ili virtuelnog okruženja tako da možete instalirati različite verzije biblioteke bez nailaženja na konflikte. Za Python, postoji virtualenv, a za Ruby, postoji RVM.
- Zavisno od operativnog sistema i arhitekture hardvera, neki od ovih paketa mogu doći sa binarnim ili će možda biti potrebno da se kompajliraju. Na primer, u ARM kompjuterima kao što je Raspberry Pi, korišćenje menadzera paketa sistema može biti bolji nego sa nekim koji je vezan za specifični jezik, ukoliko on dolazi u binarnoj formi i kasnije mora biti kompajliran. Ovo je veoma zavisno od vaše specifične postavke. 

Možete probati da koristite jedno rešenje ili drugo, ali ne i oba jer to može dovesti do konflikta kod kojeg je teško ispraviti greške. Naša preporuka je da koristite menadzer paketa za specifični jezik kada god je to moguće, i da koristite izolovano okruženje (kao Pythonov virtualenv) da bi izbjegli zagađenje globalnog okruženja.

## Koji je najlakši i najbolji alat za profilisanje da bi se unaprijedile performanse mog koda?

Najlakši alat koji je koristan za profilisanje je [print timing](https://missing.csail.mit.edu/2020/debugging-profiling/#timing). Vi samo ručno izračunate vrijeme koje je potrebno između dva različita dijela vašeg koda. Ukoliko ovo stalno ponavljate, možete efetkivno uraditi binarnu pretragu vašeg koda, i pronaći segmente koda kojima je trebalo najviše vremena.

Za naprednije alate, Valgrindov [Callgrind](http://valgrind.org/docs/manual/cl-manual.html) vam dopušta da pokrenete programe i mjerite koliko je svemu potrebno vremena i sve stakove poziva, naime koja funkcija poziva drugu funkciju. On zatim stvara verzuju vašeg programa sa napomenama izvornog koda sa vremenom koje je bilo potrebno za svaku liniju. Ipak, on usporava vaš program redom veličine i ne podržava niti. Za druge slučajeve, [perf](http://www.brendangregg.com/perf.html) alat i drugi specifični profajleri uzorkovanja programa mogu proizvesti korisne podatke veoma brzo. [Flamegraphs](http://www.brendangregg.com/flamegraphs.html) je dobar alat za vizualizaciju za navedene profajlere za uzorkovanje. Trebali bi takođe da koristite specifične alate za programske jezike ili zadatke na kojima radite. Na primer, za veb development, alatke za programere koje su ugrađene u Chrome i Firefox imaju fantastične profajlere.

Ponekad dio vašeg koda će biti spor jer sistem čeka na događaj kao što je čitanje diska ili network paket. U ovim slučajevima, vrijedno je provjeriti omotnicu kalkulacija o teorijskim brzinama u pogledu hardverskih sposobnosti da ne odstupa od stvarnih očitavanja. Postoje takođe specijalizovani alati za analiziranje vremena provedenog čekajući u sistemskim pozivima. Posebno je [bpftrace](https://github.com/iovisor/bpftrace) vrijedan provjere ukoliko morate da izvedete ovakvu vrstu niskog nivoa profilisanja.

## Koje plugine za pretraživače vi koristite?

Neki od naših omiljenih, najviše se odnose na bezbjednost i upotrebljivost:

- [uBlockOrigin](https://github.com/gorhill/uBlock) - On je blocker [širokog spektra](https://github.com/gorhill/uBlock/wiki/Blocking-mode) koje ne zaustavlja samo reklame, nego i sve vrste komunikacija treće strane koju bi stranica pokušala da uradi. Ovo takođe obuhvata inline skripte i druge vrste učitavanja resursa. Ukoliko ste voljni da provedete neko vrijeme na podešavanjima da bi stvari radile, idite na [medium mode](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-medium-mode) ili čak na [hard mode](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-hard-mode). Ovo će učiniti da neki sajtovi ne funkcionišu dok se dovolno ne pokrenete sa postavkama, ali će takođe veoma unaprijediti vašu online bezbjednost. Drugi način, [easy mode](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-easy-mode) je dobar default koji blokira većinu reklama i trackera. Možete takođe definisati vaša pravila o tome koji objekat sajta bi trebalo blokirati.
- [Stylus](https://github.com/openstyles/stylus/) fork Stylish-a(nemojte koristiti Stylish, pokazalo se da [krade korisnikovu istoriju](https://www.theregister.co.uk/2018/07/05/browsers_pull_stylish_but_invasive_browser_extension/), dozvoljava vam da sa strane učitate ručne CSS fajlove na sajtu. Ovdje možete skloniti sidebar, promijeniti pozadinsku boju ili čak veličinu teksta ili izbor fonta. Ovo je fantastično da učinite čitljivijim sajtove koje često posjećujete. Štaviše, Stylus može pronaći stilove koji su napisani od strane drugih korisnika i objavljeni u [userstyles.org](https://userstyles.org/). Većina sajtova, na primer, ima jednu ili više crnih tema. 
- Snimak ekrana čitave stranice - Ugrađen je u Firefox i [Chrome ekstenzije](https://chrome.google.com/webstore/detail/full-page-screen-capture/fdpohaocaechififmbbbbbknoalclacl?hl=en). Dopušta vam da napravite snimak ekrana čitave stranice, često mnogo bolji od štampanja u referentne svrhe.
- [Kontejneri za više naloga](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/) - dopušta vam da odvojite cookies u "kontejnere", dopuštajući vam da pretražujete veb sa različitim identitetima i/ili osiguravajući da sajtovi ne mogu da dijele informacije između sebe.
- Integracija menadzera lozinke - Većina menadzera lozinki imaju pretraživačke ekstenzije koje čine da unošenje vaših kredencijala ne bude samo pogodnije nego i sigurnije. U poređenju sa jednostavnim copy/pasting vašeg korisničkog imena i lozinke, ovi alati će prvo provjeriti da li je domen sajta na listi, sprečavajući phishing napade koji se lažno predstavljaju da bi ukrali kredencijale.

## Koji su drugi korisni alati za upravljanje podacima?

Neki od alata za upravljanje podacima za koje nismo imali vremena za obrađivanje tokom lekcije o upravljanju podacima uključuju `jq` ili `pup` koji su specijalizovani parseri za JSON i HTML podatke. Perl programski jezik je još jedan dobar alat za naprednije pajplajne upravljanja podataka. Još jedan trik je `column -t` komanda koja može biti korišćena da se konvertuje bijeli prostor teksta (koji možda nije poravnan) u pravilno usklađen tekst u koloni. 

Još opštije, par nekonvencionalnih alata za upravljanje podacima su Vim i Python. Za neke kompleksne i višelinijske transformacije, vim makroi mogu biti neprocjenjivi za korišćenje. Možete snimiti niz akcija i ponoviti ih onoliko puta koliko želite, na primer u editorima [zabilješki lekcija](https://missing.csail.mit.edu/2020/editors/#macros) (i [videu](https://missing.csail.mit.edu/2019/editors/) iz prošle godine) postoji primjer konvertovanja XML-formatiranog fajla u JSON koristeći samo vim makroe.

Za tabelarne podatke, često predstavljene u CSVs, [pandas](https://pandas.pydata.org/) Python biblioteka je odličan alat. Ne samo što čini veoma lakim definisanje kompleksnih operacija kao što je grupisanje, spajanje ili filtriranje; nego čini veoma lakim plotting različitih svojstava vaših podataka. Takođe podržava eksport u mnoge tabelarne formate uključujući XLS, HTML ili LaTeX. Alternativno R programski jezik (argumentovano [loš](http://arrgh.tim-smith.us/) programski jezik) ima puno funkcionalnosti za računanje statistike na osnovu podataka i može biti veoma koristan kao poslednji korak vašeg pajplajna. [ggplot2](https://ggplot2.tidyverse.org/) je odlična plotting biblioteka u R.

## Koja je razlika između Dockera i Virtuelne mašine?

Docker se bazira na generalnijem konceptu zvanom kontejneri. Glavna razlika između kontejnera i virtuelne mašine jeste da će virtuelna mašina izvršiti čitav OS stack, uključujući kernel, čak iako je kernel isti kao kod host mašine. Za razliku od VM, kontejneri izbjegavaju izvršavanje još jedne instance kernela i umjesto toga dijele kernel sa hostom. U Linuxu, ovo se izvršava kroz mehanizam koji se naziva LXC, i koristi niz mehanizama za izolaciju da bi napravio program koji misli da se pokreće na sopstvenom hardveru ali zapravo on dijeli hardver i kernel sa hostom. Dodatno, kontejneri imaju niži overhead nego čitava VM. Na drugoj strani, kontejneri imaju slabiju izolaciju i samo rade ukoliko host pokreće isti kernel. Na primer, ukoliko pokrenete Docker na macOS, Docker mora da zavrti Linux virtuelnu mašinu da bi dobio inicijalni Linux kernel a dodatno je overhead prilično značajan. Najzad, Docker je specifična implementacija kontejnera i napravljen je za softver deployment. Zbog ovoga, ima određene mane: na primer, Docker kontejneri neće zadržati bilo koju formu skladišta između dva podizanja sistema po default-u.

## Koje su prednosti i mane svakog od operativnih sistema i kako da biramo između njih (npr. izbor najboljeg Linux distra za naše potrebe)

Kada su u pitanju Linux distroi, iako ih ima mnogo, većina njih će se ponašati gotovo jednako za mnoge slučajeve. Mnoge Linux i UNIX funkcije i unutrašnji rad se mogu naučiti iz bilo kojeg distra. Osnovna razlika između njih jeste kako se nose sa ažuriranjima paketa. Neki distroi, kao što je Arch Linux, koriste rolling politiku ažuriranja sa najnovijim stvarima, ali one se često mogu pokvariti. Na drugoj strani, neki distroi kao što je Debian, CentOS ili Ubuntu LTS su mnogo konzervativniji sa izbacivanjem ažuriranja u njihova skladišta tako da su stvari obično mnogo stabilnije po cijenu žrtvovanja novijih funkcija. Naša preporuka za lako i stabilno iskustvo i na desktopu i na serveru jeste da koristite Debian ili Ubuntu.

Mac OS je dobra tačka u sredini između Windowsa i Linuxa koji ima lijep interfejs. Ipak, Mac OS se bazira na BSD prije nego na Linuxu, tako da su neki dijelovi sistema i komande različiti. Alternativa vrijedna provjere je FreeBSD. Iako se neki programi neće pokrenuti na FreeBSD, BSD ekosistem je mnogo manje fragmentriran i bolje dokumentovan u odnosu na Linux. Obeshrabrujemo Windows za bilo šta osim za razvoj Windows aplikacija, ili ukoliko postoji neka veoma bitna funkcija koja vam je potrebna, kao što je dobra podrška za drajvere za gejming.

Za dual boot sisteme, smatramo da je najprimjereniji bootcamp macOS-a i da bilo koja druga kombinacija može biti problematična na duže staze, posebno ukoliko je kombinujete sa drugim funkcijama kao što je disk enkripcija.

## Vim vs Emacs?

Nas troje koristimo Vim kao primarni editor ali je i Emacs takođe dobra alternativa i vrijedi probati i jedno i drugo da bi vidjeli šta vam bolje odgovara. Emacs ne prati Vimovo modalno uređivanje, ali ovo može biti omogućeno kroz Emacs plugin kao što je [Evil](https://github.com/emacs-evil/evil) ili [Doom Emacs](https://github.com/hlissner/doom-emacs). Prednost korišćenja Emacsa jeste da ekstenzije mogu biti implementirane u Lisp, bolji skripting jezik u odnosu na vimscript, Vimov defaultni skripting jezik.

## Neki trik ili savjet za Machine Learning aplikaciju?

Neke od lekcija iz ovog časa se mogu direktno primjeniti na ML aplikacije. Kao i u slučaju mnogih naučnih disciplina, u ML vi često izvodite seriju eksperimenata i želite da provjerite koje stvari su radile a koje nisu. Možete koristiti shell alate da lako i brzo pretražite kroz te eksperimente i objedinite rezultate na senzibilan način. Ovo bi moglo da znači subselektovanje svih eksperimenata u datom vremenskom okviru ili koji koriste određeni skup podataka. Korišćenjem JSON fajla da sačuvate sve relevantne parametre eksperimenta, ovo može biti veoma jednostavno sa alatima koje smo pokrili u ovim lekcijama. Najzad, ukoliko ne radite sa nekom vrstom klastera gdje prilažete vaše GPU poslove, trebali bi da pogledate kako da automatizujete ovaj proces jer to može biti zadatak koji vam oduzima puno vremena i troši vašu mentalnu energiju.

## Još neki Vim trik?

Par dodatnih trikova:

- Plugini - Uzmite vremena i istražite pejzaž plugina. Postoji mnogo dobrih plugina koji adresiraju neke nedostatke Vima ili dodaju nove funkcionalnosti koje se dobro uklapaju u postojeći Vim rad. Za ovo, dobri resursi su [Vim Awesome](https://vimawesome.com/) i drugi programerski dot fajlovi. 
- Marks - u Vimu, možete postaviti mark radeći `m<X>` za neko slovo `X`. Možete onda ići nazad na taj mark koristeći `'<X>`. Ovo vam pruža brzu navigaciju na specifičnu lokaciju unutar fajla ili izvan fajla. 
- Navigacija - `Ctrl+O` i `Ctrl+I` vas vode nazad ili naprijed kroz skoro posjećene lokacije.
- Undo Tree - Vim ima prilično fensi mehanizam za praćenje promjena. Za razliku od drugih editora, vim čuva drvo promjena pa čak iako uradite undo i onda napravite različitu izmjenu još uvijek možete ići nazad na originalno stanje krećući se kroz undo tree. Neki plugini kao što je [gundo.vim](https://github.com/sjl/gundo.vim) i [undotree](https://github.com/mbbill/undotree) prikazuju ovaj tree na grafički način. 
- Undo sa vremenom - `:earlier` i `:later` komande će vam dopustiti navigaciju kroz fajlove koristeći vremenske reference umjesto jedne promjene istovremeno.
- [Persistent Undo](https://vim.fandom.com/wiki/Using_undo_branches#Persistent_undo) je sjajna ugrađena funkcija vima koja je po defaultu onemogućena. On čuva vim undo istoriju između vim invokacija. Podešavanjem `undofile` i `undodir` u vašem `.vimrc`, vim će čuvati per-file istoriju promjena.
- Leader key - Leader key je specijalan ključ koji se često ostavlja korisnicima da ga podese za prilagođene komande. Obrazac je često pritisnuti i otpustiti ovo dugme (često space dugme) i onda neko drugo dugme da bi se izvršila određena komanda. Često, plugini će koristiti ovaj ključ da bi dodali njihovu sopstvenu funkcionalnost, na primer UndoTree plugin koristi `<Leader> U` da bi otvorio undo drvo.
- Napredni tekst objekti - Tekst objekti kao što su pretrage takođe mogu biti napravljeni sa Vim komandama. Npr. `d/<pattern>` će brisati do sledećeg poklapanja datog obrasca ili `cgn` će promijeniti sledeću pojavu poslednje pretraženog stringa.

## Šta je 2FA i zašto bih trebao da je koristim?

Dvostruka autentikacija (2FA) dodaje poseban sloj zaštite vašim nalozima iznad vaših lozinki. Da bi se ulogovali, ne samo da morate da znate neku lozinku, već takođe morate "dokazati" na neki način da imate pristup nekom hardveru. U većini jednostavnih slučajeva, ovo može biti ostvareno primanjem SMS-a na vaš telefon, iako postoje [poznati problemi](https://www.kaspersky.com/blog/2fa-practical-guide/24219/) sa SMS 2FA. Bolja alternativa koju mi odobravamo jeste korišćenje [U2F](https://en.wikipedia.org/wiki/Universal_2nd_Factor) rešenja kao što je [YubiKey](https://www.yubico.com/).

## Neki komentar na razlike između veb pretraživača?

Trenutni pejzaž pretraživača u 2020 jeste da su većina njih kao Chrome zato što koriste isti engine (Blink). Ovo znači da je Microsoft Edge koji je takođe baziran na Blinku, i Safari, koji je baziran na WebKit, sličnom enginu kao što je Blink, su samo gore verzije od Chrome-a. Chrome je dosta dobar pretraživač i u pogledu performansi i upotrebljivosti. Ukoliko želite alternativu, Firefox je naša preporuka. Uporediv je sa Chrome-om na gotovo svim poljima, i nadmašuje ga u pogledu privatnosti. Još jedan pretraživač koji se zove [Flow](https://www.ekioh.com/flow-browser/) nije još spreman za korisnike, ali implementira novi rendering engine koji obećava da će biti brži od onih koje trenutno imamo.
