{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {
    "colab_type": "text",
    "id": "HvPzuta3jjkX"
   },
   "source": [
    "\n",
    "\n",
    "**Learning to Rank-RankNet**\n",
    "\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "colab_type": "text",
    "id": "H_PRpw95kbdF"
   },
   "source": [
    "Problem rangiranja stavki će ovde biti razmatran. Rangiranje možemo posmatrati kao problem regresije ili klasifikacije, medjutim postoje situacije kada nam takav pristup problemu može ograničiti mogućnosti u odnosu na zahteve koje imamo za rangiranje. To je situacija kada nam npr. nije dovoljna verovatnoća da korisnik klikne na ponudjenu stavku, već na osnovu podataka želimo da formiramo sistem preporuka, za šta nam je neophodno rangiranje. Osim toga potreba rangiranja se\n",
    "često javlja kod problema pretrage informacija, na velikoj kolekciji dokumenata.\n",
    "Korisnik u takvom scenariju preko nekog upita pokušava da pronadje odredjene informacije, a na osnovu dokumenata i upita potrebno je rangirati dokumente koji\n",
    "će biti najrelevantniji za korisnika. Zato je potrebno na osnovu upita odrediti koji su to dokumenti(obično su to web stranice) relevantni i koliko. U ovakvim\n",
    "situacijama dokumenti se obično dele u grupe prema upitima kojima odgovaraju, te se nakon toga rangiraju. Ovde će bit prikazani neki modeli koji su se pokazali korisnim u ovakvim vrstama problema."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "colab_type": "text",
    "id": "9GoMEcMPpV6e"
   },
   "source": [
    "**Opis skupa podataka, i problema koji rešavamo**"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "colab_type": "text",
    "id": "3PZO0qtypMqV"
   },
   "source": [
    "Skup podataka je nad kojim ćemo demonstrirati rad modela je iz paketa LETOR 4.0 koji sadrži skupove podataka za rangiranje. Ova verzija paketa dostupna je od 2009. godine. Ona koristi kolekciju  Gov2 web stranica (oko 25 miliona stranica) i dva skupa upita iz Million Query Track , tkzv. skupove upita TREC 2007 i\n",
    "TREC 2008. Ova dva skupa upita označeni su sa  MQ2007 i MQ2008. MQ2007 sadrži oko 1700 različitih upita, dok  MQ2008 sadrži oko 800 upita sa označenim dokumentima. Paket LETOR4.0 sadrži 8 skupova podataka za različita rangiranja podeljenih u dva skupa upita i Gov2 kolekcije stranica. Svaki od skupova sadrži 5 foldera, u svakom folderu postoje tri podskupa, za trening, validaciju i test.\n",
    "Mi ćemo koristiti MQ2008 skup podataka za nadgledano učenje. Sam skup podataka koji koristimo generisan je tako da je svaka vrsta par dokument i upit. Prva kolona je oznaka relevantnosti para dokument-upit, što će nam i biti ciljna vrednost, druga kolona je id upita, zatim sledi 46 kolona koje su atributi koji opisuju par dokument - upit, nakon toga sledi kolona koja sadrži id dokumenta kao i komentar o posmatranom paru.  Sledi lista sa atributima koji opisuju par dokument-upit:\n",
    "\n",
    "\n",
    "1.   TF(Term frequency) of body\n",
    "2.   TF of anchor\n",
    "3.   TF of title\n",
    "4.   TF of URL\n",
    "5.   TF of whole document\n",
    "6.   IDF(Inverse document frequency) of body\n",
    "7.   IDF of anchor\n",
    "8.   IDF of title\n",
    "9.   IDF of URL\n",
    "10.  IDF of whole document\n",
    "11.  TF*IDF of body\n",
    "12.  TF*IDF of anchor\n",
    "13.  TF*IDF of title\n",
    "14.  TF*IDF of URL\n",
    "15.  TF*IDF of whole document\n",
    "16.  DL(Document length) of body\n",
    "17.  DL of anchor\n",
    "18.  DL of title\n",
    "19.  DL of URL\n",
    "20.  DL of whole document\n",
    "21.  BM25 of body\n",
    "22.  BM25 of anchor\n",
    "23.  BM25 of title\n",
    "24.  BM25 of URL\n",
    "25.  BM25 of whole document\n",
    "26.  LMIR.ABS of body\n",
    "27.  LMIR.ABS of anchor\n",
    "28.  LMIR.ABS of title\n",
    "29.  LMIR.ABS of URL\n",
    "30.  LMIR.ABS of whole document\n",
    "31.  LMIR.DIR of body\n",
    "32.  LMIR.DIR of anchor\n",
    "33.  LMIR.DIR of title\n",
    "34.  LMIR.DIR of URL\n",
    "35.  LMIR.DIR of whole document\n",
    "36.  LMIR.JM of body\n",
    "37.  LMIR.JM of anchor\n",
    "38.  LMIR.JM of title\n",
    "39.  LMIR.JM of URL\n",
    "40.  LMIR.JM of whole document\n",
    "41.  PageRank\n",
    "42.  Inlink number\n",
    "43.  Outlink number\n",
    "44.  Number of slash in URL\n",
    "45.  Length of URL\n",
    "46.  Number of child page\n",
    "\n",
    "Opis atributa i način na koji su generisani dostupni su u zvaničnoj dokumentaciji, više informacija dostupno je na strani [link](https://www.microsoft.com/en-us/research/project/letor-learning-rank-information-retrieval/). \n",
    "Lista atributa  sadrži različite karakteristike odnosa upita i dokumenta, npr. prvi atribut je frekvencija pojavljivanja reči iz upita u telu dokumenta, slično i drugi atributi opisuju odnos upita i dokumenta i svi su numeričkog tipa. Dalje sledi jedan primer para dokument-upit:\n",
    "\n",
    "\n",
    "\n",
    "0 qid:18219 1:0.052893 2:1.000000 3:0.750000 4:1.000000 5:0.066225 6:0.000000 7:0.000000 8:0.000000 9:0.000000 10:0.000000 11:0.047634 12:1.000000 13:0.740506 14:1.000000 15:0.058539 16:0.003995 17:0.500000 18:0.400000 19:0.400000 20:0.004121 21:1.000000 22:1.000000 23:0.974510 24:1.000000 25:0.929240 26:1.000000 27:1.000000 28:0.829951 29:1.000000 30:1.000000 31:0.768123 32:1.000000 33:1.000000 34:1.000000 35:1.000000 36:1.000000 37:1.000000 38:1.000000 39:0.998377 40:1.000000 41:0.333333 42:0.434783 43:0.000000 44:0.396910 45:0.447368 46:0.966667 #docid = GX004-93-7097963 inc = 0.0428115405134536 prob = 0.860366\n",
    "\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {
    "colab": {},
    "colab_type": "code",
    "id": "boQtW_gg9ePf"
   },
   "outputs": [],
   "source": []
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {
    "colab": {},
    "colab_type": "code",
    "id": "mFJitxaron5e"
   },
   "outputs": [],
   "source": []
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {
    "colab": {},
    "colab_type": "code",
    "id": "9NbiLxpAkKqp"
   },
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "colab": {
   "collapsed_sections": [],
   "name": "Copy of objasnjenje atributa.ipynb",
   "provenance": []
  },
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.8.3"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 1
}
