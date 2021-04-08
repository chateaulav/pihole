# Graylog3 supported
``Content Pack for piHole with Graylog``  
  
Built and based off of https://jalogisch.de/2017/der-eigene-dns-resolver-zuhause/, your own dns resolver (at home) by Jan Doberstein. Includes setting GeoIP, so ensure you download the current City db from Maxmind, and install the current Threat intelligence content packs. A seperate input is established to collect only pihole syslog traffic.

## Syslog requirements
**rsyslog** *Not recommended as you will get other unrelated logs*
```
#vi /etc/rsyslog.d/10-pihole.conf
`*.* @server.ip:1515;RSYSLOG_SyslogProtocol23Format

module(load="imfile" PollingInterval="10")

input(type="imfile"
File="/var/log/pihole.log"
StateFile="/var/run/pihole.log.state"
Tag="pihole"
Severity="info"
Facility="local7")`
```

**syslog-ng** *Best option, simple and only sends pihole data*
```
#apt install syslog-ng -y
#vi /etc/syslog-ng/conf.d/10-pihole.conf
source s_pihole_log { file("/var/log/pihole.log"); };
destination d_graylog {udp("server.ip" port(1515)); };
log { source(s_pihole_log); destination(d_graylog); };
```

### Content Pack includes:

* **GROK patterns**:  
``Defines all new fields to be set when matched in pipeline``
  * DNSMASQ
  * PIHOLE
  
* **Input**:  
``Sets a seperate input for just pihole DNS Logs``
  * piHole Syslog (``Listen 0.0.0.0:1515/UDP``)
  
* **Extractor**:
  * application_name (Looking for ``pihole``)
  
* **Pipeline**:  
``Creates Multiple fields to enable stronger queries and analytics``
  * -1 Rules:
    * ``dnsmasq pihole list``
    * ``dnsmasq split``
  * 0 Rules:
    * ``PiHole GeoIP Set``
  * 1 Rules:
    * ``threatintel (dnsmasq)``
    * ``dnsmasq clean message``
  * 2 Rules:
    * ``threatintel (2) inflate``
  
* **Lookup Table\Cache**:
  * geolite2-city (``/etc/graylog/server/GeoLite2-City.mmdb``)
  
* **Dashboard called ``DNS Intel``**:
  * DNS Location Requested IP (from answers)
  * DNS Activities (24h)
  * Thread Names (24h)
  * Blocked Domains (24h)
  * Blackholed Requests (24h)
  * Threat Indicated (24h)
  * DNS Clients (24h)
  * DNS Querys (24h)
  * Owning Companies (24h)
