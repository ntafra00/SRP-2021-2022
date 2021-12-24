# ONLINE AND OFFLINE PASSWORD GUESSING ATACK

Datum: 21.12.2021

# ONLINE PASSWORD GUESSING ATACK

Na početku same laboratorijske vježbe instalirali smo alat *nmap* koji nam omogućava uvid u trenutno stanje naše mreže obavljajući *port scanning.*

Range ip adresa odredili smo koristeći naredbu `nmap -v 10.0.15.0/28`

Izvedbom prethodno navedene naredbe dobili smo uvid u sve aktivne hostove i njihove portove u određenom range-u.  Svim aktivnim hostovima nešto je bilo zajedničko, a to je port broja 22 koji pripada ssh-u.

Na adresi [http://a507-server.local/](http://a507-server.local/) nalazili su se svi docker kontejneri među kojima smo morali pronaći onaj s našim imenom.

`ssh tafra_nikola@10.0.15.9`

Jednom kad smo se pomoću *Secure Shell-a* spojili na odgovarajući kontejner, nedostajala nam je samo jedna stvar, a to je lozinka.

Nakon nekoliko neuspjelih pokušaja vidjeli smo da nije podešen nikakav rate limit stoga bismo mogli besprijekorno izvršavati *online napad*.

Uzevši u obzir profesorove hintove oko lozinke koju je potrebno probiti, matematičkom analizom došli smo do odgovora da je nekakva aproksimacija keyspace-a 2^30.

Za pokušaj probijanja tražene lozine koristili smo alat imena *hydra* 

`hydra -l tafra_nikola -x 4:6:a 10.0.15.9 -V -t 4 ssh`

Ako smo dobili da je keyspace otrpilike 2^30, a broj pokušaja po minuti iznosi 64, vrijeme potrebno za doći do lozinke jednostavno je preveliko i nema smisla čekati.

Promjena vrijednosti flaga -t ne bi nam bila puno od pomoći iz razloga što bi tada server bio problem.

Stoga, kao rješenje preostalo nam je koristiti predefined dictionary-je i njih iskoristiti prilikom izvođenja offline napada.

Stoga, kao rješenje nam je preostalo koristiti *predefined dictionary-je* i njih iskoristiti prilikom izvođenja napada.

*Dictionary-je* smo dohvatii sa servera i hydru podesili da sada obavlja testiranje iz traženog *dictionary-ja* a ne iz *keyspace-a*. Sada je potrebno čekati da hydra obavi svoj dio posla i dobivenu lozinku testirati na kontejneru. 

`hydra -l tafra_nikola -P dictionary/g4/dictionary_online.txt 10.0.15.9 -V -t 4 ssh`

Jednom kad je hydra uspješno pronašla lozinku, dobili smo pristup kontejneru. S lokalnog računala spojili smo se na remote računalo.

# OFFLINE PASSWORD GUESSING ATACK

Sada kad smo dobili pristup remote računalu, u interesu nam je bilo doći do lozinke nekog od drugih user-a. Kako bismo to napravili, prvo smo trebali pronaći folder unutar kojeg se nalaze sve hashirane vrijednost lozinki. 

Folderu /etc/shadow pristupili smo koristeći naredbu sudo /etc/shadow

Izabrali smo neku nasumičnu hash vrijednost i spremili je u lokalni file imena hash.txt.  (u mom slučaju to je bila hashirana vrijednost lozinke korisnika Freddie Mercury)

Koristeći alat hashcat i naredbu 

`hashcat --force -m 1800 -a 3 hash.txt ?l?l?l?l?l?l --status --status-timer 10`

došli smo do zaključka da je ovo previše vremena za čekati te ćemo, isto kao i u prvom slučaju, koristiti unaprijed pripremljene *dictionary-je*.

`hashcat --force -m 1800 -a 0 hash.txt dictionary/g4/dictionary_offline.txt --status --status-timer 10`

Izvršavanjem navedene naredbe, jedino što nam je preostalo je čekati da hashat pronađe lozinku i istu iskoristiti na remote računalo.

Jednom kad smo do lozinke došli, uspješno smo se spojili na remote računalo kao neki drugi user. (Freddie Mercury)