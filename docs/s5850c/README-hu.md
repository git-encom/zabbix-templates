# FS S5850C-24S2C – Zabbix 7.0 / SNMPv2c

Elkészült a `fs_s5850c_zabbix70.yaml` sablon első, a kapott SNMP-dumpokkal ellenőrzött változata. A célmodell az S5850C-24S2C, firmware 3.3.1.2.2, sysObjectID `1.3.6.1.4.1.52642.1.99.21252`.

Az ellenőrzés helyi YAML-beolvasást, hivatkozás-ellenőrzést, a discovery szabályok visszajátszását és az értékátalakítások futtatását tartalmazta. **Élő Zabbix-import és élő SNMP-lekérdezés még nem történt.** Az S5860-20SQ és az S5810-28TS támogatását ez a fájl nem igazolja; azok gyártói OID-jait külön kell felmérni.

## Mit figyel?

| Terület | Mérések és viselkedés |
|---|---|
| Rendszer | Leírás, sysObjectID, modell, firmware, sorozatszám, gyártói uptime, SNMP-elérhetőség |
| CPU | 5 másodperces, 1 perces és 5 perces átlag; tartós magas terhelés riasztása |
| Memória | Jelentett teljes/szabad/foglalt memória, foglalt/teljes arány és riasztás |
| Portok | Automatikus felismerés, alias a megjelenített névben, Counter64 be/kimenő forgalom bps-ban, jelentett sebesség, forgalomgrafikon, magas sávszélesség-kihasználtság riasztása |
| Hőmérséklet | Szenzoronkénti érték, alsó/felső/ kritikus küszöb, grafikon és küszöbökből képzett riasztások |
| Ventilátor | Állapot és a switch által jelentett sebesség százalékban; inaktív ventilátor riasztása |
| Tápegység | Jelenlét, üzemállapot, típus, hibajelzés; üzemhiba riasztása, opcionális hiányriasztás |
| Optika | Modul típusa, gyártója, cikkszáma, állapota; hőmérséklet, feszültség, bias, TX/RX dBm, modul által közölt négy küszöb; nyers DOM-értékek és optikai teljesítménygrafikon |
| Opcionális IF-MIB | Adminisztratív/üzemállapot, hibák és eldobott csomagok másodpercenként, link-down és hibaráta-riasztás. A discovery kezdetben kikapcsolt. |

Nincs szükség külső szkriptre, MIB-telepítésre vagy hivatkozott másik sablonra. Az OID-ok numerikusak. A lekérdezések olvasási műveletek.

## Import és első kipróbálás

1. Zabbix 7.0: **Data collection → Templates → Import**, válaszd a `templates/fs/s5850c/fs_s5850c_zabbix70.yaml` fájlt. Az import előnézetében a `FS S5850C by SNMP` sablonnak kell megjelennie. Első importnál nincs szükség „Delete missing” használatára.
2. Egy S5850C hoston adj meg SNMP interfészt: menedzsment IP, UDP 161, **SNMPv2**, a már használt read-only community. A sablon nem tartalmaz community értéket. Használható például host szintű `{$SNMP_COMMUNITY}` makró az interfész Community mezőjében.
3. Kapcsold hozzá a `FS S5850C by SNMP` sablont. Első kipróbálásra egy hostot válassz. Meglévő hálózati sablon mellett több mérés párhuzamosan is futhat, és a közös `zabbix[host,snmp,available]` kulcs ütközhet. Ilyenkor előbb tervezd meg, melyik sablon tartsa meg ezt az itemet; ne törölj meglévő történetet automatikusan.
4. A host Discovery listájában az öt bekapcsolt szabályra használható az **Execute now**. Az opcionális IF-MIB szabályt egyelőre hagyd kikapcsolva. A discovery ütemezése 1 óra, a legtöbb mérésé 1 perc; a rendszerazonosítóké 1 óra.
5. A **Monitoring → Latest data** nézetben ellenőrizd a CPU-t, memóriát, a két hőmérsékletet és a portforgalmat. A forgalmi ráta csak két egymást követő számlálóminta után jelenik meg.
6. Ellenőrizd az itemek és a két master walk hibaüzeneteit is. A master walk itemek nem tárolják a nyers dumpot (`history=0`), a függő itemek értékei viszont megmaradnak. Alapérték: 14 nap history, numerikus méréseknél 365 nap trend; a Zabbix housekeeping felülírhatja ezeket.

Importhiba esetén a Zabbix pontos hibaüzenete szükséges a javításhoz. Lekérdezési hiba esetén az érintett item neve, OID-ja és hibaüzenete elegendő; community-t ne küldj.

