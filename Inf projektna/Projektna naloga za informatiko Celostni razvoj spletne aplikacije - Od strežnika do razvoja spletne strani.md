2
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
Večino časa s takšno uporabo strojne opreme ne izkoristimo 100% procesorskega časa in imamo manj učinkovit sistem. Prav tako bi rabili več fizičnih naprav, če bi želeli ustvariti več strežnikov (npr. bazo, web strežnik...) To pomeni, da kljub temu, da bi 1 računalnik mogoče lahko imel dovolj obdelovalne moči, za upravljanje obeh strežnikov, moramo imeti 2 fizična računalnika. To pomeni manjša učinkovitost, več porabe elektrike, vzdrževanja ...To težavo nam sedaj reši tehnologija virtualizacije, ki omogoča kreiranje večjega števila virtualnih strežnikov na enem fizičnem. Dodatno nam virtualizacija poenostavi vzdrževanje,saj nam omogoča izvajanje periodičnih varnostnih kopij in olajša nadzor nad strežniki samimi, ker imamo boljši pregled nad uporabo resourcov in njihovo prilagajanje (tako procesorskega časa, diskovnega prostora, mrežnih vmesnikov kot delovnega pomnilnika), za kar bi sicer potrebovali posebna orodja oziroma celo menjavo strojne opreme v strežniku.

##### 2.2 Kaj je virtualizacija in kako delujejo VM-ji
	- definicija virtualnega stroja
	- virtualna strojna oprema vs. realna strojna oprema
	- koncept izolacije

Virtualizacija je proces razdelitve enega fizičnega strežnika na več ločenih virtualnih strežnikov (virtualnih strojev ali VM (virtual machine) ) s pomočjo specializirane programske opreme, imenovane hipervizor. 

Hipervizor, znan tudi kot virtual machine monitor ali VMM, je programska oprema, ki ustvarja in izvaja virtualne stroje (VM). Hipervizor omogoča enemu gostiteljskemu računalniku, da podpira več gostujočih VM-jev z virtualnim deljenjem svojih virov, kot so pomnilnik in procesor.

Poznamo 3 vrste virtualizacije:
- Popolna virtualizacija uporablja hipervizor, ki nadzira vire fizičnega strežnika, hkrati pa ohranja neodvisnost vsakega virtualnega strežnika in ne omogoča medsebojnega komuniciranja. Hipervizor posreduje dostop do strojne opreme, kot so procesor, pomnilnik, shranjevanje in omrežne naprave, ko virtualni stroji izvajajo aplikacije. Vendar je omejitev popolne virtualizacije, da hipervizor sam potrebuje procesorske vire za delovanje. Upravljati in usklajevati mora virtualne stroje z dodeljevanjem časa procesorja, mapiranjem virtualnega pomnilnika na fizični RAM, emuliranjem strojne opreme in upravljanjem diskovnih in omrežnih operacij. Te upravljavske naloge porabljajo procesorski čas in sistemske vire, kar ustvarja dodatne "stroške", ki lahko zmanjšajo zmogljivost v primerjavi z aplikacijami, ki tečejo neposredno na fizični strojni opremi.

- Para-virtualizacija (*Para virtualization* ) je vrsta virtualizacije, pri kateri gostujoči operacijski sistemi vedo, da delujejo v virtualiziranem okolju, in so zasnovani za "sodelovanje" s hipervizorjem. Zaradi tega zavedanja lahko operacijski sistemi neposredno komunicirajo s hipervizorjem prek posebnih vmesnikov, imenovanih **hypercalls**, da zahtevajo dostop do strojne opreme, kot so čas procesorja, pomnilnik, shranjevanje in omrežne naprave. Namesto da bi operacijski sistemi komunicirali s popolnoma emulirano strojno opremo, pošiljajo zahteve neposredno hipervizorju, ki nato upravlja in razporeja vire fizičnega strežnika med virtualnimi stroji. Ker hipervizorju ni treba emulirati toliko komponent strojne opreme ali prestrezati toliko navodil strojne opreme, potrebuje manj procesnih virov za upravljanje virtualnih strojev. To lahko izboljša splošno učinkovitost in zmanjša stroške virtualizacije. Vendar je omejitev para-virtualizacije, da je treba gostujoče operacijske sisteme običajno spremeniti ali posebej zasnovati, da podpirajo to vrsto virtualizacije, tako da lahko pravilno komunicirajo s hipervizorjem.

- Virtualizacija na ravni operacijskega sistema: Za razliko od popolne in para-virtualizacije virtualizacija na ravni operacijskega sistema ne uporablja hipervizorja. Namesto tega vse naloge hipervizorja opravlja virtualizacijska zmogljivost, ki je del operacijskega sistema fizičnega strežnika. Vendar morajo vsi virtualni strežniki pri tej metodi virtualizacije strežnikov uporabljati isti operacijski sistem.