
```toc
Uvod
Teorija virtualizacije
```

# 1. Uvod
V projektni nalogi bom predstavila razvoj preproste spletne aplikacije ter proces njene postavitve na virtualen strežnik s pomočjo hypervisorja Proxmox. Najprej bom razložila osnovne koncepte virtualizacije naprav in namen uporabe virtualnih strežnikov v modernih informacijskih sistemih.

V nadaljevanju bom prikazala, kako ustvarimo virtualni strežnik v Proxmoxu in kaj je potrebno na njem postaviti, da je primeren za gostovanje spletne aplikacije. To vključuje izbiro in namestitev operacijskega sistema, osnovno konfiguracijo in varovanje strežnika ter pripravo storitev, ki jih aplikacija potrebuje za delovanje.

Nazadnje bom prikazala še razvoj in postavitev preproste testne spletne aplikacije. Predstavila bom postopek razvoja, premik aplikacije na strežnik ter končno konfiguracijo, da je aplikacija dostopna preko omrežja. S tem bom pokazala celoten potek – od teorije virtualizacije do konkretne postavitve aplikacije v delujočem produkcijskem okolju.

# 2. Teorija virtualizacije

##### 2.1 Od tradicionalnih računalnikov do virtualizacije

Včasih je bilo običajno, da na 1 fizičnem računalniku deluje samo 1 operacijski sistem z enim kernelom, ki lahko neposredno dostopa do strojne opreme (CPU, RAM, ...)
Večino časa s takšno uporabo strojne opreme ne izkoristimo 100% procesorskega časa in imamo manj učinkovit sistem. Prav tako bi potrebovali več fizičnih naprav, če bi želeli ustvariti več strežnikov (npr. bazo, web strežnik...) To pomeni, da kljub temu, da bi 1 računalnik mogoče lahko imel dovolj obdelovalne moči, za upravljanje obeh strežnikov, moramo imeti 2 fizična računalnika. To pomeni manjša učinkovitost, več porabe elektrike, vzdrževanja ...To težavo nam sedaj reši tehnologija virtualizacije, ki omogoča kreiranje večjega števila virtualnih strežnikov na enem fizičnem. Dodatno nam virtualizacija poenostavi vzdrževanje,saj nam omogoča izvajanje periodičnih varnostnih kopij in olajša nadzor nad strežniki samimi, ker imamo boljši pregled nad uporabo resourcov in njihovo prilagajanje (tako procesorskega časa, diskovnega prostora, mrežnih vmesnikov kot delovnega pomnilnika), za kar bi sicer potrebovali posebna orodja oziroma celo menjavo strojne opreme v strežniku.

##### 2.2 Kaj je virtualizacija in kako delujejo VM-ji
	- definicija virtualnega stroja
	- virtualna strojna oprema vs. realna strojna oprema
	- koncept izolacije

Virtualizacija je proces razdelitve enega fizičnega strežnika na več ločenih virtualnih strežnikov. S pomočjo virtualizacije vzamemo strojno opremo ene naprave (CPU, RAM, SSD, ...) njihove vire razdelimo med več različnih "okolij", ki jih imenujemo virtualni strežniki (*ang. virtual machine ali VM*). Naredimo to, da iz perspektive vsakega od teh virtualnih strežnikov zgleda, kot da vsak od njih deluje na svojem računalniku. Te virtualni strežniki lahko poganjajo vsak svoj operacijski sistem, ki pa se lahko razlikuje od operacijskega sistema drugih virtualnih strežnikov na isti fizični napravi in lahko med seboj komunicirajo. Uporabnik  virtualnega strežnika lahko med različnimi operacijskimi sistemi preklaplja na podoben način, kot preklaplja med procesi, ki se izvajajo sočasno v enem operacijskem sistemu.
Virtualizacija omogoča poganjanje operacijskih sistemov kot aplikacije, ki delujejo znotraj  drugih operacijskih sistemov
To počnemo s pomočjo specializirane programske opreme, imenovane hipervizor. 
Hipervizor, znan tudi kot virtual machine monitor ali VMM, je programska oprema, ki ustvarja in izvaja virtualne stroje (VM). Hipervizor omogoča enemu gostiteljskemu računalniku, da podpira več gostujočih VM-jev z virtualnim deljenjem svojih virov, kot so pomnilnik in procesor.