## Alapértelmezett riasztások és makrók

A CPU-, memória-, hőmérséklet-, ventilátor-, tápegység-üzemhiba-, sávszélesség- és SNMP-elérhetőségi riasztások aktívak. A portnevekre alapértelmezésben `^eth-` szűrés érvényes, így a dumpban szereplő VLAN- és loopback-interfészek kimaradnak. Ez a `{$FS.IF.NAME.MATCHES}` makróval módosítható.

| Makró | Alapérték | Jelentés |
|---|---:|---|
| `{$FS.CPU.WARN}` | 80 | CPU-küszöb, % |
| `{$FS.MEMORY.WARN}` | 90 | Jelentett foglalt/teljes memória küszöbe, % |
| `{$FS.IF.UTIL.WARN}` | 90 | Forgalom / jelentett portsebesség küszöbe, % |
| `{$FS.SNMP.TIMEOUT}` | 5m | SNMP-elérhetetlenség vizsgálati ablaka |
| `{$FS.TEMP.HYSTERESIS}` | 2 | Hőmérséklet-riasztás visszaállási margója, °C |
| `{$FS.IF.ERRORS.WARN}` | 2 | Hibaráta-küszöb, csomag/s; az opcionális discovery része |
| `{$FS.IF.CONTROL}` | 0 | Link-down riasztás engedélyezése |
| `{$FS.OPTICS.CONTROL}` | 0 | Optikai küszöbriasztások engedélyezése |
| `{$FS.PSU.PRESENCE.CONTROL}` | 0 | Tápegység-hiány riasztása |

A link-down és optikai riasztásokat érdemes az érintett portokra engedélyezni, a host Macros lapján például:

```text
{$FS.IF.CONTROL:"eth-0-1"} = 1
{$FS.OPTICS.CONTROL:"eth-0-1"} = 1
```

A link-down riasztás az opcionális IF-MIB discovery bekapcsolását is igényli. Adminisztratívan letiltott port nem riaszt; engedélyezett portnál tartós down állapot riaszt, akkor is, ha az első méréskor már down volt. Az optikai riasztások a modul alarm küszöbeit használják; a warning küszöbök külön itemként elérhetők. Sötét vagy használaton kívüli optikánál az alacsony RX érték önmagában lehet normális, ezért nincs minden porton automatikusan engedélyezve az optikai riasztás.

## A még hiányzó IF-MIB ellenőrzése

A `s5850c-interfaces.txt` az `ifXTable` adatait tartalmazza; ebből a forgalom és az `ifHighSpeed` ellenőrizhető. A portállapot, hibák és eldobott csomagok az `ifTable` alatt vannak, amelyet még nem kaptam meg.

A Zabbix szerverről/proxyról vagy egy Net-SNMP eszközökkel rendelkező Linux gépről az alábbi, célzott lekérdezés elegendő. A community és az IP helyére a saját beállítás kerüljön; a példa interaktívan bekéri a community-t.

```bash
read -rs -p 'SNMP community: ' fs_snmp_community; echo
fs_switch_ip='SWITCH_IP'
snmpwalk -v2c -c "$fs_snmp_community" -On -t 3 -r 1 \
  "$fs_switch_ip" .1.3.6.1.2.1.2.2.1 > s5850c-iftable.txt
unset fs_snmp_community
```

Ellenőrizendő oszlopok: `.7` adminStatus, `.8` operStatus, `.13` inDiscards, `.14` inErrors, `.19` outDiscards, `.20` outErrors. Ha ezek válaszolnak és az indexek egyeznek az `ifXTable` indexeivel, bekapcsolható a hoston az **Optional IF-MIB status/errors discovery (verify then enable)** szabály, majd **Execute now**. Ezután állítsd be a fontos portok `FS.IF.CONTROL` kontextusmakróit.

## Az adott firmware fontos sajátosságai

