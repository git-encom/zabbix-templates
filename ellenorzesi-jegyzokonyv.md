# Ellenőrzési jegyzőkönyv – FS S5850C sablon 1.0.0

Dátum: 2026-10-02. Cél: Zabbix 7.0, SNMPv2c, S5850C-24S2C, firmware 3.3.1.2.2.

## Eredmény

A helyi ellenőrzések sikeresek. Az öt bekapcsolt discovery szabály a kapott walkokból a következő sorokat választotta ki:

| Terület | Felismert sorok |
|---|---:|
| Fizikai interfész | 26 |
| Ventilátor | 2 |
| Tápegység | 1 |
| Hőmérsékletszenzor | 2 |
| Optikai modul | 7 |

Az egy tápegységsor a közzétett SNMP-tábla tartalma; nem állítás a fizikai tápegységek teljes számáról. Az interfészek között az eth-0-1 … eth-0-26 nevek maradtak meg; a VLAN- és loopback-sorok kiszűrésre kerültek. Nem üres cikkszám alapján hét optikai sor található. Az összetett hardverindexek változatlanul maradnak.

## Elvégzett vizsgálatok

- A generált YAML sikeresen visszaolvasható; az export verziója karakterláncként `7.0`.
- 111 egyedi, 32 hexadecimális karakterből álló, UUIDv4 formátumú azonosító; nincs ismétlődés.
- 75 egyedi item-/prototype-/discovery-kulcs; nincs ismétlődés a sablon definícióiban.
- Minden dependent item a sablonban létező master itemre hivatkozik; minden grafikon hivatkozása és értéktérkép-hivatkozás feloldható.
- A bekapcsolt SNMP-get és függő item OID-jai megtalálhatók a kiválasztott discovery soroknál a feltöltött walkokban. Kivételként a sysDescr és sysObjectID a beszélgetésben megadott adatokból került ellenőrzésre; ezekhez nem csatolt rendszerwalkot kaptam.
- 197 JavaScript preprocessing-ellenőrzés futott sikeresen Node.js alatt. Ezek közül 177 a ventilátor/DOM discovery sorainak tényleges numerikus értékét vizsgálta. További esetek ellenőrizték a CPU-t, a szóközöket, a negatív dBm értéket és az érvénytelen bemeneteket.
- Az üres, többcsatornás, `NaN`, `-inf`, mértékegységgel kevert értékek elutasításra kerültek; a numerikus DOM itemek hibakezelése ezeket a mintákat eldobja. A nyers aktuális DOM item megmarad.
- A memóriaarány a dumpban 34,72%; a két hőmérséklet 49 és 61 °C.
- A Counter64 forgalom a natív SNMP_WALK_VALUE → CHANGE_PER_SECOND → MULTIPLIER 8 láncot használja; nincs pontosságvesztést okozó JavaScript Number-konverzió a forgalmi számlálókon.
- Az exportmezőket és preprocessing típusokat a hivatalos 7.0 sablon és importvalidator forrásához igazítottam. Ez forrásalapú ellenőrzés, nem a PHP importvalidator futtatása.
- A kimeneti fájlok nem tartalmaznak community-t vagy a teljes vendor dump hitelesítési ágait. Nem kerültek át hostcímek vagy konkrét sorozatszámok.

## Amit ez a vizsgálat nem igazol

**Nem történt élő Zabbix-import, Zabbix API-hívás, élő SNMP-poll vagy időbeli trigger-visszajátszás.** A JavaScript futtatása Node.js-ben történt, nem a Zabbix preprocessing motorjában. A natív számlálópreprocessing és a triggerkifejezések szerveroldali kiértékelése nem futott ebben a környezetben.

A portállapot és hibaszámlálók standard ifTable OID-jai még nincsenek meg a kapott dumpokban, ezért az opcionális discovery alapértelmezésben kikapcsolt. A QSFP csatornánkénti DOM-feldolgozásához nincs minta. Az S5860/S5810 modellekhez ez az ellenőrzés nem használható kompatibilitási igazolásként.

Az első élő próba lépéseit és az elvárt ellenőrzéseket a README-hu.md tartalmazza. A tényleges import vagy polling során jelentkező hibákat a pontos Zabbix hibaüzenettel lehet javítani.

## Fájlazonosítás

`fs_s5850c_zabbix70.yaml` SHA-256:

```text
e1da3d6ede2a976e13dcf605d2cceedf59c0c81e86b51dd0f75d25a0ca15659c
```