Poznamo 3 vrste virtualizacije:
- Popolna virtualizacija uporablja hipervizor, ki nadzira vire fizičnega strežnika, hkrati pa ohranja neodvisnost vsakega virtualnega strežnika in ne omogoča medsebojnega komuniciranja. Hipervizor posreduje dostop do strojne opreme, kot so procesor, pomnilnik, shranjevanje in omrežne naprave, ko virtualni stroji izvajajo aplikacije. Vendar je omejitev popolne virtualizacije, da hipervizor sam potrebuje procesorske vire za delovanje. Upravljati in usklajevati mora virtualne stroje z dodeljevanjem časa procesorja, mapiranjem virtualnega pomnilnika na fizični RAM, emuliranjem strojne opreme in upravljanjem diskovnih in omrežnih operacij. Te upravljavske naloge porabljajo procesorski čas in sistemske vire, kar ustvarja dodatne "stroške", ki lahko zmanjšajo zmogljivost v primerjavi z aplikacijami, ki tečejo neposredno na fizični strojni opremi.

- Para-virtualizacija (*Para virtualization* ) je vrsta virtualizacije, pri kateri gostujoči operacijski sistemi vedo, da delujejo v virtualiziranem okolju, in so zasnovani za "sodelovanje" s hipervizorjem. Zaradi tega zavedanja lahko operacijski sistemi neposredno komunicirajo s hipervizorjem prek posebnih vmesnikov, imenovanih **hypercalls**, da zahtevajo dostop do strojne opreme, kot so čas procesorja, pomnilnik, shranjevanje in omrežne naprave. Namesto da bi operacijski sistemi komunicirali s popolnoma emulirano strojno opremo, pošiljajo zahteve neposredno hipervizorju, ki nato upravlja in razporeja vire fizičnega strežnika med virtualnimi stroji. Ker hipervizorju ni treba emulirati toliko komponent strojne opreme ali prestrezati toliko navodil strojne opreme, potrebuje manj procesnih virov za upravljanje virtualnih strojev. To lahko izboljša splošno učinkovitost in zmanjša stroške virtualizacije. Vendar je omejitev para-virtualizacije, da je treba gostujoče operacijske sisteme običajno spremeniti ali posebej zasnovati, da podpirajo to vrsto virtualizacije, tako da lahko pravilno komunicirajo s hipervizorjem.

- Virtualizacija na ravni operacijskega sistema: Za razliko od popolne in para-virtualizacije virtualizacija na ravni operacijskega sistema ne uporablja hipervizorja. Namesto tega vse naloge hipervizorja opravlja virtualizacijska zmogljivost, ki je del operacijskega sistema fizičnega strežnika. Vendar morajo vsi virtualni strežniki pri tej metodi virtualizacije strežnikov uporabljati isti operacijski sistem.


V naslednjem poglavju bom te koncepte uporabila pri postavitvi virtualnega strežnika v okolju Proxmox.

##### Postavitev strežnika v Proxmoxu

Najprej moram narediti virtualno napravo, ki bo gostovala mojo spletno aplikacijo. Za spletno aplikacijo potrebujem: spletni strežnik (*ang. web server*) - v našem primeru bo to Apache. Polek spletnega strežnika, pa potrebujem dodatne programske pakete, kot so podatkovna baza (PostgreSQL) in PHP.
Strežnik bo poganjal operacijski sistem Ubuntu, ki nam zagotavlja osnovno okolje, znotraj katerega bomo namestili zgoraj naštete programe. Prav tako nam zagotavlja datotečni sistem, da lahko vse naše projektne datotečne vire ( `index.php`, `forum.php`, `header.php`, `css.css`, `js.js` in `posts.txt`) shranimo na strežniku. Hkrati nam že sam datotečni sistem zagotavlja SSH in FTPS dostop, ki omogočata varen dostop do ukazne vrstice in varen prenos (prek SSH protokola) na in iz strežnika. Za dostop do strežnika bomo morali kreirati 


Proxmox je sistem, ki nam teče na večih fizičnih stežnikih in na vsakem lahko teče več VM. Mi bomo naredili complete virtualization z Ubuntujem

Z gumbom Create VM sprožim postopek ustvarjanja virtualne naprave. Moram ugotoviti katero številko virtualne naprave bom izbrala (v našem primeru 601)  in jo poimenovala SaraUB-projektna. Resource pool je oznaka s katero lahko označim kdo je odgovoren za določeno virtualno napravo. Tej oznaki se lahko tudi določi pravice dostopa. Kasneje, ko bom končala postavitev bom odkljukala tudi, da se virtualna naprava samodejno zažene (Start at boot) ob morebitnem ponovnem zagonu hypervisorja
![[Pasted image 20260411221253.png]]



V naslednjem oknu moram izbrati kateri OS bom inštalirala na našo napravo. Standardna namestitev se na Proxmoxu naredi iz ISO datoteke, ki predstavlja inštalacijski virtualni CD operacijskega sistema. Ta CD izberem kot virtualni CD tako, da na vstreznem datotečnem sistemu izberem ISO datoteko, v kateri je zapisana vsebina tega CD-ja. Ker hočemo namestiti na našo virtualno napravo operacijski sistem Ubuntu, bomo za inštalacijo izbrali najnovejšo verzijo Ubuntu inštalacijskega diska (Ubuntu-24.04.4-live-server-amd64).
![[Pasted image 20260411231527.png]]


