## Tabela sadržaja

- [Podešavanje tastature](#podešavanje-tastature)
- [Daemons](#daemons)
- [FUSE](#fuse)
- [Backups](#backups)
- [APIs](#apis)
- [Uobičajeni flagovi/obrasci komandne linije](#uobičajeni-flagovi-obrasci-komandne-linije)
- [Menadžeri prozora](#menadžeri-prozora)
- [VPN's](#vpns)
- [Markdown](#markdown)
- [Hammerspoon (desktop automatizacija na macOS)](#hammerspoon-desktop-automatizacija-na-macOS)
- [Booting + Live USBs](#booting--live-usbs)
- [Docker, Vagrant, VMs, Cloud, OpenStack](#docker-vagrant-vms-cloud-openstack)
- [Notebook programiranje](#notebook-programiranje)
- [GitHub](#github)

### Podešavanje tastature

Kao programer, vaša tastatura je vaš glavni input metod. Kao sa gotovo svim stvarima u vašem računaru, podesiva je (i vrijedi je podesiti).

Osnovna promjena je remapiranje tastera. Ovo obično uključuje neki softver koji osluškuje i, kada se određeni taster pristisne, on presretne taj događaj i zamjeni ga drugim događajem koji odgovara drugom tasteru. Neki od primjera:

- Remapirajte Caps Lock na Ctrl ili Escape. Mi (instruktori) ovo jako preporučujemo jer Caps Lock ima veoma pogodnu lokaciju, a rijetko se koristi.
- Remapirajte PrtSc na Play/Pause music. Većina OS imaju play/pause dugme.
- Zamijenite Ctrl i Meta (Windows ili Command) taster.

Možete takođe mapirati proizvoljne tastere po vašem izboru. Ovo je korisno za uobičajene zadatke koje izvodite. Ovdje, neki softveri osluškuju specifičnu kombinaciju tastera i izvršavaju određenu skriptu kada je taj događaj detektovan.

- Otvorite novi terminal ili pretraživač prozora.
- Ukucajte neki specifičan tekst npr. vašu dugu mejl adresu ili vaš MIT ID broj.
- Uspavajte računar ili ekran.

Postoje i kompleksnije modifikacije koje možete da napravite:

- Remapiranje dijela tastera, npr. pritisnite shift pet puta da bi upalili/ugasili Caps Lock.
- Remapiranje na dodir umjesto na držanje, npr. Caps Lock se remapira u Esc ukoliko ga brzo taknete, ali se remapira u Ctrl ukoliko ga duže držite i koristite ga kao modifikator.
- Određivanje podešavanja prilagođenih tastaturi ili softveru.

Neki softver resursi da biste počeli sa ovom temom:

- macOS - [karabiner-elements](https://karabiner-elements.pqrs.org/), [skhd](https://github.com/koekeishiya/skhd) ili [BetterTouchTool](https://folivora.ai/)
- Linux - [xmodmap](https://wiki.archlinux.org/index.php/Xmodmap) ili [Autokey](https://github.com/autokey/autokey)
- Windows - Ugrađeni u Control Panelu, [AutoHotkey](https://www.autohotkey.com/) ili [SharpKeys](https://www.randyrants.com/category/sharpkeys/)
- QMK - Ukoliko vaša tastatura podržava ručna podešavanja možete koristiti [QMK](https://docs.qmk.fm/) da bi podesili same hardver uređaje tako da remapiranje radi za bilo koju mašinu na kojoj koristite tastaturu.

### Daemons

Vjerovatno ste već upoznati sa pojmom daemona, iako se riječ čini kao nova. Većina računara prije ima seriju procesa koji su pokrenuti u pozadini nego što čeka korisnika da ih pokrenu i da vrše interakciju sa njima. Ovi procesi se nazivaju daemons a programi koji se pokreću kao daemon obično završavaju sa `d` da bi ukazali na to. Na primer `sshd`, SSH daemon, je program koji je odgovoran za slušanje dolaznih SSH zahtjeva i provjeru da li udaljeni korisnik ima neophodne akreditive da bi se ulogovao.

U Linuxu, `systemd` (system daemon) je najčešće rešenje za pokretanje i podešavanje daemon procesa. Možete pokrenuti `systemctl status` da bi izlistali trenutne daemone koji su pokrenuti. Većina njih vam može zvučati strano, ali su odgovorni za ključne dijelove sistema kao što je upravljanje networkom, rešavanje DNS upita ili prikazivanje grafičkog interfejsa za sistem. Systemd se može pristupiti komandom `systemctl` da bi se `omogućio`, `onemogućio`, `započeo`, `zaustavio`, `restartovao` ili provjerio `status` usluge (to su `systemctl` komande).

Još interesantnije, `systemd` ima sasvim prisutpan interfejs za podešavanje i omogućavanje novih daemons-a (ili usluga). Ispod je primjer daemona za pokretanje jednostavne Python aplikacije. Nećemo ići u detalje, ali kao što možete da vidite, većina polja su samorazumljiva.

```shell
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom App
After=network.target

[Service]
User=foo
Group=foo
WorkingDirectory=/home/foo/projects/mydaemon
ExecStart=/usr/bin/local/python3.7 app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### FUSE

Savremeni softverski sistemi se obično sastoje od manjih blokova koji su sastavljeni zajedno. Vaš operativni sistem podržava upotrebu različitih backend sistema fajlova jer postoji uobičajen jezik koji operacije sistema fajlova podržavaju. Na primer, kada izvršite `touch` da bi kreirali fajl, `touch` izvršava sistemski poziv kernelu da bi se kreirao fajl a kernel izvodi odgovarajući poziv sistemu fajlova da bi dati fajl bio kreiran. Imajte u vidu da su sistemi fajlova UNIX-a tradicionalno implementirani kao kernel moduli i samo je kernelu dozvoljeno da obavlja pozive sistema fajlova.

[FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) (Sistem fajlova u korisničkom prostoru) dozvoljava sistemu fajlova da bude implementiran od strane korisničkog programa. FUSE dozvoljava korisnicima da izvrše korisnički kod za poziv sistema fajlova i zatim spaja neophodni poziv sa interfejsom kernela. U praksi, to znači da korisnici mogu implementirati proizvoljnu funkcionalnost za pozive sistema fajlova.

Na primer, FUSE se može koristiti kada god izvršite operaciju u virtuelnom sistemu fajlova, ta operacija se prosleđuje kroz SSH do udaljene mašine, tamo se izvršava, i output se vraća vama. Na ovaj način, lokalni programi mogu vidjeti fajl kao da je u vašem sistemu, dok je u realnosti on na udaljenom serveru. To je ono što `sshfs` radi efektno. 

Neki zanimljivi primjeri FUSE sistema fajlova su:

- [sshfs](https://github.com/libfuse/sshfs) - Lokalno otvara udaljeni fajl/folder kroz SSH konekciju.
- [rclone](https://rclone.org/commands/rclone_mount/) - Postavlja cloud servis skladišta kao što je Dropbox, GDrive, Amazon S3 ili Google Cloud skladište i otvara podatke lokalno.
- [gocryptfs](https://nuetzlich.net/gocryptfs/) - Enkriptuje sistem prekrivanja. Fajlovi se čuvaju enkriptovani, ali kada se postavi FS oni se pojavljuju kao čisti tekst u tački postavljanja.
- [kbfs](https://keybase.io/docs/kbfs) - Distribuirani sistem fajlova sa end-to-end enkripcijom. Možete imati privatne, dijeljene i javne foldere.
- [borgbackup](https://borgbackup.readthedocs.io/en/stable/usage/mount.html) - Postavlja vaše deduplicirane, kompresovane i enkriptovane backup-e za lakše pretraživanje.

## Backups

Bilo koji podaci za koje nije izvršen back-up su podaci koji mogu nestati u svakom trenutku, zauvijek. Lako je kopirati podatke okolo, teško je izvršiti pouzdan back-up podataka. Evo nekih dobrih back-up osnova i zamki nekih pristupa.

Prva, kopija podataka na istom disku nije back-up, jer je disk zajednička tačka pada svih podataka. Slično, eksterni drajv u vašoj kući je takođe slabo backup rešenje jer može biti izgubljen u požaru/pljački i sl. Umjesto toga, imati backup van vaše lokacije je preporučena praksa.

Rešenja za sinhronizaciju nisu back-up. Na primer, Dropbo/GDrive su pogodna rešenja, ali kada su podaci izbrisani ili korumpirani oni šire tu promjenu. Iz istog razloga, disk mirroring rešenja kao što je RAID nisu backup-ovi. Oni ne pomažu ukoliko se podaci izbrišu, korumpiraju ili enkriptuju od strane virusa.

Neke ključne funkcije back-up rešenja su verziranje, deduplikacija i bezbjednost. Verziranje backup-a osigurava da možete pristupiti istoriji promjena i efektno oporaviti fajlove. Efektna backup rešenja koriste deduplikaciju podataka da samo sačuvaju inkrementalne promjene i smanje upotrebu skladišta. Što se tiče bezbjednosti, trebali bi da zapitate sebe šta bi neko trebao da zna/ima da bi mogao da čita vaše podatke i, još važnije, da izbriše sve vaše podatke koji su povezani sa backup-om. Najzad, slijepo vjerovanje backup-u je jako loša ideja i trebali bi redovno da provjeravate da li možete da ih koristite za oporavak podataka.

Backups idu izvan lokalnih fajlova u vašem računaru. Imajući u vidu značajan rast veb aplikacija, velike količine vaših podataka se samo čuvaju u cloud-u. Na primer, vaš veb mejl, slike sa društvenih mreža, muzička plejlista u striming servisu ili online dokumenti će nestati ukoliko izgubite pristup odgovarajućim nalozima. Imajući offline kopiju ovih informacija je put kojim treba ići, i možete pronaći online alate koje su ljudi napravili da bi povukli podatke i sačuvali ih.

Za detaljnije objašnjenje, pogledajte zabilješke iz 2019. godine [Backups](https://missing.csail.mit.edu/2019/backups).

### APIs

Dosta smo pričali u ovoj lekciji o efikasnijem korišćenju vašeg kompjutera da bi izvršili _lokalne_ zadatke, ali ćete vidjeti da se većina ovih lekcija proširuje i na internet. Većina online servisa će imati "APIs" koji će vam omogućiti da programatično pristupite njihovim podacima. Na primer, US vlada ima API koji vam dopušta da dobijete podatke o vremenskoj prognozi, što možete iskorititi da lako dobijete podatke o vremenskoj prognozi u vašem shell-u.

Većina ovih API-ja ima sličan format. Oni su struktuirani URL-ovi, obično u korijenu kao `api.service.com`, gdje putanja i upit parametri ukazuju koje podatke želite da pročitate ili koju akciju želite da izvedete. Za US podatke o vremenu, na primer, da bi dobile podatke o posebnoj lokaciji, vi pravite GET zahtjev (sa `curl` na primer) na https://api.weather.gov/points/42.3604,-71.094. Sam odgovor sadrži gomilu drugih URL-ova koji vam dopuštaju da dobijete specifičnu prognozu za izabrani region. Obično, odgovor je formatiran kao JSON, koji nakon toga možete pajpovati kroz alate kao što je [jq](https://stedolan.github.io/jq/) da bi dobili ono što želite.

Neki APIs traže autentikaciju, i ovo obično uzima oblik neke vrste tajnog _tokena_ koji morate priključiti zahtjevu. Trebali bi pročitati dokumentaciju za API da bi vidjeli šta poseban servis koji tražite koristi, ali ["OAuth"](https://www.oauth.com/) je protokol koji ćete često vidjeti da se koristi. U suštini, OAuth je način da vam se daju tokeni koji se "ponašaju kao vi" na datom servisu, i koji se mogu koristiti samo za posebne namjene. Imajte u vidu da su ti tokeni _tajni_, i bilo ko, ko dobije pristup vašem tokenu, može da radi šta god token dozvoljava _vašem_ nalogu. 

[IFTTT](https://ifttt.com/) je sajt i usluga koja je centrirana oko ideje APIs - pruža mogućnost integracije sa gomilom usluga, i dopuštaju vam da vežete njihove događaje na skroz proizvoljan način. Provjerite ih!

### Uobičajeni flagovi obrasci komandne linije

Alati komandne linije prilično variraju, često ćete željeti da provjerite njihovu `man` stranicu prije upotrebe. Oni često dijele neke zajedničke funkcije što može biti dobro za znati: 

- Većina alata podržava neku vrstu `--help` flag-a da bi se prikazala kratka instrukcija za alat.
- Mnogi alati koji mogu izazvati neopozive promjene podržavaju notaciju "dry run" u kojoj oni sami ispisuju _šta bi mogli da urade_, ali u stvari ne izvode promjenu. Slično, oni često imaju "interaktivan" flag koji će biti tražen od vas za svaku destruktivnu akciju. 
- Možete obično koristiti `--version` ili `-V` da bi program ispisao svoju verziju (pogodno za prijavljivanje bagova!).
- Gotovo svi alati imaju `--verbose` ili `-v` flag da bi pružili širi output. Možete obično uključiti flag više puta (`-vvv`) da bi dobili _više_ opširnih outputa, koji mogu biti pogodni za ispravljanje grešaka. Slično, mnogi alati imaju `--quiet` flag da bi se ispisalo nešto samo prilikom greške. 
- U mnogim alatima, `-` na mjestu imena fajla znači "standardan input" ili "standardan output", zavisno od argumenta. 
- Potencijalno destruktivni alati uobičajeno nisu rekurzivni, ali podržavaju "rekurzivni" flag (često `-r`) za rekurziju.
- Ponekad, hoćete da proslijedite nešto što _izgleda_ kao flag kao normalan argument. Na primjer, zamislite da želite da uklonite fajl sa nazivom `-r`. Ili želite da pokrenete jedan program "kroz" drugi, kao `ssh machine foo`, i želite da proslijedite flag "unutrašnjem" programu (`foo`). Specijalni argument `--` čini da program _stopira_ procesiranje flagova i opcija (stvari koje počinju sa `--`) sa onim što slijedi, dozvoljavajući vam da proslijedite stvari koje izgledaju kao flagovi a da ne budu tako interpretirani kao što je: `rm -- -r` ili `ssh machine --for-ssh -- foo --for-foo`.

### Menadžeri prozora

Većina vas koristi "prevuci i otpusti" menadzer prozora, kao oni koji dolaze na Windows, macOS, i Ubunutu po defaultu. Postoje prozori koji samo nekako vise na ekranu, i možete ih povlačiti okolo, mijenjati veličinu, i preklapati ih jedan preko drugog. Ali ovo je samo jedna _vrsta_ menadžera prozora, koji se često nazivaju "plutajući" menadzeri prozora. Postoji još mnogo drugih, posebno u Linuxu. Posebno česta alternativa je "tiling" menadzer prozora. U tiling menadzeru prozora, prozori se nikada ne poklapaju, i umjesto toga su uređeni kao pločice na vašem ekranu, slično kao okna u tmux-u. Sa tiling menadzerom prozora, ekran je uvijek popunjen sa svim prozorima koji su otvoreni, uređeni u skladu sa nekim _izgledom_. Ukoliko imate samo jedan prozor, zauzima čitav ekran. Ukoliko otvorite još jedan, originalan prozor se smanjuje da bi napravio prostor za njega (obično nešto kao 2/3 i 1/3). Ukoliko otvorite treći, ostali prozori će se isto smanjiti da bi napravili mjesta za novi prostor. Isto kao sa oknima u tmux-u, možete se kretati oko ovih popločanih prozora koristeći vašu tastaturu, i možete im promijeniti veličinu i pomjerati ih okolo, sve to bez diranja miša. Vrijedi pogledati ih!

### VPNs

VPN-ovi bjesne ovih dana, ali ne jasno da li postoji [dobar razlog za to](https://gist.github.com/joepie91/5a9909939e6ce7d09e29). Trebali bi da budete svjesni šta vam VPN donosi i šta vam ne donosi. VPN je, u najboljem slučaju, _samo_ način za vas da promijenite provajdera internet usluga što se samog interneta tiče. Sav vaš saobraćaj će izgledati kao da dolazi od VPN provajdera umejsto sa vaše "prave" adrese, i network na koji ste konektovani će samo vidjeti enrkitpovani sadržaj.

Dok to može izgledati atraktivno, imajte u vidu da kada koristite VPN, sve što vi zapravo radite je pomijeranje vašeg povjerenja od vašeg trenutnog ISP-a do VPN hosting kompanije. Sve što bi vaš ISP _mogao_ da vidi, VPN provajder vidi _umjesto njega_. Ukoliko im vjerujete _više_ nego vašem ISP, onda je to pobjeda, ali mimo toga, nije jasno da li ste nešto dobili. Ukoliko ste na nekom nejasnom nešifrovanom javnom Wi-Fi na aerodromu, onda možda ne vjerujete dovoljno konekciji, ali kada ste kući, zamjena nije toliko jasna.

Takođe bi trebalo da znate da ovih dana, većina vašeg saobraćaja, koja je makar osjetljive prirode, je _već_ enkriptovana kroz HTTPS ili TLS šire posmatrano. U tom slučaju, obično malo znači da li ste na "lošem" networku ili ne - network operator će samo vidjeti sa kojim serverima komunicirate, ali ne i bilo šta od podataka koji su razmijenjeni.

Imajte u vidu da sam rekao "u najboljem slučaju" iznad. Nije da se ne dešava da VPN provajderi slučajno pogrešno podese njihov softver tako da je enkripcija ili slaba ili totalno onemogućena. Neki VPN provajderi su maliciozni (ili najblaže rečeno oportunisti, i zabilježiće sav vaš saobraćaj, i moguće prodati informacije trećim stranama. Izbor lošeg VPN provajdera je često lošije nego da se uopšte ne koristi. 

U nevolji, MIT [pokreće VPN](https://ist.mit.edu/vpn) za studente, tako da će možda značiti da pogledate. Isto, ukoliko se sami snalazite, pogledajte [WireGuard](https://www.wireguard.com/).

### Markdown

Postoji velika šansa da ćete pisati neki tekst u toku vaše karijere. I često, željećete da obilježite taj tekst na jednostavan način. Želite da neki tekst bude boldovan ili italic, ili želite da dodate zaglavlja, linkove, i fragmente koda. Umjesto da izvlačite teški alat kao što je Word ili LaTeX, mogli biste razmotriti korišćenje laganog markup jezika [Markdown](https://commonmark.org/help/).

Vjerovatno ste već vidjeli Markdown, ili makar neke njegove varijante. Podskupovi se koriste i podržani su gotovo svuda, čak iako nisu pod imenom Markdown. U suštini, Markdown je pokušaj da se kodifikuje način na koji ljudi već obilježavaju tekst kada pišu čiste tekst dokumente. Isticanje (*italics*) se dodaje tako što se riječ okruži sa `*`. Jako isticanje (**bold**) se dodaje koristeći `**`. Linije koje počinju sa `#` su naslovi (A broj `#` je nivo podnaslova). Svaka linija koja počinje sa `-` je stavka liste, a bilo koja linija koja počinje sa brojem + `.` je numerisana stavka liste. Backtick se koristi da bi se riječi prikazale u `kod fontu`, blok koda se može unijeti uvlačenjem linije sa četiri razmaka ili okruživanjem sa tri backtick-a.

```
code goes here
```

Da bi dodali link, stavite _text_ linka u uglastim zagradama, i URL koji ga odmah prati u zagradama: [name](url). Lako je početi sa Markdown-om, i možete ga koristiti gotovo svuda. U stvari, zabilješke za ovu lekciju, kao i za sve ostale, su napisane u Markdown-u, i čisti Markdown možete vidjeti [ovdje](https://raw.githubusercontent.com/missing-semester/missing-semester/master/_2020/potpourri.md).

### Hammerspoon desktop automatizacija na macOS

[Hammerspoon](https://www.hammerspoon.org/) je desktop framework za automatizaciju za macOS. Dopušta vam da pišete Lua skriptu koja se kači u funkcionalnost operativnog sistema, dopuštajući vam interakciju sa tastaturom/mišem, prozorom, displejom, sistemom fajlova, i još mnogo stvari. 

Neki primjeri stvari koje možete raditi sa Hammerspoon-om:

- Vezivanje prečica da bi se prozor pomjerio na specifičnu lokaciju
- Kreiranje dugmeta na traci menija koje automatski postavlja prozore u određeni raspored
- Isključivanje zvučnika kada stignete u lab (detekcijom Wi-Fi mreže)
- Pokazivanje upozorenja ukoliko ste greškom uzeli punjač vašeg prijatelja

Na visokom nivou, Hammerspoon vam dopušta da proizvoljno izvršavate Lua kod, vezan za dugme u meniju, pritiskanje tastera, ili događaja, i Hammerspoon pruža opsežnu biblioteku za interakciju sa sistemom, tako da u suštini nema granice za sve što možete da uradite sa njim. Mnogi ljudi su napravili njihove Hammerspoon konfiguracije javnim, tako da generalno možete saznati šta vam treba pretraživanjem na internetu, ali uvijek možete napisati vaš kod od početka.

#### Resursi

- [Počnite sa Hammerspoon-om](https://www.hammerspoon.org/go/)
- [Primjeri konfiguracije](https://github.com/Hammerspoon/hammerspoon/wiki/Sample-Configurations)
- [Anishova Hammerspoon konfiguracija](https://github.com/anishathalye/dotfiles-local/tree/mac/hammerspoon)

### Booting + live USBs

Kada se vaš sistem podigne, prije nego što se operativni sistem učita, [BIOS](https://en.wikipedia.org/wiki/BIOS)/[UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) inicijalizuje sistem. U toku ovog procesa, možete pritisnuti specifičnu kombinaciju tastera da bi podesili ovaj sloj softvera. Na primer, vaš računar može reći nešto kao "Pritisnite F9 da bi podesili BIOS. Pritisnite F12 da bi ušli u boot meni" u toku boot procesa. Možete podesiti razne stvari koje se odnose na podešavanja hardvera u BIOS meniju. Možete takođe ući u boot meni da bi podigli sistem preko alternativnog uređaja umjesto sa vašeg hard diska. 

[Live USBs](https://en.wikipedia.org/wiki/Live_USB) su USB flash drajveri koji sadrže operativni sistem. Možete kreirati jedan od njih preuzimanjem operativnog sistema (npr. Linux distra) i narezivanjem na fleš disk. Ovaj proces je malo komplikovaniji od jednostavnog kopiranja `.iso` fajla na disku. Postoje alati kao što su [UNetbootin](https://unetbootin.github.io/) koji će vam pomoći da kreirate live USBs.

Live USBs su korisni za razne stvari. Osim ostalih stvari, ukoliko dodje do pada vaše trenutne instalacije operativnog sistema, pa se više ne podiže, možete koristiti live USB2 da vratite podatke ili popravite operativni sistem.

### Docker, Vagrant, VMs, Cloud, OpenStack

[Virtuelne mašine](https://en.wikipedia.org/wiki/Virtual_machine) su slični alati kao i kontejneri koji vam dopuštaju da oponašate čitav računarski sistem, uključujući operativni sistem. Ovo može biti korisno za kreiranje izolovanog okruženja za testiranje, razvoj, ili istraživanje (npr. izvršavanje potencijalno malicioznog koda).

[Vagrant](https://www.vagrantup.com/) je alat koji vam dopušta da opišete konfiguraciju mašine (operativni sistem, servise, pakete itd.) u kodu, a zatim instancirate VMs sa jednostavnim `vagrant up`. [Docker](https://www.docker.com/) je konceptualno isti ali umjesto toga koristi kontejnere.

Takođe možete iznajmiti virtualnu mašinu u cloudu, i to je dobar način da dobijete pristup:

- Jeftinoj uvijek upaljenoj mašini koja ima javnu IP adresu, koja se koristi kao host za usluge
- Mašinu sa dosta CPU, diska, RAM-a i/ili GPU
- Još puno mašina kojima imate fizički pristup (naplata je često po sekundi, pa ukoliko želite mnogo računara za kratki period vremena, izvodiljivo je iznajmiti 1000 računara za nekoliko minuta)

Popularni servisi uključuju [Amazon AWS](https://aws.amazon.com/), [Google Cloud](https://cloud.google.com/), i [DigitalOcean](https://www.digitalocean.com/).

Ukoliko ste član MIT CSAIL, možete dobiti besplatnu VM za istraživačke svrhe kroz [CSAIL OpenStack instance](https://tig.csail.mit.edu/shared-computing/open-stack/).

### Notebook programiranje

[Notebook okruženje za programiranje](https://en.wikipedia.org/wiki/Notebook_interface) može biti izuzetno pogodno za rad sa nekim vrstama interaktivnog ili istraživačkog razvoja. Možda je trenutno najpopularnije Notebook okruženje za programiranje [Jupyter](https://jupyter.org/), za Python (i nekoliko drugih jezika). [Wolfram Mathematica](https://www.wolfram.com/mathematica/) je još jedno Notebook okruženje za programiranje za rad sa matematički orjentisanim programiranjem.

### GitHub

[GitHub](https://github.com/) je jedna od najpopularnijih platformi za razvoj open-source softvera. Mnogi alati o kojima smo pričali u ovim lekcijama, od [vim-a](https://github.com/vim/vim) to [Hammersppon](https://github.com/Hammerspoon/hammerspoon), su hostovani na GitHub-u. Lako je početi sa doprinosom open-sourca da bi pomogli u poboljšanju alata koje koristite svaki dan.

Postoje dva primarna načina na kojima ljudi doprinose projektima na GitHub-u:

- Kreiranje [issue](https://help.github.com/en/github/managing-your-work-on-github/creating-an-issue). Ovo se može koristiti da bi se prijavili bagovi ili zatražila nova funkcija. Ništa od navedenog ne podrazumijeva čitanje ili pisanje koda, tako da to može biti dosta lako za uraditi. Bag prijave visokog kvaliteta mogu biti veoma vrijedni za programere. Komentarisanje na postojećim raspravama takođe može biti od koristi.
- Prilaganje koda kroz [pull request](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests). Ovo je generalno češće od kreiranja issue. Možete [fork](https://help.github.com/en/github/getting-started-with-github/fork-a-repo) repository na GitHub-u, klonirati vaš fork, napraviti novu granu, napraviti neke izmjene (npr. ispravljanje baga ili implementacija funkcije), push granu, i zatim [kreirati pull request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request). Nakon ovoga, može generalno biti naprijed-nazad sa osobom koja održava projekat, koji će vam pružiti povratnu informaciju za vaš patch. Konačno, ukoliko sve prođe dobro, vaš patch će biti spojen sa glavnim repository-em. Vrlo često, veći projekti će imati vodič za doprinose, tagovati probleme za početnike, a neki čak imaju program mentorstva da bi pomogli onima koji po prvi put doprinose da se upoznaju sa projektom.

