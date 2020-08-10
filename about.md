Tokom tradicionalne nastave računarskih nauka, šanse su da ćete imati puno lekicja koje će vas učiti o naprednim temama CS-a, sve od operativnih sistema preko programskih jezika do machine learning-a. Ali u mnogim institucijama postoji jedna ključna tema koja se rijetko obrađuje, i umjesto toga se prepušta studentima da sami uče o njoj: pismenost u računarskom ekosistemu.

Tokom godina, pomagali smo u predavanjima nekoliko predmeta na MIT-u, i iznova smo vidjeli da studenti imaju ograničeno znanje kada su u pitanju alati koji su im dostupni. Računari su napravljeni da bi automatizovali zadatke koji su se obavljali ručno, ipak studenti često ručno izvršavaju zadatke koji se ponavljaju ili ne uspijevaju da iskoriste puni potencijal moćnih alata kao što su kontrola verzije i editori teksta. U najboljem slučaju, ovo ima za rezultat neefektivnost i gubljenje vremena; u najgorem slučaju, ovo ima za rezultat probleme kao što je gubljenje podataka ili nemogućnost da se završi određeni zadatak.

Ove teme se ne uče kao dio univerzitetskog nastavnog plana i programa: studentima nikada nije pokazano kako da koriste ove alate, ili makar kako da ih koriste efikasno, i dodatno gube vrijeme i trud na zadatke koji bi _trebalo_ da budu jednostavni. U Standardnom CS nastavnom planu i programu nedostaju ključne teme o računarskom ekosistemu koje bi mogle da studentima značajno olakšaju život.

## CS semestar vašeg obrazovanja koji vam nedostaje

Da bi se ovo riješilo, mi vodimo lekcije koje pokrivaju sve teme za koje smatramo da su ključne da bi bili efikasan softverski inženjer i programer. Lekcije su pragmatične i pružaju praktičan uvod u alate i tehnike koje možete odmah primijeniti u širokom spektru situacija na koje ćete naići. Nastava se vodi tokom MIT-ovog "Nezavisnog Perioda Aktivnosti" u Januaru 2020 - jednomjesečni semestar koji se sadrži od kraćih lekcija koje predavaju studenti. Dok su same lekcije samo dostupne MIT studentima, pružićemo sve materijale lekcija zajedno sa video zapisima lekcija javnosti.

Ukoliko ovo zvuči kao da je za vas, evo nekih konkretnih primjera o čemu će se predavati u ovim lekcijama: 

## Komandna linija

Kako da automatizujete česte zadatke koji se ponavljaju sa alijasima, skriptama, i gradnjom sistema. Nema više copy-paste komandi iz tekst dokumenta. Nema više "pokrenite ovih 15 komandi jednu za drugom". Nema više "zaboravili ste da pokrenete ovu stvar" ili "zaboravili ste da proslijedite ovaj argument".

Na primer, brzo pretraživanje kroz istoriju može sačuvati puno vremena. U primjeru ispod pokazećemo nekoliko trikova koji se odnose na kretanje kroz shell istoriju za `convert` komande. 

[History](https://missing.csail.mit.edu/static/media/demos/history.mp4)

## Kontrola verzije
  
 Kako da _ispravno_ koristite kontrolu verzije, i iskoristite njenu prednost da bi izbjegli katastrofu, sarađivali sa drugima, i brzo pronašli i izolovali problematične promjene. Nema više `rm -rf; git clone`. Nema više konflikata pri spajanju (ili ih makar ima manje). Nema više velikih blokova koda sa komentarima. Nema više muke oko toga da pronađete grešku u kodu. Nema više "O ne, da li smo izbrisali kod koji radi?!". Čak ćemo vas naučiti kako da doprinesete projektima drugih ljudi sa pull requests!
 
 U primjeru ispod mi koristimo `git bisect` da saznamo koji commit je probio test jedinice i zatim to popravljamo koristeći `git revert`. 
 
 [Git](https://missing.csail.mit.edu/static/media/demos/git.mp4)
 
 ## Editovanje teksta
 
 Kako da efektivno editujete fajlove iz komandne linije, lokalno i udaljeno, i iskoristite prednost naprednih funkcija editora. Nema više kopiranja fajlova naprijed i nazad. Nema više ponavljajućeg editovanja fajlova. 
 
 Vim makroi su jedni od najboljih funkcija, i u primjeru ispod mi brzo konvertujemo html tabelu u csv format koristeći ugnježdene vim makroe. 
 
 [Vim](https://missing.csail.mit.edu/static/media/demos/vim.mp4)
 
 ## Udaljene mašine
 
 Kako da ostanete prisebni kada radite sa udaljenim mašinama koristeći SSH ključeve i multiplexere terminala. Nema više držanja velikog broja terminala otvorenih samo da bi pokrenuli dvije komande u isto vrijeme. Nema više kucanja vaše lozinke svaki put kada se konektujete. Nema više gubljenja svega samo zato što se internet diskonektovao ili ste morati da uradite reboot laptopa.
 
 U primjeru ispod mi koristimo `tmux` da održimo živim sesije na udaljenim serverima i `mosh` da podržimo network roaming i diskonekciju.
 
 [SSH](https://missing.csail.mit.edu/static/media/demos/ssh.mp4)
 
 ## Pretraga fajlova
 
 Kako da brzo pronađete fajlove koje tražite. Nema više klikanja kroz fajlove u vašem projektu sve dok ne pronađete onaj fajl koji ima kod koji ste tražili.
 
 U primjeru ispod mi brzo tražimo fajlove sa `fd` i isječke koda sa `rg`. Takođe brzo izvršavamo `cd` i `vim` skoriji/česti fajlovi/folderi koristeći `fasd`.
 
 [Find](https://missing.csail.mit.edu/static/media/demos/find.mp4)
 
 ## Upravljanje podacima
 
 Kako da brzo i jednostavno podesite, pregledate, proslijedite, plotujete, i računate podatke i fajlove direktno iz komandne linije. Nema više copy-paste iz log fajlova. Nema više ručnog računanja statistike nad podacima. Nema više plottinga tabele.
 
 ## Virtuelne mašine
 
 Kako da koristite virtuelne mašine i isprobate nove operativne sisteme, izolujete irelevantne projekte, i održavate vašu glavnu mašinu čistom i urednom. Nema više slučajnog korumpiranja vašeg računara dok izvršavate bezbjednosno testiranje. Nema više miliona slučajno instaliranih paketa sa različitim verzijama.
 
 ## Bezbjednost
 
 Kako da budete na internetu a da odmah ne otkrijete sve vaše tajne svijetu. Nema više izmišljanja lozinki koje zadovoljavaju sulude kriterijume. Nema više nesigurnih, otvorenih WiFi network-a. Nema više ne-enkriptovanih poruka.
 
 ## Zaključak
 
Ovo i još puno toga, će biti pokriveno kroz 12 lekcija, a svaka od njih uključuje vježbu za vas da bi se bolje upoznali sa alatima koje koristite. Ukoliko ne možete da čekate Januar, možete takođe pogledati lekicje iz [Hacker Tools](https://hacker-tools.github.io/lectures/), koje smo predavali tokom "Nezavisnog Perioda Aktivnosti" iz prošle godine. To je preteča ovih predavanja, i pokriva mnogo istih tema.

Nadamo se da se vidimo u Januaru, bilo virtuelno ili uživo.

Happy hacking, 

Anish, Jose, i Jon
