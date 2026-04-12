
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

V naslednjem koraku konfiguriram mrežne vmesnike. Ker gre za svežo inštalacijo operacijskega sistema izberemo kot model mrežne naprave VirtIO, ker na virtualni napravi deluje bolj optimalno kot simulacija fizičnih mrežnih gonilnikov, ki bi jih sicer tudi lahko izbrala, vendar se to navadno uporablja samo v primerih, ko se na virtualni strežnik prenaša napravo, ki je prej že uporabljala fizični mrežni gonilnik. Ob izboru mrežne naprave moram tudi rezervirati IP naslove, ki jih bom uporabljala na tem strežniku in hkrati določiti v katerih virtualnih WLAN omrežjih se ti IP-naslovi nahajajo. Če te nastavitve niso izbrane pravilno strežnik ne bo imel dostopa do interneta ali drugih virov v omrežju, več o tem pa bom napisala kasneje. Za izbor IP naslova pogledam kje imamo prost IP naslov in vidimo v kateremu IP naslovu se nahaja
![[Pasted image 20260411225251.png]]
![[Pasted image 20260411225840.png]]
Za to, da vem kateri VLAN vpisati v nastavitve naše virtualne naprave, moram pred tem določiti IP naslov, ki ga bo uporabljala virtualna naprava, kar naredim v zgoraj prikazanem orodju IPAM. Ko je to narejeno, v prvi od dveh slik vidim, da je VLAN ID v našem primeru 1012, tega pa vpišem v nastavitve mrežne naprave in s tem zaključim nastavitve mrežnih parametrov.
![[Pasted image 20260411225957.png]]

V naslednjem koraku preverim pravilnost vseh nastavitev, ki sem jih naredila. in s klikom na gumb Finish dejansko ustvarim virtualno napravo.
![[Pasted image 20260411230210.png]]

Po zagonu virtualne naprave začnem z namestitvijo operacijskega sistema. Ko se operacijski sistem namesti zažene aplikacijo za konfiguracijo parametrov operacijskega sistema. Znotraj te aplilacije izberem večino privzetih parametrov, razen da naslov mrežnega vmesnika nastavim na statični IPv4 naslov, ki je bil prej nastavljen (namesto privzetega DHCP)
![[Pasted image 20260412001201.png]]

Po tem pa nadaljujem s privzetimi nastavitvami, dokler ne pridem do izbora diska kamor namestim operacijski sistem.Pri tem izberm manjši hiter disk za operacijski sistem, večji počasnejši disk pa bo formatiran kot podatkovni disk. Vse izbrane nastavitve so prikazane v naslednji sliki. Ko izberemo "done" se prične dejanska konfiguracija operacijskega sistema.
![[Pasted image 20260412001638.png]]

Kasneje opišem osnovne podatke o uporabniku, kot so ime uporabnika, uporabniško ime in geslo in ime uporabnika in nadaljujemo s privzetimi parametri do SSH konfiguracije, kjer omogočimo namestitev SSH strežnika.
![[Pasted image 20260412002331.png]]
![[Pasted image 20260412002513.png]]

po končani konfiguraciji se bo virualna naprava poskušala zagnati. Pri tem se zažene iz naprave, ki je nastavljena kot zagonska naprava (*ang boot device*) vrstni red teh naprav je odvisen od vrstnega reda ki sem ga definirala v nastavitvah, zato se moram pred dejanskim zagonom prepričati, da je kot zagonska naprava izbran tisti disk na katerega smo namestili operacijski sistem. V našem primeru je to SCSI1 in tega nastavim kot primarno zagonsko napravo. Ko je to opravljeno, lahko ponovno poskusimo zagnati.
![[Pasted image 20260412004056.png]]

Ko je to opravljeno, lahko ponovno poskusimo zagnati. Vidimo, da je delovalo in tako smo se uspeli prijaviti na naš sveže zagnan virtualni strežnik. 
![[Pasted image 20260412005024.png]]