- Az `ENTITY-SENSOR-MIB` ág (`1.3.6.1.2.1.99.1.1`) a dumpban „No Such Object” választ adott. A hőmérséklet és ventilátor a gyártói `52642.1.37` ágban található.
- A ventilátor sebessége ténylegesen `40%` formájú. Bár a nyilvános MIB leírása RPM-et említ, a sablon a firmware által küldött százalékot kezeli.
- A hőmérséklet-szenzorokhoz nem érkezett értelmes elnevezés, ezért index szerint szerepelnek. Nem neveztem át őket önkényesen CPU-/ASIC-/házszenzorrá.
- Csak egy tápegységsor látszik. Ebből nem lehet két fizikai tápegység felügyeletét igazolni. A PSU-hiány makró csak a ténylegesen közzétett sorokra hat; nem észlel nem létező vagy soha nem közzétett második sort.
- A memTotalReal fizikai memóriát jelent; a MIB szerint a szabad/foglalt számlálók virtuális memóriát is tartalmazhatnak. A százalék ezért „Reported memory utilization”, nem garantáltan kizárólag fizikai RAM-terhelés.
- Az uptime TimeTicks körülbelül 497 nap után átfordulhat. A reset-riasztás kizárja az átfordulás előtti 15 perces tartományt, és „possible restart” eseményt jelez. Az SNMP-agent újraindulása is okozhat visszaállást.
- Az `ifHighSpeed` a jelentett sebesség. Egy down port névleges értékéből nem állapítható meg biztosan a hardver maximális sebessége.
- A kapott DOM-értékek egyetlen számot tartalmaznak. A MIB szerint QSFP esetén négy érték is érkezhet. A sablon a többcsatornás, üres vagy nem véges értékeket nem alakítja át önkényesen egy számmá: megőrzi őket a nyers itemekben, a numerikus minta kimarad. QSFP csatornánkénti felügyelethez külön minta kell. Az optikai riasztásokhoz friss numerikus mérés és küszöb szükséges.
- A firmware-csere után az indexeket, az értékformátumokat és az OID-okat újra ellenőrizni kell.

## S5860-20SQ stack és S5810-28TS

Ezekhez első körben külön-külön a következő célzott ágak kellenek:

```text
1.3.6.1.2.1.1              rendszerazonosítás, különösen sysObjectID
1.3.6.1.2.1.2.2.1         ifTable
1.3.6.1.2.1.31.1.1.1      ifXTable
1.3.6.1.2.1.47.1.1.1.1    ENTITY-MIB, stacktagok/alkatrészek
1.3.6.1.2.1.99.1.1        ENTITY-SENSOR-MIB, ha támogatott
1.3.6.1.2.1.25.3.3        HOST-RESOURCES CPU, ha támogatott
1.3.6.1.2.1.25.2.3        HOST-RESOURCES memória/tároló, ha támogatott
```

A `sysObjectID` alapján választható ki a helyes gyártói MIB és a további, szűk lekérdezési ág. Az S5860 két stacktagjánál a tagonkénti CPU/memória, ventilátor, tápegység és eltűnés külön ellenőrzést igényel. A CLI uptime önmagában nem igazolja, hogy az SNMP ugyanazt az uptime-ot küldi vagy melyik tagét látjuk.

**A teljes gyártói dump hitelesítési adatokat is tartalmazott. Ne tedd nyilvánossá.** A sablon és a mellékelt ellenőrzési jegyzőkönyv nem tartalmazza ezeket az értékeket, és nem kérdezi le a hitelesítési/konfigurációs ágakat. A következő körben célzott dumpokat használjunk.

## Milyen kész sablonok találhatók?

Van nyilvános FS sablon, de nem igazoltan ehhez a három pontos modell/firmware pároshoz:

- A [Zabbix FS integrációs oldalán](https://www.zabbix.com/integrations/fs) az FS S3900R sorozat közösségi sablonja szerepel, Zabbix 7.0+ jelöléssel. A leírás szerint S3900-24T4S-R készüléken készült; ebből nem következik S5850C-kompatibilitás.
- Az [FS saját útmutatója](https://resource.fs.com/mall/doc/20230601120410iwbaf4.pdf) az `FS-S5860` sablon importját dokumentálja, Zabbix 4.4.4 és S5860-24XB-U példával. A régi `Template Module Interfaces SNMPv2` függőséget is megemlíti. Ez nem S5860-20SQ stack és nem ellenőrzött Zabbix 7.0 export.
- A saját sablon OID-jaihoz a [nyilvános SWITCH MIB](https://github.com/librenms/librenms/blob/master/mibs/fs/centec/SWITCH) szolgált szemantikai forrásként, a tényleges jelenlétet és formátumot a referencia-SNMP-dumpok igazolták.
- Az export és a natív preprocessing kialakításának referenciája az [eredeti Zabbix 7.0 hálózati sablon](https://github.com/zabbix/zabbix/blob/release/7.0/templates/net/generic_snmp/template_net_generic_snmp.yaml), a [7.0 importvalidator](https://github.com/zabbix/zabbix/blob/release/7.0/ui/include/classes/import/validators/C70XmlValidator.php) és a [preprocessing dokumentáció](https://www.zabbix.com/documentation/7.0/en/manual/config/items/preprocessing).