Po tem bom določila sistemske parametre, in sicer: kakšno grafično kartico in kakšne gonilnike za razne parametre bomo uporabili. V tem primeru bomo omogočili uporabo Quemu agenta, ki omogoča lažje upravljanje virtualnih naprav s strani operaterja Hiperviserja in sporoča dodatne informacije kot so IP naslovi ipd.
![[Pasted image 20260411221604.png]]


V naslednjem koraku izberem parametre virtualiziranega diska. Pri tem moramo predvsem določiti kje se bo nahajal, velikost in nekaj dodatnih parametrov (npr. ali delamo avtomatske varnostne kopije vsebine diska, itd - glej sliko). 
![[Pasted image 20260411221751.png]]


Zaradi varnosti bom na naši virtualni napravi naredila 2 diska. Eden bo sistemski, kjer bo nameščen operacijski sistem in bo tekel na flash disku (SCSI0). Ta obsega samo 32GB, kar je ravno toliko kolikor potrebujemo za namestitev operacijskega sistema. Za namestitev aplikacije in uporabniških podatkov pa kreiramo dodatni disk (SCSI1), ki ga kreiramo na počasnejšem, a bistveno večjem skladiščnem sistemu (ang.*Storage*) imenovanem SharedCeph2. Pri izboru diska sem hkrati omogočila, da uporabimo Write Back Cache, ki pohitri delovanje diska (predvsem pri pisanju.)
![[Pasted image 20260411222545.png]]

V naslednjem koraku izberem koliko procesorskih virov dodelimo naši virtualni napravi. V našem primeru bomo dodelili 2 procesorja in 2 jedri (*ang. core*) v vsakem procesorju, tako da imamo skupaj 4 jedra.
![[Pasted image 20260411222800.png]]

Po tem izberem količino pomnilnika. Spomin izberemo tako, da se naprava lahko dinamično povečuje do 8192MB. V našem primeru izberemo minimalno 2MB. V ta namen se uporablja Balooning gonilnik (*ang.driver*). Tako lahko bolj učinkovito izkoriščamo fizičen pomnilnik, ki si ga deli večje št. virtualnih naprav.
![[Pasted image 20260411231757.png]]

V naslednjem koraku konfiguriram mrežne vmesnike. Ker gre za svežo inštalacijo operacijskega sistema izberemo kot model mrežne naprave VirtIO, ker na virtualni napravi deluje bolj optimalno kot simulacija fizičnih mrežnih gonilnikov, ki bi jih sicer tudi lahko izbrali, vendar se to navadno uporablja samo v primerih, ko se na virtualni strežnik prenaša napravo, ki je prej že uporabljala fizični mrežni gonilnik. Ob izboru mrežne naprave moramo tudi rezervirati IP naslove, ki jih bomo uporabljali na tem strežniku in hkrati določiti v katerih virtualnih WLAN omrežjih se ti IP-naslovi nahajajo. Če te nastavitve niso izbrane pravilno naš strežnik ne bo imel dostopa do interneta ali do drugih virov v omrežju, več o tem pa bom napisala kasneje. Za izbor IP naslova pogledamo kje imamo prost IP naslov in vidimo v kateremu IP naslovu se nahaja
![[Pasted image 20260411225251.png]]
![[Pasted image 20260411225840.png]]
Za to, da vem kateri VLAN vpisati v nastavitve naše virtualne naprave moram pred tem določiti IP naslov, ki ga bo uporabljala virtualna naprava, kar naredim v zgoraj prikazanem orodju IPAM. Ko je to narejeno v prvi od dveh slik vidim, da je VLAN ID v našem primeru 1012, tega pa vpišem v nastavitve mrežne naprave in s tem zaključim nastavitve mrežnih parametrov.
![[Pasted image 20260411225957.png]]

V naslednjem koraku preverim pravilnost vseh nastavitev, ki sem jih naredila. in s klikom na gumb Finish dejansko ustvarim virtualno napravo.
![[Pasted image 20260411230210.png]]


















# Delovanje spletne aplikacije s strežniškega vidika

##### 1. Zagon aplikacije na virtualnem stroju

Aplikacija teče na virtualnem strežniku v okolju Proxmox, kjer je nameščen spletni strežnik ( Apache) in PHP. Vsi projektni datotečni viri ( `index.php`, `forum.php`, `header.php`, `css.css`, `js.js` in `posts.txt`) se nahajajo na strežniku.

