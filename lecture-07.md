# Uklanjanje grešaka i profilisanje

<a href="http://www.youtube.com/watch?feature=player_embedded&v=l812pUnKxME
" target="_blank"><img src="" 
alt="Lecture 7: Debugging and Profiling (2020)" width="240" height="180" border="10" /></a>

Zlatno pravilo u programiranju jeste da kod ne radi ono što očekujete od njega, već ono što mo kažete da uradi. Premostiti taj problem ponekad može biti veoma krupan korak. U ovoj lekciji ćemo preći korisne tehnike za nošenje sa kodom koji ima bagove i kojem fale resursi: debugging i profilisanje. 

## Uklanjanje grešaka

### Printf uklanjanje grešaka i logovanje

"Najefikasnije sredstvo za uklanjanje grešaka je i dalje pažljivo razmišljanje, zajedno sa dobro postavljenim izjavama o ispisu" - Brian Kernighan, __Unix for Beginners.__

Prvi pristup da bi se uklonile greške iz programa i dodavanje print izjave okolo dijela u kojem imate problem, i to ponavljate dok ne dobijete dovoljno informacija da shvatite ko je krivac za dati problem.

Drugi pristup je upotreba logovanja u vašem programu, umjesto ad hoc print izjava. Logovanje je bolje u odnosu na print izjave iz više razloga: 

- Možete logovati fajlove, sockets-e, ili čak i udaljene servere umjesto standardnog output-a.
- Logovanje podržava ozbiljne nivoe (kao što su  INFO, DEBUG, WARN, ERROR, i sl.), koji vam omogućuju da prema njima filtrirate output.
- Za nove probleme, postoji dobra šansa da će vaši logovi sadržati dovoljno informacija da shvatite šta nije u redu.

[Ovdje](https://missing.csail.mit.edu/static/files/logger.py) je primjer koda koji loguje poruku: 

```python
$ python logger.py
# Raw output as with just prints
$ python logger.py log
# Log formatted output
$ python logger.py log ERROR
# Print only ERROR levels and above
$ python logger.py color
# Color formatted output
```

Jedan od mojih omiljenih savjeta da bi poboljšali čitljivost logova jeste da obilježite kod bojom. Do sada ste vjerovatno shvatili da vaš terminal koristi boje da učini stvari čitljivijim. Ali, kako to radi? Programi kao što su `ls` ili `grep` koriste [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code), koji su specijalizovani dijelovi karaktera koji pokazuju vašem shell-u da promijeni boju output-a. Na primer, izvršavanje `echo -e "\e[38;2;255;0;0mThis is red\e[0m"` ispisuje poruku `This is red` crvenom bojom na vašem terminalu, dokle god podržava [true color](https://gist.github.com/XVilka/8346728#terminals--true-color). Ako vaš terminal ne podržava ovo (npr. macOS Terminal.app), možete koristiti univerzalnije podržani escape code sa izborom od 16 boja, na primer `echo -e "\e[31;1mThis is red\e[0m"`. 

Sledeća skripta pokazuje kako ispisati mnogo RGB boja u vašem terminalu (opet, sve dok podržavaju true color).

```shell
#!/usr/bin/env bash
for R in $(seq 0 20 255); do
    for G in $(seq 0 20 255); do
        for B in $(seq 0 20 255); do
            printf "\e[38;2;${R};${G};${B}m█\e[0m";
        done
    done
done
```

### Logovi trećih strana

Kako počinjete da pravite šire softverske sisteme vjerovatno ćete naići na zavisnosti koje se pokreću kao poseban program. Veb serveri, baze podataka ili brokeri poruka su česti primjeri ovakvih vrsta zavisnosti. Kada imate interakciju sa ovim sistemima često je neophodno pročitati njihove logove, jer error poruka na klijent strani možda neće biti dovoljna.

Srećom, većina programa piše svoje logove negdje u vašem sistemu. U UNIX sistemima, uobičajeno je da programi pišu svoje logove u `var/log`. Na primer, [NGINX](https://www.nginx.com/) webserveri ih čuvaju u `/var/log/nginx`. U skorije vrijeme, sistemi su počeli da koriste **system log**, što je sve više mjesto gdje vaše log poruke idu. Većina (ali ne i svi) Linux sistemi koriste `systemd`, sistem daemon koji kontroliše mnoge stvari u vašem sistemu kao npr. koje usluge su omogućene i pokrenute. `systemd` stavlja logove u `/var/log/journal` u posebnom formatu i možete koristiti [journalctl](https://www.man7.org/linux/man-pages/man1/journalctl.1.html) komandu da prikažete poruke. Slično, na macOS takođe postoji `/var/log/system.log` ali povećan broj alata koristi system log, koji može biti prikazan sa [log show](https://www.manpagez.com/man/1/log/). Na većini UNIX sistema takođe možete koristiti [dmesg](https://www.man7.org/linux/man-pages/man1/dmesg.1.html) komandu da pristupite logu kernela.

Za logovanje u sistem logs možete koristiti [logger](https://www.man7.org/linux/man-pages/man1/logger.1.html) shell program. Evo primjera korišćenja `logger`-a i kako da provjerite da li je unos stigao do sistem logova. Štaviše, većina programskih jezika ima vezivanje logginga za sistem log.

```shell
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

