# Bezbjednost i Kriptografija

<a href="http://www.youtube.com/watch?feature=player_embedded&v=tjwobAmnKTo
" target="_blank"><img src="" 
alt="Lecture 9: Security and Cryptography (2020)" width="240" height="180" border="10" /></a>

Prošlogodišnja [lekcija o bezbjednosti i privatnosti](https://missing.csail.mit.edu/2019/security/) se fokusirala na to kako možete biti bezbjedniji kao _korisnik_ računara. Ove godine, fokusiraćemo se na bezbjednost i koncepte kriptografije koji su relevantni u razumijevanju alata koji su pokriveni u ranijim lekcijama, kao što su hash funkcije u Git-u, ili funkcija za izvoženje ključeva i simetrični/asimetrični kriptosistemi u SSH.

Ova lekcija nije zamjena za rigoroznije i kompletnije kurseve zaštite računarskih sistema ([6858](https://css.csail.mit.edu/6.858/2020/)) ili kriptografije ([6857](https://courses.csail.mit.edu/6.857/2020/) i 6875). Nemojte se baviti bezbjednosnim poslom bez formalne obuke o bezbjednosti. Osim ako ste ekspert, nemojte [roll your own crypto](https://www.schneier.com/blog/archives/2015/05/amateurs_produc.html). Isti principi važe za bezbjednost sistema.

Ova lekcija ima veoma informativnu (ali mi razmišljamo praktično) primjenu osnovnih kriptografskih koncepata. Ova lekcija neće biti dovoljna da bi vas naučila _dizajn_ bezbjednosnih sistema ili kriptografskih protokola, ali se nadamo da će biti dovoljna da vam pruži opšte razumijevanje programa i protokola koje vi već koristite.

## Entropija

[Entropija](https://en.wikipedia.org/wiki/Entropy_(information_theory)) je mjera slučajnosti. Ovo je korisno, na primer, kada određujete jačinu lozinke.

![img1][img1]

[img1]: https://imgs.xkcd.com/comics/password_strength.png

Kao što [XKCD comic](https://xkcd.com/936/) iznad ilustruje, lozinke kao što su "correcthorsebatterystaple" je sigurnija u odnosu na "Tr0ub4dor&3". Ali kako mjerite nešto ovog tipa?

Entropija se mjeri u _bitovima_, i kada izaberete jednoobrazan slučajan odabir iz niza mogućih rezultata, entropija je jednaka `log_2(# of possibilities)`. Fer bacanje novčića daje 1 bit entropije. Bacanje kockice (sa 6 strana) ima približno ~2.58 bitova entropije.

Trebali bi da uzmete u obzir da napadač zna _model_ lozinke, ali ne i slučajnosti (npr. iz [dice rolls](https://en.wikipedia.org/wiki/Diceware) koje se koriste da bi se selektovala posebna lozinka.

Koliko bitova entropije je dovoljno? Zavisi od vašeg modela prijetnje. Za online nagađanje, kao što XKCD comic iznosi, ~40 bitova entropije je prilično. Da bi bili otporni na offline nagađanje, jača lozinka će biti neophodna (npr. 80 bitova, ili više).

## Hash funkcije

[Kriptografske hash funkcije](https://en.wikipedia.org/wiki/Cryptographic_hash_function) mapiraju podatke proizvoljne veličine u fiksnu veličinu, i imaju neka specijalna svojstva. Slijedi gruba specifikacija hash funkcije:

```shell
hash(value: array<byte>) -> vector<byte, N>  (for some fixed N)
```

Primjer hash funkcije je [SHA1](https://en.wikipedia.org/wiki/SHA-1), koji se koristi u Git-u. Mapira proizvoljnu veličinu inputa na 160-bit outputa (što može biti predstavljeno kao 40 heksadecimalnih karaktera). Možemo isprobati SHA1 hash na inputu koristeći `sha1sum` komandu:

```shell
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'Hello' | sha1sum 
f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0
```

Na visokom nivou, hash funkcija se može posmatrati kao teško inverzivna slučajnog izgleda (ali deterministička) funkcija (i ovo je [idealni model hash funkcije](https://en.wikipedia.org/wiki/Random_oracle)). Hash funkcija ima sledeća svojstva:

- Determinizam - isti input uvijek generiše isti output.
- Ne-inverzivnost - teško je pronaći input `m` kao što je `hash(m) = h` za neki željeni output `h`.
- Ciljano otporna na udar - ukoliko je dat input `m_1`, teško je pronaći drugačiji input `m_2` tako da je `hash(m_1) = hash(m_2)`.
- Otporna na sudar - teško je pronaći dva inputa `m_1` i `m_2` tako da je `hash(m_1) = hash(m_2)` (imajte u vidu da je ovo mnogo jače svojstvo u odnosu na ciljani otpor na udar)

Napomena: Iako može funkcionisati za određene svrhe, SHA-1 se više [ne smatra](https://shattered.io/) jakom kriptografskom hash funkcijom. Tabela [životnog vijeka hash funkcija](https://valerieaurora.org/hash.html) vam može biti interesantna. Ipak, imajte u vidu preporuku da su preporučene specifične hash funkcije izvan opsega ove lekcije. Ukoliko se bavite poslom gdje je ovo bitno, potrebna vam je formalna obuka u bezbjednosti/kriptografiji.

### Aplikacije

- Git, za sadržajno-adresirano skladište. Ideja [hash funkcije](https://en.wikipedia.org/wiki/Hash_function) je više opšti koncept (postoje ne-kriptografske hash funkcije). Zašto Git koristi kriptografsku hash funkciju?
- Kratak rezime sadržaja datoteke. Softver može često biti preuzet iz (potencijalno manje pouzdanih) mirrors, npr. Linux ISO's, i bilo bi lijepo da im ne moramo vjerovati. Zvanični sajtovi obično postavljaju hashove zajedno sa linkom za preuzimanjem (to upućuje na mirror treće strane), tako da hash može biti provjeren nakon preuzimanja fajla.
- [Commitment schemes](https://en.wikipedia.org/wiki/Commitment_scheme). Pretpostavimo da želite da commitujete određenoj vrijednosti, ali da otkrijete samu vrijednost kasnije. Na primer, ukoliko želim "u svojoj glavi" da uradim fer bacanje novčića bez provjerenog zajedničkog novčića koji dvije strane mogu da vide. Mogao bih da izaberem vrijednost `r=random()`, i onda da podijelim `h = sha256(r)`. Zatim, mogli bi da izaberete glavu ili pismo (čak smo se i usaglasili da parno `r` znači glava, i neparno `r` znači pismo). Nakon vašeg izbora, mogao bih da otkrijem moju vrijednost `r`, a vi bi mogli da potvrdite da nisam varao provjeravajući da li se `sha256(r)` poklapa sa hashom koji sam ranije podijelio.

## Derivacione funkcije ključa

Povezani koncept sa kriptografskim hash-evima, [derivacione funkcije ključa](https://en.wikipedia.org/wiki/Key_derivation_function) (KDFs) se koriste za više aplikacija, uključujući stvaranje outputa fiksne dužine da bi se koristili kao ključevi u drugim kriptografskim algoritmima. Obično, KDF su namjerno spori, u cilju usporavanja offline napada brutalnom silom. 

### Aplikacije

- Stvaranje ključeva iz fraza da bi se koristili u drugim kriptografskim algoritmima (npr. simetrična kriptografija, pogledajte ispod).
- Čuvanje login kredencijala. Čuvanje lozinki u čistom tekstu je loše: pravi pristup je generisanje i čuvanje random [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) `salt = random()` za svakog korisnika, čuvanje `KDF(password + salt)`, i verifikovanje pokušaja prijave ponovnim izračunavanjem KDF-a s obzirom na unešenu lozinku i uskadišteni salt.

## Simetrična kriptografija

Skrivanje sadržaja poruke je vjerovatno prvi koncept koji vam padne na pamet kada razmišljate o kriptografiji. Simetrična kriptografija ovo izvršava sa sledećim setom funkcionalnosti:

```shell
keygen() -> key  (this function is randomized)

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```
Šifrovana funkcija ima svojstvo koje je dao output (ciphertext), teško je utvrditi input (plaintext) bez ključa. Dekriptovana funkcija ima očigledna svojstva za ispravljanje, tako da je `decrypt(encrypt(m, k), k) = m`.

Primjer simetričnih kriptosistema koji se široko koristi danas je [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard).

### Aplikacije

- Enkriptovani fajlovi za skladištenje u cloud servisu kojem ne vjerujete. Ovo može biti u kombinaciji sa KDF-om, tako da možete enkriptovati fajl sa frazom. Generišite `key = KDF(passphrase`, a zatim čuvajte `encrypt(file, key)`.

## Asimetrična kriptografija

Termin "asimetrična" odnosi se na postojanje dva ključa, sa dvije različite uloge. Privatni ključ, kao što mu samo ime kaže, znači da je napravljen da bi bio privatan, dok se javni ključ može javno dijeliti i to neće uticati na bezbjednost (za razliku od dijeljenja ključa u simetričnim kriptosistemima). Asimetrični kriptosistemi pružaju sledeći set funkcionalnosti, da enkriptuju/dekriptuju i da potpišu/ovjere:

```shell
keygen() -> (public key, private key)  (this function is randomized)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)

sign(message: array<byte>, private key) -> array<byte>  (the signature)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
```

Encrypt/decrypt funkcije imaju svojstva koja su slična njihovim analozima iz simetričnih kriptosistema. Poruka može biti enkriptovana koristeći _public_ ključ. Uzimajući u obzir output (ciphertext), teško je odrediti input (plaintext) bez _privatnog_ ključa. Decrypt funkcija ima očigledna svojstva ispravljanja, tako da je `decrypt(encrypt(m, public key), private key) = m`.

Simetrična i asimetrična enkripcija se mogu uporediti sa fizičkim zaključavanjem. Simetrični kriptosistem je kao zaključavanje vrata: svako sa ključem može da ih zaključa i otključa. Asimetrična enkripcija je kao katanac sa ključem. Mogli biste nekome dati otključanu bravu (javni ključ), oni bi mogli ostaviti poruku u kutiji i zatim zaključati, i nakon toga, samo vi biste mogli da otvorite bravu jer samo vi imate ključ (privatni ključ).

Sign/Verify funkcije imaju ista svojstva za koja bi se nadali da ih fizički potpisi imaju, sa njima je teško falsifikovati potpis. Bez obzira na poruku, bez _privatnog_ ključa, teško je stvoriti takav potpis pa da `verify(message, signature, public key)` vraća true. I naravno, verify funkcija ima očigledna svojstva za ispravljanje tako da je `verify(message, sign(message, private key), public key) = true`.

### Aplikacije

- [PGP email enkripcija](https://en.wikipedia.org/wiki/Pretty_Good_Privacy). Ljudi mogu njihove javne ključeve objaviti online (npr. na PGP keyserver, ili na [Keybase](https://keybase.io/)). Bilo ko im može poslati enkriptovani mejl.
- Privatno dopisivanje. Aplikacije kao [Signal](https://signal.org/) i [Keybase](https://keybase.io/) koriste asimetrične ključeve da omoguće privatne komunikacione kanale. 
- Signing softver. Git može imati GPG-signed commits i tagove. Sa objavljenim javnim ključem, svako može potvrditi autentičnost preuzetog softvera.

### Distribucija ključeva 

Asimetrična kriptografija ključeva je odlična, ali ima veliki izazov kada je u pitanju distribucija javnih ključeva / mapiranja javnih ključeva identitetima u stvarnom svijetu. Postoji mnogo rešenja za ovaj problem. Signal ima jedno jednostavno rešenje: povjerenje pri prvoj upotrebi, i podrška out-of-band razmjene javnih ključeva (provjeravate "sigurne brojeve" vaših prijatelja uživo). PGP ima različito rešenje, a to je [web of trust](https://en.wikipedia.org/wiki/Web_of_trust). Keybase ima još jedno rešenje [social proof](https://keybase.io/blog/chat-apps-softer-than-tofu) (zajedno sa ostalim dobrim idejama). Svaki model ima svoje prednosti: mi (instruktori) volimo Keybase model. 

## Studije slučaja

### Menadzeri lozinke

Ovo je suštinski alat koji bi svako trebao da pokuša da koristi (npr. [KeePassXC](https://keepassxc.org/)). Mendzeri lozinki vam dopuštaju da koristite jedinstvenu, slučajno generisanu lozinku visoke entropije za sve vaše sajtove, i oni čuvaju sve vaše lozinke na jednom mjestu, enkriptovani sa simetričnom šifrom i ključem dobijenim od fraze koristeći KDF.

Korišćenje menadzera lozinke utiče na izbjegavanje ponovnog korišćenja lozinke (tako da ste manje oštećeni kada sajt postane ugrožen), koriste se lozinke sa velikom entropijom (tako da su vam manje šanse da postanete ugroženi), i samo je potrebno zapamtiti jednu lozinku sa visokom entropijom.

### Dvofaktorna autentikacija

[two-factor authentication](https://en.wikipedia.org/wiki/Multi-factor_authentication) (2FA) zahtjeva od vas da koristite frazu ("nešto što vi znate") zajedno sa 2FA autentikatorom (kao što je [YubiKey](https://www.yubico.com/), "nešto što imate") da bi vas zaštitila od krađe lozinki i [phishing](https://en.wikipedia.org/wiki/Phishing) napada.

### Enkripcija čitavog diska

Enkripcija čitavog diska vašeg laptop-a je lak način da zaštitite vaše podatke u slučaju da je vaš laptop ukraden. Možete koristiti [cryptsetup + LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_a_non-root_file_system) na Linuxu, [BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/) na Windowsu, ili [FireVault] na macOS. Ovo će enkriptovati čitav disk sa simetričnim cipher-om, i sa ključem koji je zaštićen frazom.

### Privatne poruke

Koristite [Signal](https://signal.org/) ili [Keybase](https://keybase.io/). End-to-End bezbjednost je pokrenuta od asimetrične enkripcije ključa. Ovdje je pribavljanje javnog ključa vaših kontakata kritičan korak. Ukoliko želite dobru bezbjednost, morate da autentikujete javne ključeve out-of-band(Sa Signalom ili Keybase-om), ili da vjerujete društvenim dokazima (sa Keybase-om).

## SSH

Pokrili smo upotrebu SSH i SSH ključeva u [ranijoj lekciji](https://missing.csail.mit.edu/2020/command-line/#remote-machines). Hajde da pogledamo kriptografske aspekte ovoga.

Kada pokrenete `ssh-keygen`, on generiše asimetrični par ključeva, `public_key`, `private_key`. Ovo se random generiše, koristeći entropiju koja je omogućena od strane operativnog sistema (prikupljena od hardverskih događaja itd.). Javni ključ se čuva as-is (javan je, pa nije potrebno da se čuva kao tajna), ali na kraju, privatni ključ bi trebao biti enrkiptovan na disku. `ssh-keygen` program od korisnika traži frazu, i ovo je moguće preko derivacione funkcije ključa da bi se napravio ključ, koji se onda koristi da bi se enkriptovao privatni ključ sa simetričnim cipher-om.

U praksi, jednom kada server sazna javni ključ klijenta (sačuvan u `.ssh/authorized_keys` fajlu), klijent koji se povezuje može dokazati njegov identitet koristeći asimetrični potpis. Ovo se obavlja preko [challenge-response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication). Na visokom nivou, server uzima random broj i šalje ga klijentu. Klijent zatim potpisuje tu poruku i šalje potpis nazad na server, koji onda provjerava potpis sa javnim ključem kojeg ima zabilježenog. Ovo efektno dokazuje da je klijent u posjedu privatnog ključa koji odgovara javnom ključu koji je u fajlu `.ssh/authorized_keys` servera, tako da server može dopustiti klijentu da se uloguje.

## Resursi

- [Zabilješke od prošle godine](https://missing.csail.mit.edu/2019/security/): kada je ova lekcija bila više fokusirana na bezbjednost i privatnost korisnika računara
- [Pravi odgovori vezani za kriptografiju](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html): odgovori "koji crypto bih trebao da koristim za x?" za mnoge česte x.

## Vježbe

1. **Entropija**
    - Pretpostavimo da je lozinka izabrana kao spajanje četiri riječi sa malim slovima iz rječnika, gdje je svaka riječ izabrana jednako nasumično iz rečnika od veličine od 100.000. Primjer takve lozinke je `correcthorsebatterystaple`. Koliko bitova entropije on ima?
    - Uzmite u obzir alternativnu šemu gdje je lozinka izabrana kao dio od 8 random alfanumeričkih karaktera (uključujući karaktere i sa malim i sa velikim slovom). Jedan primjer je `rg8Ql34g`. Koliko on ima bitova entropije?
    - Koja je lozinka jača?
    - Pretpostavimo da napadač pokušava da pogodi 10.000 lozinki po sekundi. Prosječno, koliko će mu vremena trebati da probije svaku od lozinki?
2. **Kriptografkse hash funkcije**. Preuzmite Debian sliku iz [mirror](https://www.debian.org/CD/http-ftp/) (npr. [iz ovog Argentinean mirror](http://debian.xfree.com.ar/debian-cd/current/amd64/iso-cd/)). Unaprijed provjerite hash (npr. koristeći `sha256sum` komandu) sa hashom preuzetim sa zvanične Debian stranice (npr. [ovaj fajl](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS) je hostovan na `debian.org`, ukoliko ste preuzeli link fajl sa Argentinean mirror).
3. **Simetrična kriptografija**. Enrkiptujte fajl sa AES enkripcijom, koristeći [OpenSSL](https://www.openssl.org/): `openssl aes-256-cbc -salt -in {input filename} -out {output filename}`. Pogledajte sadržaje koristeći `cat` i `hexdump`. Dekriptujte ga sa `openssl aes-256-cbc -d -in {input filename} -out {output filename}` i potvrdite da se sadržaj poklapa sa originalom koristeći `cmp`.
4. **Asimetrična kriptografija**. 
    - Postavite [SSH keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) na računar kojem imate pristup (ne Athena, jer Kerberos ima čudnu interakciju sa SSH ključevima). Umjesto da koristite RSA ključeve kao u linkovanom tutorijalu, koristite sigurnije [ED25519 keys](https://wiki.archlinux.org/index.php/SSH_keys#Ed25519). Provjerite da li je vaš privatni ključ enkriptovan sa frazom, tako da je zaštićen.
    - [Postavite GPG](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)
    - Pošaljite Anishu enkriptovani mejl ([javni ključ](https://keybase.io/anish)).
    - Potpišite Git commit sa `git commit -S` ili kreirajte potpisani Git tag sa `git tag -s`. Ovjerite potpis commita sa `git show --show-signature` ili sa tagom koristeći `git tag -v`.