Ker je prijava z geslom manj varna, bo prvi naslednji korak ustvaritev para SSH ključev (javnega in privatnega) in vgradnja mojega javnega SSH ključa na strežnik. 
Za kreiranje privatnega in javnega SSH ključa uporabimo program PuTTYgen 
![[Pasted image 20260412005306.png]]
Ko zgenerira ključa oba shranim.
Zaradi varnosti datoteko s privatnim ključem shranimo z geslom. Tako bomo vsakič, ko je potrebno uporabiti privatni ključ potrebni vpisati tudi geslo. Na tak način se zaščitimo pred tem, da bi nekdo samo skopiral našo datoteko s ključem in dostopal do vseh računalnikov, kjer imam omogočen dostop z mojim SSH ključem.
##### Dostop do strežnika s SSH ključem
Ob namestitvi strežnika smo namestili open SSH prgramski paket, ki omogoča dostop do ukazne vrstice prek enkriptiranega SSH protokola. Dostop je mogoč z uporabo uporabniškega imena in gesla ali z uporabo SSH ključa, kar je bolj varen način. Zato, da bi lahko uporabljali SSH ključ, moramo prvo generirati par ključev, kar bilo opisano v prejšnjem koraku. Zdaj imamo datoteko s privatnim ključem in datoteko z javnim ključem v UNIX formatu.
Zato, da se lahko prijavim na strežnik s SSH ključem je potrebno na tem strežniku v domačem direktoriju uporabnika s katerim se želimo prijaviti ustvariti prvo .ssh/authorized_keys če te še ni ustvarjena in tej datoteki na koncu dodamo naš javni SSH ključ. 
Primer javnega SSH ključa v UNIX formatu, ki je primeren za dodajanje v datoteko authorized_keys. 
``` bash
echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCD7SWKlGWcDMHtYNt195Each9DJEsomryBzjx6ZH/qvIogJw2I5p++x7lAiNdHGYdGQhR67gEewxpwhJfH6+pCuc7BAy56K+smfwVJvW/nOaxIzkTgydtKODIrs0/UdiRFaVB3Nk1msZGiAqvRn/xMtLFUzdNTLXa4uzQ82JmJY/yu1JjOKLxAM8EAmllgSSkuX7BM2+3BPUCqQqSwxLwHEhnejWbGDVyTDrRXa0cNNwIT4v4ivxiE11JmsNOXXMEn0iJAplbTS+4JqLZhfWykuid2SdcLQqZyLtV1PQ6YzYfAmFROpnlLvKRz73jssdu+W/P6mmmV5xlzi2/gEBIl rsa-key-20260412 >> .ssh/authorized_keys
```
Če ta ukaz izvedemo v domačem direktoriju uporabnika lahko enostavno dodamo ta ključ v ustrezno datoteko, ki omogoča dostop na strežnik s tem uporabniškim imenom in SSH ključem brez uporabe gesla.
Na tak načn - z uporabo SSH ključa - lahko tudi relativno varno dostopamo do root računa na liux strežnikih. Tudi v tem primeru v /root/.ssh/authorized_keys dodamo naš javni SSH ključ in se lahko prijavimo na strežnik kot root uporabnik.


##### Dostop do strežnika s SSH ključem iz Windows računalnika

Enako kot za dostop z uporabniškim imenom in geslom za sam dostop tudi v tem primeru uporabljam program Putty. V primeru, ko želimo dostopati s SSH ključem uporabimo program Pagent v katerega dodamo naš privatni SSH ključ in šele po tem poženemo program Putty, ki dostopa do strežnika z imenom uporabnika kamor smo skopirali naš javni ključ kot je opisano zgoraj. Ko bo Putty odprl sejo do strežnika ne bo potrebno vpisati gesla, temveč se bo prijava zgodila s SSH ključem. (Authenticating with public key "rsa-key-20260412" from agent)
![[Pasted image 20260412182856.png]]
Poleg tega, da je to bolj enostavno je tudi bolj varno. 
Na tem mestu, ker vemo, da se lahko s SSH ključem prijavimo na računalnik bi lahko tudi preprečili možnost prijave z geslom, kar doatno poveča varnost. To lahko naredimo s spremembo sistemske ssh_config datoteke, kjer odkomentiramo vrstico PasswordAuthentication kjer piše 'yes' in jo spremenimo na 'no'.

Pred nadaljevanjem se prepričamo, da: 
- imamo dostop do interneta
![[Pasted image 20260412185453.png]]
vidimo, da URL pretvori v IP naslov (kar pomeni, da DNS mehanizem dela)
- naredimo nadgradnjo operacijskega sistema na najnovejšo verzijo, kar naredimo na naslednji način: 
```bash
sudo apt get update
sudo apt get upgrade
```
po tem pa namestimo še ostalo aplikativno programsko opremo
##### Nastavitev strežniškega okolja in spletne aplikacije

Prvo zagotovimo, da se bo naš željeni URL app.xenya.net pretvoril v naslov našega spletnega strežnika, kar nastavim na DNS strežniku prek spletnega vmesnika PowerDNS-Admin
![[Pasted image 20260412194359.png]]
Za tem preverimo, da se ime res pretvori v IP naslov našega strežnika
![[Pasted image 20260412194533.png]]


Za to, da bi lahko postavili spletni strežnik moramo namestiti naslednjo programsko opremo, kar zajema spletni strežnik Apache in podporo za PHP:
- **apache2** → spletni strežnik (prikaže tvojo spletno stran uporabnikom)
- **php** → osnovni PHP interpreter (omogoča izvajanje PHP kode)
- **php-cli** → PHP za ukazno vrstico (uporabljaš v terminalu, npr. skripte)
- **libapache2-mod-php** → poveže PHP z Apache (da Apache lahko poganja PHP datoteke)
- **php-mysql** → omogoča povezavo PHP aplikacije z MySQL bazo
To naredimo z naslednjim ukazom:
```bash
sudo apt install apache2 php php-cli libapache2-mod-php php-mysql
```


