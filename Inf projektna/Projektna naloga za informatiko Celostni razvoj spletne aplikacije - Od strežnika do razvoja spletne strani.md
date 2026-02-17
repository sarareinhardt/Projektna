
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
Virtualizacija je proces razdelitve enega fizičnega strežnika na več ločenih virtualnih strežnikov (virtualnih strojev ali VM) s pomočjo specializirane programske opreme, imenovane hipervizor.