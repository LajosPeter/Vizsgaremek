https://www.lsoft.hu/wp-content/uploads/2023/10/IBF-1-0043-IT-szolgaltataskatalogus-publikus.pdf
innen fel lehet e használni a címeket igen lehet 


# Szolgáltatások


Szolgáltatási igény felmérése telephelyenként 

>Bele kell írni a pontos felhasználókat, mennyiségét és biztonsági technikák minimális jellemzése. 

Az Avalon bentlakásos Gimnázium megbízott minket, hogy a  négy telephelyre tervezzün meg és telepítsük az eszközöket és configuráljuk a hálózatot: "Gimnázium", "Tankerület", "Bentlakásos kollégium", "Sportpálya". Emelett készítsük el a szerver szol
A Gimnáziumba ``FIXME`` minden egyes alkalmazottnak van külön IP cím alapján működő telefonja. A gazdasági osztálynál 5 alkalmazott található és egy nyomtató. Az igazgató helyettesi irodában 1 alkalmazott és egy nyomtató van.  Az igazgatói irodában, a portán és a rendszergazdánál csak egy alkalmazott található. Három tanári teremet is ki kell alakítani ahol termenként 10 gépet kell elhelyezni telefonnal. Emellett igény, hogy a teljes épületben legyen WiFi lefedettség a tanárok számára. Emelett fontos, hogy autómatikusan kapják meg az IP címeket a hálózatban a számítógépek és csakis a Gimnázium eszközei férhessenek hozzá a hálózathoz. Igény továbbá, hogy el lehessen érni a távoli szervereken a szolgáltatásokat. (Bővebben a Szerverek pontban)


A Tankerületben ``FIXME`` ezen a telephelyen csak a 3 csoport vezetőknek, a recepciósnak, a kancellárnak, a kancellár helyettesnek és a rendszergazdának van IP cím alapján működő telefonjuk és 3 darab nyomtató található az irodában amelyek a csoport vezetők mellé vannak elhelyezve. A rendszergazdát, a Kancellárt és Kancellár helyettest biztonság technikailag le kell választani a többi felhasználótol. Emellett WiFi lefedés kell az irodában. A sima irodai dolgozók autómatikus címkiosztással kapjanak IP címet viszont a Rendszergazda, a Kancellár, a Kancellár helyettesnek. Igény továbbá, hogy el lehessen érni a távoli szervereken a szolgáltatásokat. (Bővebben a Szerverek pontban) Fontos, hogy a távmunkában dolgozó alkalmazottak is elérhessék a fájlszervert, hogy tudjanak dolgozni.

A Sportpálya telephelyen csak kettő számítógépre van szükség hogy tudják vezetni a diákok részvételét az órán és a sporteszközök leltárának vezetésére.

Az utolsó telephelyen a Bentlakásos kollégiumban ``FIXME`` szeretnének WiFi lefedést a diákoknak és a nevelőknek. Emelett igény az is, hogy a diákok ne férhessenek hozzá rosszindulatú weboldalakhoz azonban a nevelőkre ne vonatkozzon semmilyen korlátozás. Igény továbbá, hogy el tudják érni a Gimnázium fájltároló szerverét, hogy fel tudják tölteni a megoldott feladataikat. Feladat továbbá egy olyan átfogó hálózatbiztonsági megoldás kifejlesztése, amely biztosítja a hálózat védelmét. 



A Servereknél a a gimnáziumban
Az gimnáziumnak weboldalra is szüksége van és igény, hogy ne IP címmel kelljen hozzá férni a webhelyeikhez azért, hogy ne kelljen megjegyezni az IP címeket. Emelett megbízhatóan integrálniuk kell a meglévő bérelt adattároló szervert a rendszerükbe. Ezen felül egy telefon központot is ki kell alakítani a Gimnázium  és a Tankerület számára.

Fontos az is, hogy a címkiosztás úgy történjen, hogy mind a jelenlegi, mind a jövőbeli címzési protokollokat támogassa, ezáltal biztosítva a megfelelő jövőbeli fejlesztés lehetőségét.
  A szolgáltatási környezet felmérése
