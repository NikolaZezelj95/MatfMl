

**Learning to Rank**

Problem rangiranja stavki će ovde biti razmatran. Rangiranje možemo posmatrati kao problem regresije ili klasifikacije, medjutim postoje situacije kada nam takav pristup problemu može ograničiti mogućnosti u odnosu na zahteve koje imamo za rangiranje. To je situacija kada nam npr. nije dovoljna verovatnoća da korisnik klikne na ponudjenu stavku, već na osnovu podataka želimo da formiramo sistem preporuka, za šta nam je neophodno rangiranje. Osim toga potreba rangiranja se često javlja kod problema pretrage informacija, na velikoj kolekciji dokumenata. Korisnik u takvom scenariju preko nekog upita pokušava da pronadje odredjene informacije, a na osnovu dokumenata i upita potrebno je rangirati dokumente koji će biti najrelevantniji za korisnika. Zato je potrebno na osnovu upita odrediti koji su to dokumenti(obično su to web stranice) relevantni i koliko. U ovakvim situacijama dokumenti se obično dele u grupe prema upitima kojima odgovaraju, te se nakon toga rangiraju. Ovde će bit prikazani neki modeli koji su se pokazali korisnim u ovakvim vrstama problema.

