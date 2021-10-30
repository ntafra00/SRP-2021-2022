# SYMMETRIC KEY CRYPTOGRAPHY - A CRYPTO CHALLENGE

DATUM: 26.10.2021.

U sklopu druge laboratorijske vježbe cilj je bio riješiti odgovarajući crypto izazov, odnosno dešifrirati ciphertext u kontekstu simetrične kriptografije. 

# Uvod

Na samom početku laboratorijske vježbe kreirali smo virtualni environment imena lab2.

```bash
python -m venv lab2 
```

Pozicioniravši se u odgovarajući direktorij pokrenuli smo skritpu komandom activate.

Za izvođenje samog izazova bilo je nužno korištenje Python biblioteke *cryptography* stoga je idući korak bilo instaliranje iste.

```bash
pip install cryptography
```

Plaintext kojeg smo otkrivali bio je enkriptiran pomoću high-level sustava za simetričnu enkripciju iz navedene biblioteke - *Fernet.*

Upoznali smo se s osnovnim funkcionalnostima Ferneta:

**Generiranje ključa:**

```python
key = Fernet.generate_key()
```

**Inicijalizacija fernet sustava pomoću dobivenog ključa:**

```python
f = Fernet(key)
```

**Enkripcija proizvoljne varijable *plaintext:***

```python
ciphertext = f.encrypt(plaintext)
```

**Dekripcija varijable ciphertext:**

```python
deciphertext = f.decrypt(ciphertext)
```

# Početak izazova

Nakon što smo se upoznali sa osnovama, vrijeme je za početak rada.

Prvi zadatak bio je pronalazak file-a s našim imenom koji se nalazio unutar foldera sa mnoštvo drugih enkriptiranih file-ova. Imena su enkriptirana koristeći SHA-256 algoritam.

```python
from cryptography.hazmat.primitives import hashes

def hash(input):
    if not isinstance(input, bytes):
        input = input.encode()

    digest = hashes.Hash(hashes.SHA256())
    digest.update(input)
    hash = digest.finalize()

    return hash.hex()

filename = hash('prezime_ime') + ".encrypted"
```

Izvođenjem gore navedenog programa unutar varijable filename dobili smo enkriptiranu verziju našeg imena te smo preuzeli odgovarajući file.

Korišteni ključevi bili su ograničene entropije - 22 bita. 

Pomoću beskonačne petlje smo krenuli prolaziti kroz sve ključeve dok nije pronađen odgovarajući ključ. Kao pomoć pri pronalasku ključa poslužila nam je činjenica da svaki PNG file ima identičnih početnih 8 bajtova - `\211 P N G \r \n \032 \n`

```python
import base64
from cryptography.hazmat.primitives import hashes
from cryptography.fernet import Fernet

def hash(input):
	if not isinstance(input, bytes):
		input = input.encode()
	digest = hashes.Hash(hashes.SHA256())
	digest.update(input)
	hash = digest.finalize()
	return hash.hex()

def test_png(header):
	if header.startswith(b"\211PNG\r\n\032\n"):
		return True

def brute_force():
# Reading from a file
	filename = "c03e5400a3728c9149c525ea48b6499238a2a5565540f96f23c4409b5865054f.encrypted"
	with open(filename, "rb") as file:
		ciphertext = file.read()
# Now do something with the ciphertext
	ctr = 0
	while True:
		key_bytes = ctr.to_bytes(32, "big")
		key = base64.urlsafe_b64encode(key_bytes)
		if not (ctr + 1) % 1000:
			print(f"[*] Keys tested: {ctr +1:,}", end = "\r")
		try:
			plaintext = Fernet(key).decrypt(ciphertext)
			header = plaintext[:32]
			if test_png(header):
				print(f"[+] KEY FOUND: {key}")
				# Writing to a file
				with open("BINGO.png", "wb") as file:
					file.write(plaintext)
						break
		except Exception:
			pass

	# Now initialize the Fernet system with the given key
	# and try to decrypt your challenge.
	# Think, how do you know that the key tested is the correct key
	# (i.e., how do you break out of this infinite loop)?
		ctr += 1

if __name__ == "__main__":
	brute_force()
```

Rezultat prethodno navedenog koda bio je PNG file sa našim imenom. Pri pronalasku rješenja za dani izazov od velike koristi je bilo poznavanje činjenica da svaki PNG file ima identičnih prvih 8 bajtova. Na taj smo način znatno suzili izbor ključeva koje moramo provjeravati i ubrzali izvođenje programa.