Pri vzpostavitvi spletne aplikacije sem najprej nastavila DNS zapis tipa A, s katerim sem domeno `app.xenya.net` povezala z javnim IP naslovom strežnika `31.7.206.105`. DNS (Domain Name System) omogoča pretvorbo domen v IP naslove, kar uporabnikom omogoča dostop do strežnika preko razumljivega imena. Delovanje sem preverila z ukazom `ping app.xenya.net`, ki preveri, ali se domena pravilno prevede v IP naslov.

Nato sem na strežnik namestila spletni strežnik Apache in programski jezik PHP z ukazom `sudo apt install apache2 php php-cli libapache2-mod-php php-mysql`, ki namesti potrebne komponente. Apache sprejema HTTP zahteve in vrača spletne strani, PHP pa omogoča izvajanje dinamične logike. Modul `libapache2-mod-php` omogoča Apache strežniku izvajanje PHP kode, medtem ko `php-mysql` omogoča povezavo z bazo podatkov. Strežnik sem zagnala z ukazom `sudo systemctl start apache2` in omogočila samodejni zagon z `sudo systemctl enable apache2`, kjer `systemctl` upravlja sistemske storitve.

Spletno aplikacijo sem prenesla v mapo `/var/www/app`, ki jo Apache uporablja kot vir datotek. Mapo sem ustvarila z ukazom `sudo mkdir -p /var/www/app`, kjer `mkdir` ustvari mapo, parameter `-p` pa omogoča ustvarjanje celotne poti. Datoteke sem kopirala z ukazom `sudo cp -r your-app/* /var/www/app/`, kjer `cp` kopira datoteke, `-r` pa omogoča rekurzivno kopiranje map.

Za pravilno delovanje sem nastavila lastništvo in pravice z ukazoma `sudo chown -R www-data:www-data /var/www/app` in `sudo chmod -R 755 /var/www/app`. Ukaz `chown` spremeni lastnika datotek na uporabnika `www-data`, pod katerim teče Apache, parameter `-R` pa pomeni, da se sprememba izvede rekurzivno. Ukaz `chmod` nastavi dovoljenja, kjer 755 pomeni, da ima lastnik pravice za branje, pisanje in izvajanje, ostali pa samo za branje in izvajanje.

Za povezavo domene s aplikacijo sem ustvarila konfiguracijo virtualnega gostitelja z ukazom `sudo nano /etc/apache2/sites-available/app.xenya.net.conf`. V tej datoteki sem določila `ServerName`, ki predstavlja domeno, in `DocumentRoot`, ki določa mapo z datotekami. Direktiva `AllowOverride All` omogoča uporabo `.htaccess` datotek za dodatno konfiguracijo, medtem ko `Require all granted` omogoča dostop vsem uporabnikom. Virtualni gostitelji omogočajo, da en strežnik gosti več različnih spletnih strani.

Konfiguracijo sem aktivirala z ukazom `sudo a2ensite app.xenya.net.conf`, kjer `a2ensite` omogoči izbrano stran, ter omogočila modul za prepisovanje URL naslovov z `sudo a2enmod rewrite`. Spremembe sem uveljavila z `sudo systemctl reload apache2`, ki ponovno naloži konfiguracijo brez popolnega ponovnega zagona strežnika.

Za omogočanje dostopa iz interneta sem nastavila požarni zid z ukazi `sudo ufw allow 80` in `sudo ufw allow 443`, kjer `ufw` (Uncomplicated Firewall) upravlja pravila požarnega zidu, ukaz `allow` pa odpre določena vrata. Vrata 80 se uporabljajo za HTTP promet, vrata 443 pa za HTTPS. Požarni zid sem aktivirala z `sudo ufw enable`.

Za varno komunikacijo sem namestila orodje Certbot z ukazom `sudo apt install certbot python3-certbot-apache` in pridobila SSL certifikat z `sudo certbot --apache -d app.xenya.net`. Certbot avtomatsko pridobi certifikat od organizacije Let's Encrypt in ga namesti v Apache konfiguracijo, kar omogoča uporabo HTTPS protokola za šifrirano komunikacijo.

Ker aplikacija zapisuje podatke v datoteko (npr. `posts.txt`), sem posebej nastavila pravice za mapo `/var/www/app/data` z ukazi `sudo chown -R www-data:www-data /var/www/app/data`, `sudo chmod 775 /var/www/app/data` in `sudo chmod 664 /var/www/app/data/posts.txt`. Dovoljenje 775 pomeni, da ima lastnik in skupina pravice za branje, pisanje in izvajanje, ostali pa samo za branje in izvajanje, medtem ko 664 za datoteko omogoča branje in pisanje lastniku in skupini ter samo branje ostalim. S tem omogočim strežniku zapisovanje podatkov, hkrati pa omejim dostop drugim uporabnikom, kar izboljša varnost.











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

KarKol!7189



