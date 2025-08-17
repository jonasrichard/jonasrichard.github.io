+++
template = "hu-index.html"
title = "Magamról"

[extra]
lang = "hu"
+++
# Egy kicsit magamról

Jónás Richárd vagyok, jelenleg szoftver architektként dolgozom olyan
megoldásokon, ahol fontos a teljesítmény, a skálázhatóság, mind horizontálisan,
mind vertikálisan.

1976-ban Debrecenben születtem, és 10 éves koromban kaptam kézhez a Basic
programnyelv könvyet, valamiért már az elejétől fogva nagyon érdekesnek
találtam. Később amikor 12 éves lettem, megkaptam az első számítógépemet,
egy Commodore Vic-20-as modellt, 3583 byte memóriával (igen 4KB-nál kisebb
volt) használtan. Itt megtanultam az elemi komputer-grafikát, vonalak és
körök rajzolását programoztam, és sokat nézegettük az akkori demo-k grafikáját.

A Mechwart András szakközépiskolában folytattam a tanulmányaimat, itt habár
a képzés nevében benne volt a számítástechnika, nem sok mident tanítottak ezen
a területen, itt gépipari alapismereteket tanultam.

Ezután a Debreceni Egyetemre mentem továbbtanulni 1995-ben, ekkor még a
Kossuth Lajos Tudományegyetem néven működött. Itt sok formális matematika és
logika mellett megtanultuk a számítástudomány (computer science) alapjait.

## A szoftverek világa

Szóval amióta világ a világ engem mindig is a szoftverfejlesztés érdekelt.
2000-ben végeztem a Debreceni Egyetemen programtervező-matematikus szakon, de
már jóval előtte is fejlesztettem szoftvereket. A C, C++ nyelvek fogtak meg
először, ahol megtanultam a feladatok komplexitása mellett kezelni hogyan kell
hardverközeli kódokat írni. Ez nem állt tőlem távol, mert évekkel előtte
Assembly nyelven is programoztam.

Diplomamunkámnak a morfológiai zajszűrést választottam a digitális
képfeldolgozásban, ebben az időben több képfeldolgozó szoftvert is
fejlesztettem a fent említett könyezetben. Diploma után rövidesen a
Java programozási nyelv fogott meg, és szerveroldalon körülbelül 10 évig
programoztam különböző rendszereket Java nyelven és a köré épülő
technológiákkal (EJB, Spring, Tomcat, Hibernate). Időközben
megismerkedtem egy projekt keretében az Erlang programozási nyelvvel,
és a lightweigh process modellje nagyon megtetszett. Ezután már máshogy láttam
a párhuzamosságot és a hibakezelést a Java programokban.

2013-ban az Erlang Solutions budapesti irodájában helyezkedtem el, és
innentől kezdve az Erlang programozási nyelv és a hozzá kötődő technológiák
a szakterületem. A Riak NoSql adatbáziskezelő segítünk integrálni, hangolni
különböző európai és amerikai nagyvállalatok számára.

### Szerverek

Már az első percektől jobban érdekelt, hogy mi zajlik a masszívan dolgozó
szervereken a háttérben, mint a frontend fejlesztés. Természetesen a Web és a
benne rejlő lehetőségek egyből megfogtak, így kézenfekvő volt, hogy webes
alkalmazásokat fejlesszek.

A szerveroldalon azt találom kihívásnak, hogy a rendelkezésre álló
kapacitásokkal takarékosan bánva, minél több felhasználót tudjunk kiszolgálni.
Ugyanakkor a szerveren megadatott kapacitásokat 100%-osan kell kihasználnunk,
tehát olyan szoftvert kell írni, ami takarékos, de mégis maximálisan ki tudja
hajtani a szervert (hiszen azért költöttek rá pénzt).

Ezt a legjobban párhozamos és konkurrens programozással tudjuk elérni. A mai
funkcionális programnyelvek párhuzamossága pontosan ezt hivatott megvalósítani
(Erlang aktor modell, illetve később Scala, Clojure, stb.)

### Párhuzamosság

Egy megadott algoritmust leprogramozni nem bonyolult feladat. Viszont olyan
algoritmusokat kitalálni, amelyek lehetővé teszik, hogy a folyamatokat
párhuzamosan lehessen végrehajtani, már komoly szellemi erőfeszítést igényelnek.
Agyunk alapvetően szekvenciális, tehát egy feladatot lépések sorozatával a
legkézenfekvőbb megoldani.

A párhuzamos programozás ezzel szembemegy azzal, hogy itt a folyamatokat
egymástól kvázi-függetlenül végrehajtható lépésekre bontja. Ez nem minden
esetben tehető meg, és ráadásul egy újszerű gondolkodást is igényel. A zip
tömörítő algoritmust például szekvenciális, tehát akárhány magos laptopot is
veszünk, ez a folyamat nem fog felgyorsulni.

Masszívan konkurrens rendszereket máshogyan is kell tervezni. Itt a feladatot,
sok kicsi, egymástól független életciklussal rendelkező folyamatra bontjuk. Ezek
a folyamatok beszélgetnek egymással - igény szerint, és mindenképpen külön-külön
halnak meg, abnormális körülmények esetén. Sokan nem értik, hogy tervezés közben
a hiba lokalizációja határozza meg, hogy miket fognak ezek a folyamatok
elvégezni (Erlang - Let it crash filozófia).

### Tesztelés, mérés

Ha valamit el tud végezni egy számítógép, miért mi végezzük el? A szoftver
folyamatosan fejlődik. Új igények jelennek meg, amire reagálnunk kell új
funkciók leprogramozásával. Változhatnak régóta stabil funkciók, adatok, az új
funkciók bekerülésével. Ennek végigellenőrzése olyan tevékenység, amit nagyon
gazdaságtalan emberi erőre bízni.

Mindig is úgy gondoltam, hogy a szoftver minőségét szoftveres tesztekkel a
legkönnyebb elfogadható szinten biztosítani. Szeretek jól tesztelhető, jól
mérhető kódokat írni, hiszen a tesztelés alapvetően fejlesztési időben
ellenőrzni a program működését, de élesben muszáj diagnosztikai pontokat építeni
a szoftverünkbe, hogy beavatkozás nélkül meg tudjuk mondani melyik részével
vannak kifogásaink. Ezáltal a hibát is könnyebben tudjuk lokalizálni.