A tervezést megalapozó adatok
Szervezeti és létszám adatok

Katalógus adatok 
Üzleti szolgáltatás katalógus
## E-mail 

A megrendelő cég egy e-mail szolgáltatást kért tőlünk, amely megbízható, biztonságos, és támogatja a vállalati kommunikációt. Ezt mi egy központi e-mail szerverrel tudtuk megoldani ami felhő alapú tárolásra alkalmas így nem kell az alkalmazottaknak a helyi gépen tárhelyet biztosítani. Minden alkalmazottnak biztosítunk egy céges e-mail címet illetve egyéb e-mail címeket is igény esetén.

### Szolgáltatás részletes leírása:

- A szolgáltatás célja egy belső hálózaton működő, biztonságos e-mail szerver kialakítása és folyamatos üzemeltetése. Az e-mail szerver kizárólag a belső hálózaton elérhető, külső kapcsolatokhoz VPN vagy tűzfalon keresztüli szabályozott elérés szükséges.

### A szolgáltatás elemei:

#### E-mail szerver telepítés

 - A szerver operációs rendszere: Windows Server (igény szerint)
 - A szerver fix belső IP-címmel rendelkezik, amelyet a belső DNS-ben is regisztrálni kell

#### Hálózati konfiguráció:
- Cisco routereken és switcheken keresztül a megfelelő VLAN-konfiguráció biztosítja az e-mail forgalmat

- A tűzfalon (Cisco ASA-n) a levelezéshez szükséges portok megnyitása


#### Biztonsági beállítások:

- TLS titkosítás használata az e-mail forgalom védelmére
- Spam és malware szűrők beállítása
- Két faktoros autentikáció (2FA) a webmail eléréshez (igény esetén)

#### Karbantartás és felügyelés:
- Naplózás és rendszeres riportok generálása a szerver teljesítményéről
- Havi biztonsági frissítések és rendszeres mentések elvégzése

#### Szolgáltatási időszak:
- 365*24 óra
- 99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc



    
## VoIP

A megrendelő cég egy megbízható VoIP szolgáltatást igényeltek cégünktől, amely lehetővé teszi a költséghatékony és rugalmas hangkommunikációt. Cégünk egy stabil, kiválló minőségű kommunikációt biztosít és így zökkenőmentesen folyhatnak az üzleti hívások.

### VoIP rendszer üzemeltetés:

#### Hálózati konfiguráció: 

- VLAN-ok kialakítása a hangforgalomnak, QoS szabályok beállítása.
  
#### Biztonsági beállítások: 
- tűzfal konfiguráció

#### Szolgáltatás időszak:
- 365*24 óra
- 99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc


## Fájl szerverek (ftp/tftp)

Az igényfelmérés alapján a vállalat egy biztonságos és megbízható fájlok tárolására alkalmas szolgáltatást igényel, amely lehetővé teszi a nagyobb fájlok hatékony átvitelét és tárolását. Ebben cégünk egy FTP szervert konfigurálna ami lehetővé teszi a követelményeknek a teljesítését.

### Szolgáltatás részletes leírása:
- A szolgáltatás célja a fájlmegosztás és konfigurációkezelés biztosítása FTP és TFTP szervereken keresztül. Ezek a szerverek lehetőséget nyújtanak a fájlok biztonságos tárolására és gyors elérésére a belső hálózaton keresztül

### Szolgáltatási elemek:

#### FTP szerver telepítés és konfiguráció:
- Felhasználóalapú hozzáférés biztosítása és jogosultságkezelés

#### Karbantartás és felügyelés:
- Rendszeres frissítések és sérülékenység-ellenőrzés
- Havi mentések és szükség esetén gyors visszaállítás biztosítása

#### Szolgáltatás időszak:
- 365*24 óra
- 99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc


## Adatbázisszolgáltatás (mariandb)