Ko uporabnik v brskalniku odpre spletno stran, se na strežnik pošlje HTTP zahteva. Strežnik nato obdela PHP kodo in vrne generirano HTML stran uporabniku.



##### 2. Vstopna stran (index.php)

Datoteka `index.php` predstavlja začetno stran aplikacije. Vključuje datoteko `header.php`, CSS in JavaScript ter prikaže osnovno vsebino strani.

##### Strežniški proces:
- brskalnik zahteva `index.php`
- strežnik izvede `include "header.php"`
- datoteki se združita v eno HTML stran
- strežnik pošlje končni HTML uporabniku

Uporabnik nikoli ne vidi PHP kode, saj se ta izvede na strežniku.

---

##### 3. Glava strani (header.php)

Datoteka `header.php` vsebuje navigacijo in funkcionalnost za nastavitev uporabniškega imena.

##### Obdelava na strežniku

Ko uporabnik odda obrazec:
- strežnik zazna POST zahtevo
- prebere vrednost iz `$_POST`
- uporabi `trim()` za odstranitev presledkov
- shrani ime v piškotek (`setcookie()`)

Nato sledi preusmeritev:

```php
header("Location: ...");
exit;
```

To prepreči ponovno pošiljanje obrazca.

Ob naslednji zahtevi:

```php
$_COOKIE["display_name"]
```

Če piškotek obstaja, se ime vstavi v HTML in prikaže uporabniku.

---

##### 4. Forum (forum.php)

Datoteka `forum.php` omogoča uporabnikom objavljanje sporočil.

##### 4.1 Določitev podatkovne datoteke

```php
$postsFile = __DIR__ . "/data/posts.txt";
```

Datoteka deluje kot preprosta baza podatkov.

---

##### 4.2 Obdelava obrazca

```php
if ($_SERVER["REQUEST_METHOD"] === "POST")
```

Strežnik:
- prebere `name` in `comment`
- uporabi `trim()`
- nastavi privzeto ime ("Anonimni uporabnik")

Ustvari zapis:
- čas (`date()`)
- ime
- besedilo

---

##### 4.3 Shranjevanje podatkov

```php
file_put_contents($postsFile, $entry, FILE_APPEND);
```

Podatki se dodajo na konec datoteke.


##### 4.4 Preusmeritev

```php
header("Location: forum.php");
exit;
```

To prepreči podvajanje objav (Post/Redirect/Get).

---

##### 5. Prikaz objav

```php
file_exists($postsFile)
file_get_contents($postsFile)
```

Za varen prikaz:

```php
htmlspecialchars()
```

Preprečuje izvajanje škodljive kode.

```php
nl2br()
```

Pretvori prelome vrstic v HTML.



##### 6. CSS in JavaScript

Datoteki `css.css` in `js.js` določata videz in interaktivnost.

Strežnik ju ne obdeluje, ampak ju samo pošlje brskalniku.



##### 7. Shranjevanje podatkov (posts.txt)

Datoteka `posts.txt` vsebuje vse objave.

Gre za preprosto obliko trajnega shranjevanja brez podatkovne baze.



##### Zaključek

Delovanje aplikacije:

1. uporabnik pošlje zahtevo  
2. strežnik izvede PHP  
3. obdela podatke (obrazci, piškotki)  
4. shrani podatke  
5. ustvari HTML  
6. pošlje odgovor brskalniku  

To predstavlja osnovno delovanje spletne aplikacije, saj strežnik dinamično obdeluje podatke in ne prikazuje samo statične vsebine.

























## 🔹 2.2 Kaj je virtualizacija in kako delujejo VM-ji

👉 DODAJ:

- ✔ definicijo virtualnega stroja (VM)
- ✔ razlika:
    - virtualna vs fizična strojna oprema
- ✔ izolacija (zelo pomembno!)
- ✔ vloga hipervizorja (1–2 stavka)

👉 OPCIJSKO (če želiš + točke):

- kratka omemba emulacije vs virtualizacije (1 stavek)



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








1.7 - v ucbeniku je virtualization
18 je pa hypervisorji pa to

##### Predlogi za izboljšave

Najbolj pomembne izboljšave so seveda te, ki izboljšujejo varnost naše apluikacije. Med njimi najpomembnejša je uporaba HTTPS protokola za dostop do spletne aplikacije, ki zagotavlja SSL enkripcijo, varen dostop do spletne aplikacije, ki onemogoča nezaželenim ljudem prisluškovanje, predvsem pa avtentikacijo spletnega mesta, ki oteži izvedbo morebitnih phishing napadov, kjer bi lahko nekdo oponašal našo aplikacijo s slabim namenom.