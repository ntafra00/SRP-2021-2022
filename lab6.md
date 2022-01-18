# LINUX PERMISSIONS AND ACLS

Datum: 11.1.2022.

U sklopu posljednje laboratorijske vježbe upoznali smo se s osnovnim postupkom upravljanja korisničkim računima na Linux OS-u. Poseban naglasak stavili smo na **kontrolu pristupa** datotekama, programima i drugim resursima Linux sustava.

# Kreiranje korisnika

Pokrenuli smo WSL i izvođenjem naredbe 

```bash
id 
```

dobili uvid u naš UID(user ID) i GID (group ID). Svakom korisniku pridjeljen je jedinstveni UID i mora biti pripadnik barem jedne grupe. (GID).

Idući zadatak bio je kreiranje novih korisnika. Kreirali smo korisnike *alice4* i *bob4* sljedećim naredbama:

```bash
sudo adduser alice4
sudo adduser bob4
```

Iz prethodnih naredbi vidimo da je kreiranje novih korisnika moguće isključivo od strane *super user-a.* Prilikom kreacije unijeli smo lozinke na potrebnim mjestima i korisnici su stvoreni.

Logirali smo se u novostvorene korisnike i potvrdili da je sve uspješno prošlo.

```bash
su - alice4
su - bob4
```

Oba korisnika pripadaju samo jednoj grupi, alice4, odnosno bob4. 

# Kreiranje datoteka

Nakon uspješnog logiranja kao Alice, zadatak je bio kreiranja novog foldera imena *srp* te unutar tog foldera kreacija *file-a* imena *security.txt.*

```bash
mkdir srp
echo “Hello World” > security.txt
```

Naredbama

```bash
ls -l srp
ls -l srp/security.txt

getfacl srp
getfacl srp/security.txt
```

dobili smo uvid u vlasnike resursa i dopuštenja definirana nad njima.

### Koja je razlika između r, w, x kod file-a i folder-a?

FILE:

- r (read) → čitanje file-a
- w (write) → pisanje u file
- x (execute) → izvršavanje file-a

FOLDER:

- r (read) → uvid u sadržaj folder-a
- w (write) → kreiranje novih stvari unutar foldera
- x (execute) → pozicioniranje unutar folder-a

Izvršavanjem naredbe

```bash
chmod u-r security.txt
```

oduzeli smo pravo čitanja *file-a* vlasniku istog.

U dopunskom terminalu smo se logirali kao Bob i provjerili imamo li pristup file-u security.txt. 

```bash
cat /home/alice4/srp/security.txt
```

Bob je korisnik koji pripada grupi *others.* Navedena grupa je imala pravo čitanja *file-a* stoga je prethodna naredba uspješno izvedena.

### Kako bismo isto mogli napraviti bez oduzimanja dopuštenja nad datotekom?

Napravit ćemo modifikacije na *folder-u* unutar kojeg se nalazi *file security.txt.*

Ovisno o tome gdje se trenutno nalazimo, ispravnim pozicioniranjem unutar folder-a srp mogli bismo izvršiti sljedeće:

```bash
chmod u-x .
```

Bob, korisnik koji pripada grupi *others,* i dalje ima pravo čitanja navedenog *file-a*. Bizarna je situacija u kojoj vlasnik *file-a* nema pravo čitanja nad istim, dok neki drugi korisnik iz grupe *others* ima. Alice je izvorna prava dobila naredbom

```bash
chmod u+x  .
```

### Kako Bobu onemogućiti čitanje *file-a*?

Da bismo Bobu onemogućili čitanje file-a, morali smo grupi kojoj Bob pripada oduzeti određena prava.

```bash
chmod o-r security.txt
```

Bob trenutno nema pristup *file-u security.txt.*

### Kako da samo Bob sada dobije prava, a ostali korisnici ne?

Boba ćemo dodati u grupu kojoj pripada Alice i dobit će sva prava nad file-om.

```bash
usermod -aG alice4 bob4
```

### Imaju li naši korisnici pristup /etc/shadow folderu?

Izlistom svih prava definiranih nad navedenim folderom vidjeli smo da Bob, odnosno Alice, ne pripadaju grupi shadow što znači da nemaju pristup navedenom folderu.

# Kontrola pristupa korištenjem *Access Control Lists (ACL)*

Sada ćemo koristiti *access control liste* kako bismo omogućili/onemogućili kontrolu pristupa.

Za inspekciju i modifikaciju ACL-ova resursa koristimo programe getfacl i setfacl.

Boba smo u ACL datoteke *security.txt* dodali sljedećom naredbom 

`setfacl -m u:bob:r /home/alice4/srp/security.txt`

Bob će nakon ove naredbe imati pravo čitanja file-a.

Ajmo sada napraviti neku novu grupu i nju dodati u ACL datoteke *security.txt*. Kreirali smo grupu *alice_reading_group* i nju dodali u ACL sljedećom naredbom:

```bash
sudo setfacl -m g:alice_reading_group:r /home/alice4/srp/security.txt
```

Na ovaj način smo sebi olakšali posao jer je sada potrebno samo *user-a* dodati u neku grupu da bi mogao obavljati određene operacije na *file-om/folder-om*.

# Linux procesi i kontrola pristupa

Svaki linux proces u izvršavanju ima svoj jedinstveni identifikator, *process identifier* PID. Osim toga, svakom od procesa se pridijeli id trenutno logiranog *user-a*, UID. Na temelju UID-ja Kernel će odlučivati ima li proces pristup određenim resursima ili ne.

Za kraj smo pripremili python skriptu sa sljedećim kodom:

```python
import os

print('Real (R), effective (E) and saved (S) UIDs:')
print(os.getresuid())

with open('/home/alice/srp/security.txt', 'r') as f:
    print(f.read())
```

Izvršavanjem skripte dobili smo *permission denied*. Zašto je to tako?

→ trenutno logirani *user* s kojim smo izvršili skriptu nema nikakva prava nad *file-om*.

Proces je privremeno dobio UID 0 kad smo skriptu pokrenuli sa sudo i tada smo vidjeli da root user ima prava nad svim stvarima u sustavu.

Probali smo file pokrenuti i kao user Bob, ali tada nije bilo nikakvih problema zbog toga što Bob ima prava nad tim *file-om*.

Ako mi kao Bob izvršimo *passwd*, otvorit će nam se mogućnost promjene lozinke. Ali vidjeli smo da Bob nema pristup */etc/shadow* folderu. Kako će on onda promjeniti?

Postoji poseban flag koji nam omogućava da se effective UID uzme od vlasnika tog *file-a* i da se sukladno tome izvrši promjena lozinke. Izvršavanjem naredbe `ps -eo pid,ruid,euid,suid,cmd`  u drugom terminalu dobili smo uvid u sve tekuće procese. Vidjeli smo da je RUID odgovara Bob-ovom dok je EUID onaj od *super user-a*.