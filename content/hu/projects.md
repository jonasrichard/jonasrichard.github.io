+++
template = "hu-index.html"
title = "Projektjeim"

[extra]
lang = "hu"
+++
# Projektjeim

### Vámraktár nyilvántartó

‟Az első webes projektemen megtanultam mennyire fontos a jó teljesítmény és a
pontosság„

Igazi LAMP project (Linux, Apache, MySQL, PHP), ahogy mind a négy
technológiát megismertem. Szüleim import-export kereskedelmi vállalkozásához
fejlesztettem ezt a szoftvert, ami a havi ÁFA fizetés alapja volt. Mondanom sem
kell, hogy a pénzügyi felelősség itt óriási volt, ez nem egy kockázati tőkes
pénze volt... Szóval aki kételkedik, hogy komoly intranetes rendszert lehet-e
fejleszteni ezekkel a technológiákkal, próbálja ki maga is.

A vámraktár jelentéseket LaTeX által generált Postscript dokumentumokba
generálta a rendszer, a vámosok gyönyörködtek a lézernyomtató által nyomtatott
reportokban.

### ejson JSON konverter

A Java világban számtalan eszköz könnyíti meg a programozók életét ha adatokat
valamilyen külső formában akarnak tárolni (adatbázis, bináris file, web).
Töménytelen számú annotáció alapú JSON konverter létezik.

Erlang-ban egy kicsit más a helyzet, ez olyan csináld magad világ, mindenki a
saját kényelmére szab egy low-level JSON konvertert. Én erre találtam ki egy
deklaratív keretrendszert, amivel szabályokat meghatározva az ejson
automatikusan átkonvertálja az Erlang adatokat JSON formátumba. És ami a múltban
még fájdalmasabb volt, JSON-ből automatikusan konvertál Erlang termekbe -
hibaellenőrzés mellett.

Ez egy open source projekt, a github-on hosztolom.
