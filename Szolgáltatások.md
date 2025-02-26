https://www.lsoft.hu/wp-content/uploads/2023/10/IBF-1-0043-IT-szolgaltataskatalogus-publikus.pdf
innen fel lehet e használni a címeket?


# A feladat leírása

>Bele kell írni a pontos felhasználókat, mennyiségét és biztonsági technikák minimális jellemzése. 

### Az Avalon bentlakásos Gimnázium megbízott minket, hogy a  négy telephelyre tervezzün meg és telepítsük az eszközöket és configuráljuk a hálózatot: "Gimnázium", "Tankerület", "Bentlakásos kollégium", "Sportpálya". Emelett készítsük el a szerver szolgáltatást.

### A Gimnáziumba ``FIXME`` minden egyes alkalmazottnak van külön IP cím alapján működő telefonja. A gazdasági osztálynál 5 alkalmazott található és egy nyomtató. Az igazgató helyettesi irodában 1 alkalmazott és egy nyomtató van.  Az igazgatói irodában, a portán és a rendszergazdánál csak egy alkalmazott található. Három tanári teremet is ki kell alakítani ahol termenként 10 gépet kell elhelyezni telefonnal. Emellett igény, hogy a teljes épületben legyen WiFi lefedettség a tanárok számára. Emelett fontos, hogy autómatikusan kapják meg az IP címeket a hálózatban a számítógépek és csakis a Gimnázium eszközei férhessenek hozzá a hálózathoz. Igény továbbá, hogy el lehessen érni a távoli szervereken a szolgáltatásokat. (Bővebben a Szerverek pontban)


### A Tankerületben ``FIXME`` ezen a telephelyen csak a 3 csoport vezetőknek, a recepciósnak, a kancellárnak, a kancellár helyettesnek és a rendszergazdának van IP cím alapján működő telefonjuk és 3 darab nyomtató található az irodában amelyek a csoport vezetők mellé vannak elhelyezve. A rendszergazdát, a Kancellárt és Kancellár helyettest biztonság technikailag le kell választani a többi felhasználótol. Emellett WiFi lefedés kell az irodában. A sima irodai dolgozók autómatikus címkiosztással kapjanak IP címet viszont a Rendszergazda, a Kancellár, a Kancellár helyettesnek. Igény továbbá, hogy el lehessen érni a távoli szervereken a szolgáltatásokat. (Bővebben a Szerverek pontban) Fontos, hogy a távmunkában dolgozó alkalmazottak is elérhessék a fájlszervert, hogy tudjanak dolgozni.

### A Sportpálya telephelyen csak kettő számítógépre van szükség hogy tudják vezetni a diákok részvételét az órán és a sporteszközök leltárának vezetésére.

### Az utolsó telephelyen a Bentlakásos kollégiumban ``FIXME`` szeretnének WiFi lefedést a diákoknak és a nevelőknek. Emelett igény az is, hogy a diákok ne férhessenek hozzá rosszindulatú weboldalakhoz azonban a nevelőkre ne vonatkozzon semmilyen korlátozás. Igény továbbá, hogy el tudják érni a Gimnázium fájltároló szerverét, hogy fel tudják tölteni a megoldott feladataikat. Feladat továbbá egy olyan átfogó hálózatbiztonsági megoldás kifejlesztése, amely biztosítja a hálózat védelmét. 



### A Servereknél a a gimnáziumban
Az gimnáziumnak weboldalra is szüksége van és igény, hogy ne IP címmel kelljen hozzá férni a webhelyeikhez azért, hogy ne kelljen megjegyezni az IP címeket. Emelett megbízhatóan integrálniuk kell a meglévő bérelt adattároló szervert a rendszerükbe. Ezen felül egy telefon központot is ki kell alakítani a Gimnázium  és a Tankerület számára.

Fontos az is, hogy a címkiosztás úgy történjen, hogy mind a jelenlegi, mind a jövőbeli címzési protokollokat támogassa, ezáltal biztosítva a megfelelő jövőbeli fejlesztés lehetőségét.




# Szolgáltatási igény felmérése telephelyenként 

"Gimnázium"


"Tankerület"


"Bentlakásos kollégium"


"Sportpálya"


  
# A szolgáltatási környezet felmérése - fizikiai tervezet



A tervezést megalapozó adatok
Szervezeti és létszám adatok



Katalógus adatok 
Üzleti szolgáltatás katalógus
    E-mail
    VoIP
    Fájl szerverek (ftp/tftp)
    Adatbázisszolgáltatás (mariandb)
    Backupmentés
    VPN tunnel
    Webszerver


Rendszerszolgáltatási katalógus
    DHCP
    DNS
    NTP
    IMAP
    Autómatizált szoftvert telepítés
    Tűzfal
    Directory Services
    Wi-Fi szolgáltatások
    Switching és Routing
    AAA (Authentication, Authorization, Accounting)



