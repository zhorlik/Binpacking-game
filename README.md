# Binpacking-game
Egy játék a 2 dimenziós ládapakolás problémáról

Menü és beállítások:
A program lefuttatásakor, a főmenübe kerülünk, ahol 3 gombot látunk: play, settings és quit. 
Érdemes elsőnek a settings gombra nyomni, ahol beállíthatjuk, hogy mennyi és milyen méretű elemet, mekkora ládákba szeretnénk pakolni. 
Kattintásra a jobb oldalon látszódni fog, hogy mit választottunk ki. 
Ezután a back gombbal visszatérhetünk a menübe ahhonan a play-jel elindíthatjuk a játékot. 

A játék:

A képernyő alján találhatjuk a pakolni való elemeket, felül pedig hét darab gombot.  
Rendre: Add Bin, Remove Bin, Reset Items, FFD solution, BFD solution, Check, és Back to Menu. 
Add és Remove Bin buttonok, hozzáadnak, illetve elvesznek ládákat. 
A Reset Items az összes elemet visszarakja a kezdő pozícióba. 
Az FFD solution a first fit decreasing közelítő algoritmust (ilyen kis imputokra nekünk eddig mindig optimális megoldást ad) alkalmazva bepakolja a ládákba az elemeket. 
A BFD solution a best fit decreasing közelítő algoritmust alkalmazva bepakolja a ládákba az elemeket. 
Ezeknek a megoldásait tekintjük jó megoldásnak, mivel ilyen kis imputokra optimális megoldást adnak. 
A Check gomb leellenőrzi a megoldásunkat:
Ha túl sok dobozt használtunk fel: "You used too many bins... get back to work!" 
Ha az elemek ütköznek: "Stacking items on top of each other? This isn't Jenga, my friend?" 
Ha kihagytunk valamit: "Keeping items untouched, are we? Keeping things minimalist, huh?" 

Ha úgy gondoljuk elég volt, akkor a Back to Menu gombbal vissza mehetünk a menübe.
