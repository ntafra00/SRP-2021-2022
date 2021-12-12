# MESSAGE AUTHENTICATION AND INTEGRITY

Datum: 09.11.2021. / 07.12.2021.

Cilj treće laboratorijske vježbe bila je primjena teoretske spoznaje o osnovnim kriptografskim mehanizmima za autentikaciju i zaštitu integriteta poruka. Koristili smo simetrične i asimetrične kripto sustave: ***message authentication code (MAC)*** i ***digitalne potpise*** zasnovane na javnim ključevima.


# Uvod

Napravili smo virtualni environment sa svojim imenom. 

Pozicioniravši se u odgovarajući direktorij bili smo spremni za početak vježbe.

# Zadatak 1

Cilj prvog zadatka bila je implementacija zaštite integriteta sadržaja dane poruke primjenom odgovarajućeg MAC algoritma. 

Pri tome smo koristili HMAC mehanizam iz Python biblioteke cryptography

### Rješenje

Kreirali smo datoteku imena message.txt proizvoljnog sadržaja čiji smo integritet htjeli zaštititi.

Na temelju pročitanog sadržaja iz datoteke i proizvoljno unesenog ključa generirali smo MAC pomoću funkcije ***generate_MAC*** te smo ga spremili u file imena message.sig.

Unutar funkcije ***verify_MAC*** smo usporedili novostvoreni MAC s onim kojeg smo poslali kao argument funkcije.

```python
from cryptography.hazmat.primitives import hashes, hmac
from cryptography.exceptions import InvalidSignature

def generate_MAC(key, message):
    if not isinstance(message, bytes):
        message = message.encode()

    h = hmac.HMAC(key, hashes.SHA256())
    h.update(message)
    mac = h.finalize()
    return mac

def verify_MAC(key, mac, message):
    if not isinstance(message, bytes):
        message = message.encode()

    h = hmac.HMAC(key, hashes.SHA256())
    h.update(message)
    try:
        h.verify(mac)
    except InvalidSignature:
        return False
    else:
        return True

if __name__ == "__main__":
    key = b"my very super secret"

    with open("message.txt", "rb") as file:
        message = file.read()

    # mac = generate_MAC(key, message)

    # with open("message.sig", "wb") as file:
    #     file.write(mac)

    with open("message.sig", "rb") as file:
        mac = file.read()

    isAuthentic = verify_MAC(key, mac, message)

    print(isAuthentic)
```

# Zadatak 2

Cilj drugog zadatka bilo je utvrditi vremenski ispravnu sekvenciju transakcija sa odgovarajućim dionicama. Nalozi za pojedine transakcije nalazili su se lokalnom web poslužitelju: 

http://a507-server.local

### Rješenje

Pomoću GNU Wget softvera smo skinuli sve potrebne file-ove sa gore navedenog servera.

Unutar funkcije ***checkIfAuthentic*** smo vršili provjeru je li traženi MAC dobivenom kombinacijom odgovarajuće poruke i ključa. 

U slučaju da je, spremili bismo sadržaj poruke u napravljenu listu valid.

Funkcija ***extractOrderDateTime*** poslužila je kao pomoć pri sortiranju transakcija.

```python
from os import read
from cryptography.hazmat.primitives import hashes, hmac
from cryptography.exceptions import InvalidSignature
import os
from datetime import datetime

def generate_MAC(key, message):

	if not isinstance(message, bytes):
			message = message.encode()

	h = hmac.HMAC(key, hashes.SHA256())
	h.update(message)
	mac = h.finalize()

	return mac

def verify_MAC(key, mac, message):

	if not isinstance(message, bytes):
		message = message.encode()

	h = hmac.HMAC(key, hashes.SHA256())
	h.update(message)
	try:
		h.verify(mac)
	except InvalidSignature:
		return False
	else:
		return True

def extractOrderDateTime(order):

	first = order.index('(')
	second = order.index(')')
	orderDate = order[first+1: second]

	return datetime.strptime(orderDate, "%Y-%m-%dT%H:%M")

def checkIfAuthentic(path, key):

	valid = []

	for ctr in range(1,11):

		msgFile = os.path.join(path, f"order_{ctr}.txt")
		sigFile = os.path.join(path, f"order_{ctr}.sig")

		with open(msgFile, "rb") as file:
			message = file.read()

		with open(sigFile, "rb") as file:
			mac = file.read()

		is_authentic = verify_MAC(key, mac , message)

		if is_authentic:
			valid.append(message.decode("utf-8"))
			valid.sort(key = extractOrderDateTime)

	print(valid)

if __name__ == "__main__":

	key = "tafra_nikola".encode()
	path = os.path.join("challenges","tafra_nikola", "mac_challenge")
	checkIfAuthentic(path,key)
```

# Zadatak 3

U ovome zadatku bavili smo se problemom public-key kriptografije.

Sa servera smo dohvatili dvije slike i odgovarajuće digitalne potpise.

Slike je profesor potpisao koristeći svoj privatni ključ.

Na temelju danog javnog ključa kojeg smo pročitali iz datoteke ***public.pem***  određivali smo koja je od dvije slike autentična. 

## Rješenje

```python
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives import hashes
from cryptography.exceptions import InvalidSignature

def load_public_key():
	with open("public.pem", "rb") as f:
		PUBLIC_KEY = serialization.load_pem_public_key(
		f.read(),
		backend=default_backend()
		)
	return PUBLIC_KEY

def verify_signature_rsa(signature, message):
	PUBLIC_KEY = load_public_key()
	try:
		PUBLIC_KEY.verify(
			signature,
			message,
			padding.PSS(
				mgf=padding.MGF1(hashes.SHA256()),
				salt_length=padding.PSS.MAX_LENGTH
			),
			hashes.SHA256()
		)
	except InvalidSignature:
		return False
	else:
		return True

if __name__ == "__main__":

	# Reading from a file
	with open("image_2.sig", "rb") as file:
		signature = file.read()

	with open("image_2.png", "rb") as file:
		image = file.read()

	is_authentic = verify_signature_rsa(signature, image)
	print(is_authentic)
```