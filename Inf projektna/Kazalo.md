2. teoretično ozadje virtualizacije 
	2.1 Od tradicionalnih računalnikov do virtualizacije
	- en sam operacijski sistem na strojni opremi
	- slaba izkoriščenost strojnih resursov
	- problem več fizičnih strežnikov
	2.2 Kaj je virtualizacija in kako delujejo VM-ji
	- definicija virtualnega stroja
	- virtualna strojna oprema vs. realna strojna oprema
	- koncept izolacije
	- Kaj so hipervizorji (kratka uvodna razlaga)
	- definicija hipervizorja
	- visokonivojska razlika tip-1 vs tip-2
	2.3 Prednosti virtualizacije
	- večja učinkovitost & nižji stroški
	- snapshot-i & enostavne varnostne kopije
	- lažje testiranje
	- boljša varnostna izolacija
	- obnovitev po izpadu / napakah
	- (opcijsko) kratka primerjava s kontejnerizacijo
	

	2.5 Hipervizorji podrobneje
	- dodeljevanje virov (CPU / RAM)
	- virtualne naprave
	- CPU virtualizacijske razširitve
	- mehanizem snapshotov
	- VM disk slike
	- “live migration”
	
	2.6 Proxmox VE kot hipervizorska platforma
	- pregled Proxmoxa
	- KVM + LXC sklad
	- Debian osnova
	- storage pool-i
	- omrežni bridge-i
	- zakaj je Proxmox primeren za ta projekt


## 🔹 2.2 Kaj je virtualizacija in kako delujejo VM-ji

👉 DODAJ:

- ✔ definicijo virtualnega stroja (VM)
- ✔ razlika:
    - virtualna vs fizična strojna oprema
- ✔ izolacija (zelo pomembno!)
- ✔ vloga hipervizorja (1–2 stavka)

👉 OPCIJSKO (če želiš + točke):

- kratka omemba emulacije vs virtualizacije (1 stavek)

---

## 🔹 2.3 Prednosti virtualizacije

👉 imaš ideje, samo napiši:

- ✔ večja izkoriščenost virov
- ✔ nižji stroški (hardware + elektrika)
- ✔ snapshoti (backup)
- ✔ lažje testiranje
- ✔ izolacija (varnost)
- ✔ hitra obnovitev sistema

👉 na koncu dodaj:

- ✔ 1 stavek o uporabi v praksi (npr. data centri)

---

## 🔹 2.4 (združeno) Delovanje virtualizacije

👉 napiši NA KRATKO:

- ✔ kako se deli CPU
- ✔ kako se deli RAM
- ✔ disk slike (VM kot datoteka)
- ✔ snapshot (kaj pomeni)
- ✔ (opcijsko) live migration – 1 stavek

👉 NE iti preveč v detajle

---

## 🔹 2.5 Proxmox VE (ZELO pomembno)

👉 tukaj moraš biti malo bolj konkretna:

- ✔ kaj je Proxmox VE
- ✔ da temelji na Debianu
- ✔ da uporablja:
    - KVM (VM-ji)
    - LXC (containerji)
- ✔ storage (diski, pooli)
- ✔ network bridge (osnovno)

👉 NAJPOMEMBNEJŠE:

- ✔ zakaj si ga izbrala za projekt

(npr.)

> Proxmox omogoča enostavno upravljanje virtualnih strežnikov preko spletnega vmesnika, kar je primerno za učenje in praktično uporabo.


 3. Postavitev virtualnega strežnika
- namestitev Proxmoxa
- ustvarjanje VM
- OS (npr. Linux)
- osnovna konfiguracija

4. Razvoj spletne aplikacije
- kaj si naredila
- tehnologije (HTML/CSS/JS/Python/…)
- opis delovanja

5. Postavitev aplikacije
- prenos na strežnik
- web server (npr. Nginx)
- dostop prek omrežja

6. Zaključek