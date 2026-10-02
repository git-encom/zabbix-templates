# S5850C-24S2C OID-térkép

Célfirmware: 3.3.1.2.2. `B = 1.3.6.1.4.1.52642.1`, `i = teljes SNMP-index`. Az összetett indexeket a sablon változtatás nélkül használja. Minden alapértelmezésben bekapcsolt SNMP adat-OID megtalálható volt a kapott fájlokban vagy a beszélgetésben megadott rendszerazonosításban.

| Adat | OID | Átalakítás / jelentés |
|---|---|---|
| sysDescr | `1.3.6.1.2.1.1.1.0` | Szöveg |
| sysObjectID | `1.3.6.1.2.1.1.2.0` | OID-szöveg |
| Firmware | `1.3.6.1.2.1.47.1.1.1.1.10.1` | Szöveg |
| Sorozatszám | `1.3.6.1.2.1.47.1.1.1.1.11.1` | Szöveg; tényleges érték nem szerepel ebben a jegyzékben |
| Modell | `1.3.6.1.2.1.47.1.1.1.1.13.1` | Szöveg |
| Uptime | `B.1.3.10.0` | TimeTicks / 100 → s |
| CPU 5s / 1m / 5m | `B.1.9.1.0` / `.2.0` / `.3.0` | Egyetlen százalékos szöveg, `%` eltávolítása, 0–100 ellenőrzés |
| Teljes memória | `B.1.1.5.0` | kB × 1024 → B |
| Szabad memória | `B.1.1.11.0` | kB × 1024 → B |
| Foglalt memória | `B.1.1.12.0` | kB × 1024 → B |
| Memóriaarány | Számított | 100 × foglalt / teljes |
| Portnév / alias | `1.3.6.1.2.1.31.1.1.1.1.i` / `.18.i` | Discovery |
| HCInOctets / HCOutOctets | `1.3.6.1.2.1.31.1.1.1.6.i` / `.10.i` | Natív Counter64 változás/s × 8 → bps |
| ifHighSpeed | `1.3.6.1.2.1.31.1.1.1.15.i` | Mbps × 1 000 000 → bps |
| Ventilátor állapot | `B.37.1.1.1.1.4.i` | 1 aktív, 2 inaktív, 3 nincs telepítve, 4 nem támogatott |
| Ventilátor sebesség | `B.37.1.1.1.1.5.i` | A firmware százalékos szöveget küld; nem RPM |
| PSU jelenlét | `B.37.1.2.1.2.i` | 1 jelen, 2 hiányzik, 3 nem támogatott |
| PSU üzemállapot | `B.37.1.2.1.3.i` | 1 aktív, 2 inaktív, 3 nincs telepítve, 4 nem támogatott |
| PSU típus | `B.37.1.2.1.4.i` | 1 AC, 2 DC, 3 ismeretlen, 4 nincs telepítve, 5 nem támogatott |
| PSU hibajelzés | `B.37.1.2.1.7.i` | 1 nincs hiba, 2 hiba, 3 nem támogatott |
| Hőmérséklet | `B.37.1.3.1.4.i` | °C; csak `1.` kezdetű, hőmérséklet típusú index |
| Kritikus / felső / alsó hőmérsékletküszöb | `B.37.1.3.1.5.i` / `.6.i` / `.7.i` | °C, szenzoronként külön; negatív érték is megengedett |
| Optika típus / gyártó / cikkszám | `B.37.1.10.1.1.1.i` / `.2.i` / `.3.i` | Discovery: nem üres cikkszámú modulok |
| Optika állapot | `B.37.1.10.1.1.11.i` | `active` / `inactive` szöveg |
| DOM hőmérséklet | `B.37.1.10.2.1.c.i` | °C |
| DOM tápfeszültség | `B.37.1.10.3.1.c.i` | V |
| DOM laser bias | `B.37.1.10.4.1.c.i` | mA |
| DOM TX / RX | `B.37.1.10.5.1.c.i` / `B.37.1.10.6.1.c.i` | dBm; nincs további logaritmikus konverzió |

A DOM-táblákban `c`: 1 magas alarm, 2 alacsony alarm, 3 magas warning, 4 alacsony warning, 5 aktuális érték. A numerikus átalakítás pontosan egy véges számot fogad el, a környező szóközöket levágja. A jelenlegi értékhez külön nyers szöveges item is tartozik.

## Opcionális, még nem igazolt standard OID-ok

Külön, alapértelmezésben kikapcsolt discovery szabályhoz tartoznak. `I = 1.3.6.1.2.1.2.2.1`:

| Adat | OID | Átalakítás |
|---|---|---|
| ifAdminStatus | `I.7.i` | 1 up, 2 down, 3 testing |
| ifOperStatus | `I.8.i` | IF-MIB állapottérkép |
| ifInDiscards / ifOutDiscards | `I.13.i` / `I.19.i` | Számlálóváltozás/s → pps |
| ifInErrors / ifOutErrors | `I.14.i` / `I.20.i` | Számlálóváltozás/s → pps |

## Forrás

A gyártói elnevezések, enumok és alapegységek forrása a [SWITCH MIB](https://github.com/librenms/librenms/blob/master/mibs/fs/centec/SWITCH). Az adott firmware válaszai a mérvadók, ahol a MIB és az adatformátum eltér; ilyen a ventilátor százalékos sebessége.
