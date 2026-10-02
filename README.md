# Zabbix templates

This repository currently provides standalone SNMP templates for FS switches targeting Zabbix 7.0. Templates use numeric OIDs and require no external scripts or linked templates.

## Compatibility and validation

| Model | Firmware | Status |
|---|---|---|
| FS S5850C-24S2C | 3.3.1.2.2 | Initial template; checked against reference SNMP walks. Live Zabbix import and polling pending. |
| FS S5860-20SQ, two-member stack | FSOS 12.6(2)B0101, Release(10131123) | Planned; separate OID mapping and stack validation required. |
| FS S5810-28TS | FSOS 11.4(1)B74T4, Release(12201912) | Planned; separate OID mapping required. |

Do not assume that templates work across FS models or firmware families. The current template targets sysObjectID `1.3.6.1.4.1.52642.1.99.21252`.

## S5850C template

Import [fs_s5850c_zabbix70.yaml](templates/fs/s5850c/fs_s5850c_zabbix70.yaml) through **Data collection → Templates → Import**, then link **FS S5850C by SNMP** to a host with an SNMPv2c interface. Configure the community on the host interface; no community value is stored in the template.

Supported measurements:

- System inventory, uptime and SNMP availability.
- CPU and reported memory utilization.
- Physical interface discovery, Counter64 traffic rates, reported speed and traffic graphs.
- Temperature sensors with device-provided thresholds.
- Fan state and percentage speed; power supply presence, state and alerts.
- Optical diagnostics: temperature, voltage, laser bias and TX/RX optical power, thresholds and raw readings.

The reference walks contain 26 physical interfaces, two fans, two temperature sensors, one exposed PSU row and seven optical modules. These counts describe the reference data, not fixed template limits.

The optional IF-MIB status/error discovery is disabled until its OIDs are verified. Link-down and optical alarms require explicit control macro settings. Multi-channel QSFP readings are retained as raw text; per-lane numeric monitoring is not yet implemented.

See the [Hungarian setup guide](docs/s5850c/README-hu.md), [OID mapping](docs/s5850c/oid-mapping.md) and [validation report](docs/s5850c/ellenorzesi-jegyzokonyv.md) for details and known limitations.

## Reporting results

When reporting an import or polling issue, include the Zabbix version, switch model and firmware, sysObjectID, affected item/OID, and the exact error message. For SNMP samples, provide only the relevant monitoring branches and redact identifying information.

Full enterprise walks can include authentication and configuration data. Never commit communities, credentials, private addresses, device serial numbers or unsanitized SNMP dumps.

## References

- [SWITCH MIB used to interpret the vendor OIDs](https://github.com/librenms/librenms/blob/master/mibs/fs/centec/SWITCH)
- [Official Zabbix 7.0 generic network template](https://github.com/zabbix/zabbix/blob/release/7.0/templates/net/generic_snmp/template_net_generic_snmp.yaml)
- [Zabbix 7.0 preprocessing documentation](https://www.zabbix.com/documentation/7.0/en/manual/config/items/preprocessing)

This is a custom template project. It is not an official FS or Zabbix integration.
