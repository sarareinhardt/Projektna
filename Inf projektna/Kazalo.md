2. teoretično ozadje virtualizacije 
	2.1 Od tradicionalnih računalnikov do virtualizacije
	- en sam operacijski sistem na strojni opremi
	- slaba izkoriščenost strojnih resursov
	- problem več fizičnih strežnikov
	2.2 Kaj je virtualizacija in kako delujejo VM-ji
	- definicija virtualnega stroja
	- virtualna strojna oprema vs. realna strojna oprema
	- koncept izolacije
	2.3 Prednosti virtualizacije
	- večja učinkovitost & nižji stroški
	- snapshot-i & enostavne varnostne kopije
	- lažje testiranje
	- boljša varnostna izolacija
	- obnovitev po izpadu / napakah
	- (opcijsko) kratka primerjava s kontejnerizacijo
	
	2.4 Kaj so hipervizorji (kratka uvodna razlaga)
	- definicija hipervizorja
	- visokonivojska razlika tip-1 vs tip-2
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