Az igényfelmérés alapján a megbízó cég kért tőlünk a pénzügyi illetve egyéb információk átlátható tárolására alkalmas szolgáltatást. Cégünk egy adatbázis szolgáltatást nyújt aminek a segítségével a megbízó cég képes jogosultsághoz kötve hozzáférni az adatokhoz és átláthatóan egy adatbázisban tárolni az adatokat.

### Szolgáltatás részletes leírása:
- A szolgáltatás célja egy megbízható, nagy teljesítményű MariaDB adatbázis-szerver biztosítása, amely támogatja az alkalmazásokat, szolgáltatásokat ésadatokat

### Szolgáltatási elemek:

#### MariaDB szerver telepítés és konfiguráció:
- Operációs rendszer: Debian Server
- Optimalizált beállítások a teljesítmény és skálázhatóság érdekében

#### Hálózati konfiguráció:
- Cisco hálózati eszközök integrációja, VLAN és QoS beállítások
- Tűzfal és ACL szabályok az adatbázis-hozzáférés védelmére

#### Karbantartás és felügyelés:

- Teljesítménymonitorozás és hibaészlelés (Zabbix)
- Automatikus és ütemezett biztonsági mentések
- Verziófrissítések és sérülékenység-ellenőrzés

#### Szolgáltatás időszak:
- 365*24 óra
- 99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc


## Backupmentés

A megbízó cég megbízott minket azzal hogy esetleges emberi vagy technikai hibák esetén ne vesszen el nekik semmi adat így cégünk konfigurált egy olyan szolgáltatást ami meghatározott időnként készít egy biztonsági mentést a fontos illetve a cég által meghatározott adatokról.

### Szolgáltatás részletes leírása:
- A szolgáltatás célja az üzletileg kritikus adatok rendszeres mentése az előbb említett FTP szerverre, biztosítva az adatok visszaállíthatóságát egy esetleges adatvesztés esetén

### Szolgáltatási elemek:

#### Biztonsági mentési stratégiák: 
- Teljes mentés alkalmazása
  
#### Titkosítás és adatvédelem: 
- Az adatok titkosított formában való tárolása az FTP szerveren

#### Karbantartás és felügyelés:
- Rendszeres ellenőrzés, hogy a mentések sikeresen létrejöttek-e

#### Szolgáltatás időszak:
- 365*24 óra
- 99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc
 
## VPN tunnel

A válalat megbízott minket egy olyan szolgáltatással ami leheővé teszi számukra hogy a külső szervereket illetve a telephelyeket összekötő útvonal biztonságos és titkos maradjon. Mi ezt egy úgynevezett IPSecVPN nevezetű szolgáltatással tudjuk megoldani ami számukra a lehető legmegfelelőbb megoldást nyújtja.

### Szolgáltatás részletes leírása:
-A szolgáltatás célja biztonságos és titkosított távoli hozzáférés biztosítása a telephelyek hálózatai között illetve lehetővé teszi a felhasználók számára, hogy külső helyszínekről biztonságosan csatlakozzanak a belső rendszerekhez

### Szolgáltatási elemek:

#### VPN szerver telepítés:
- IPSec alapú megoldás

#### Hálózati konfiguráció: 
- VPN kapcsolat VLAN és tűzfal beállításokkal.

#### Karbantartás és felügyelés:
- VPN kapcsolatok figyelése, naplózása és hibaelhárítása.

#### Szolgáltatás időszak:
- 365*24 óra
- 99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc

## Webszerver

A megbízó cég az igényfelmérés során megemlítette hogy saját weboldalt szeretne fentartani pár darab al oldallal együtt. Cégünk egy webszerver szolgáltatást nyújt hogy teljesítsük a megbízó cég kérését és egy fix ip-című webszervert hozunk létre számukra ami igényel egy úgynevezett DNS névfordítást ami lehetővé teszi hogy URL alapján megtalálják a cég honlapját bárhonnan.

### Szolgáltatás részletes leírása:
- A szolgáltatás célja egy megbízható webszerver biztosítása, amely támogatja a belső weboldalakat.
### Szolgáltatási elemek:

