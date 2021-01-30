# iptables

## Aktionen

- `ACCEPT`: Paket annehmen
- `DROP`: Paket verwerfen
- `QUEUE`: Leitet an Queue Handler weiter
- `RETURN`: Paket wird in der vorangegangenen Reihe zurück geschickt
    Standardmäßig `ACCEPT`

## Ketten
Es gibt drei Standard Ketten:

- `INPUT`: Behandelt Pakete für das System
- `FORWARD`: Behandelt Pakete die weitergeleitet werden sollen
- `OUTPUT`: Behandelt Pakete die vom eigenen System erzeugt werden

Neben der Filtertabelle gibt es noch die `NAT`-Tabelle und die `MANGLE`-Tabelle
(Paketmanipulation).

## Befehlsübersicht
Optionen für Ketten:

| Option                        | Erklärung   |
| ----------------------------- | ----------- |
| -N `name`                     | Kette mit Namen erstellen     |
| -X `name`                     | Leere Kette mit Namen löschen |
| -L `name`                     | Alle Regeln einer Kette auflisten |
| -F `name`                     | Alle Regeln in einer Kette löschen |
| -P `name` `action`            | Policy für Kette festlegen (z.b. `ACCEPT`) |
| -A `name` `regel`             | Regel an Kette anhängen |
| -D `name` `regel`             | Regel in Kette löschen |
| -I `name` `position` `regel`  | Regel an bestimmer Stelle in Kette einfügen |
| -D `name` `position`          | Regel an bestimmer Stelle in Kette löschen |

Optionen für Regeln:

| Option | Erklärung |
| ------ | --------- |
| -i `interface` | Eingehendes Interface |
| -o `interface` | Ausgehendes Interface |
| -j `action`    | Aktion für Regel |
| -p `tcp/udp`   | Protokoll, das erlaubt werden soll |
| --dport `port` | "Destination Port" -> Zielport des Pakets |
| --sport `port` | "Source Port" -> Quellport des Pakets |
| -m state --state `STATUS` | Status des Pakets mit prüfen (bestehende Verbindungen: `ESTABLISHED`, auf bestehende Verbindungen beziehen: `RELATED`)|

## iptables-save
Das erstellte Regelwerk ist flüchtig. Deswegen muss die aktuelle Konfiguration
in einer `.rule`s Datei gespeichert werden:

```bash
sudo iptables-save > /etc/iptables/iptables.rules
```

Nach dem Neustart des Systems:

```bash
sudo iptables-restore < /etc/iptables/iptables.rules
```

## Einrichtung
Erste Schritte mit `iptables`:

Bestehende Einträge löschen:
```bash
sudo iptables -F
```

Default Policies für Ketten festlegen:
```bash
# Alles soll blockiert werden
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP
```

Traffic auf `localhost`-Interface erlauben:
```bash
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
```

Ausgehende HTTP(S)-Verbindungen zulassen:
```bash
sudo iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT
```

Bereits bestehende Verbindungen erlauben:
```bash
sudo iptables -A INPUT -i <interface> -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -o <interface> -m state --state RELATED,ESTABLISHED -j ACCEPT
```