#### Webszerver telepítés: 
- Apache, Nginx vagy Microsoft IIS alapú rendszer (igény szerint)

#### Biztonsági beállítások: 
- SSL/TLS titkosítás konfiguráció.

#### Szolgáltatás időszak:
- 365*24 óra
- 99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc

## Rendszerszolgáltatási katalógus
    
|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**DHCP:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|A kol telephelyen a felhasználóknak a srv-gim-01 látja el és a telefonokat a ssw-gim-01. A tank telephelyen a felhasználóknak a srv-tank-01 látja el és a telefonokat a ssw-tank-01. A kol |
|Szolgáltatási korlát|A dinamikusan kiosztott címek maximum 64 hostot tartalmaz|
    

|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**DNS:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|Tankerület srv-gim-01. Adatközpont srv-dc-01. Tankerület srv-tank-01|
|Szolgáltatási korlát|A DNS szerver az összes szerver és indokolt eszköz névfeloldásáért felel |



|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**NTP:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze||
|Szolgáltatási korlát||




|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**IMAP:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|A hálózatban található ISP routerek rtr-telekom-core-01,-02,-03,-04|
|Szolgáltatási korlát|A telekom szolgáltató álltal biztosított routerek megbízhatósága limitál minket|


    
|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**Autómatizált szoftvert telepítés:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|Gimnázium telephelyen srv-gim-01. Tankerület telephelyen srv-tank-01|
|Szolgáltatási korlát|Egy tartományba léptetjük a telephely gépeit és így lehetséges az automatikus szoftvertelepítés|
    


|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**Tűzfal:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|Adatközpont telephelyen fw-dc-01|
|Szolgáltatási korlát|Minden lehetséges támadást illetve illetéktelen behatolást próbáljuk kiszűrni|


|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**Directory Services:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|A gimnázium telephelyen: srv-gim-01 A Tankerület telephelyen: srv-tank-01 A maradéknak a srv-dc-01,-02|
|Szolgáltatási korlát|Minden alkalmazottnak saját mappát biztosítunk különböző kapacitással:|
|| Alkalmazottak: 50GB szabadon felhasználható terület|
|| Vezetők: 50GB szabadon felhasználható terület|
|| Igazgatóság 100GB szabadon felhasználható terület|



|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**Wi-Fi szolgáltatások:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|A Gimnázium telephelyen: ap-gim-01,-02,-03,-04. A Tankerület telephelyen ap-tank-01,-02,-03. A Bentlakásos kollégiumban wlc-kol-01 irányításával ap-kol-01,-02,-03,-04|
|Szolgáltatási korlát|Több vezeték nélüli hálózatot biztosítunk vendégek és dolgozók számára egyaránt|


|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**Switching és Routing:**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|A Gimnázium telephelyen: rtr-gim-01,-02,-03; sw-gim-01,-02,-03; ssw-gim-01. A Tankerület telephelyen rtr-tank-01,-02,-03; sw-tank-01,-02,-03,-04,-05,-06; ssw-tank-01. A Bentlakásos kollégiumban rtr-kol-01; sw-kol-01. A Sportpályán rtr-sp-01; sw-sp-01. A hálózatnak pedig a telekom routerei rtr-telekom-core-01,-02,-03,-04|
|Szolgáltatási korlát|Dinamikus illetve statikus forgalomirányítás mellett lehet választani illetve 2. és 3. rétegbeli switch között|


|Szolgáltatás megnevezése|Szolgáltatást leíró adatok|
|---|---|
|**AAA (Authentication, Authorization, Accounting):**||
|Szolgáltatási időszak|365*24 óra|
|Rendelkezésre állás|99,9% (max. szolg. kiesés 8,76 óra/év) Egyszerre maximum 43,8 perc|
|A szolgáltatás felelős eszköze|A linux szerver látja el ezt a feladatot amely az Adatközpont telephelyen található srv-dc-02|
|Szolgáltatási korlát|nagy terhelés mellett skálázási problémák léphetnek fel|
