
# Probewoche [#matrix](https://matrix.org/)

1. Über Matrix
2. Aufgabe
3. Installation
4. Federation
5. Raum-Konfiguration
6. Nächste Schritte

## Über Matrix

Matrix ist ein Kommunikations-Tools zum *optional verschlüsselten* Nachrichtenaustausch in Räumen und 1:1-Kanälen.

### Organisation der Kommunikation

Räume können in sogenannten 'Spaces' gruppiert werden. Siehe auch die [Ankündigung der Spaces Beta](https://matrix.org/blog/2021/05/17/the-matrix-space-beta).
Im Hintergrund sind Spaces als Räume realisiert.
*(Hinweis: Dadurch können auch komplette Spaces über die Admin-Oberfläche in das 'Raum-Register' eingetragen werden. Dadurch können externe Nutzer ganzen Spaces beitreten.)*

### Federation

Matrix-Server unterschiedlicher Organisationen können durch 'Federation' verbunden werden.
Dadurch können auch Nutzer mit Accounts bei einem anderen Matrix-Server die Dienste (Räume, Spaces, Kommunikationen) anderer Server nutzen.
[Setting up federation](https://matrix-org.github.io/synapse/latest/federate.html)
Beim Einrichten eines eigenen Servers, **bei dem externer Zugriff erwünscht ist** ist der
[federation tester](https://federationtester.matrix.org/) potentiell hilfreich.

### Moderation

In Spaces und Räumen können über 'power level' Benutzern verschiedene Rechte eingeräumt werdne.
In einem Space/Raum wird dafür für die unterschiedlichen Aktionen der *'minimale power-level'* konfiguriert.
Benutzern können im Kontext eines Spaces/Raumes dann power-level zugewiesen werden.
Siehe auch [Moderation in Matrix](https://matrix.org/docs/guides/moderation).

Die Konfiguration der Moderation kann in Clients wie [Element](https://matrix.org/docs/projects/client/element), also
über die *Client-API* vorgenommen werden.

*Hinweis: Ich konnte über die Client-API keine Möglichkeit ermitteln einen kompletten Space zur Federation in das Raum-Register einzutragen.
Dafür muss man scheinbar die Admin-API bzw. eine UI wie [synapse-admin](https://github.com/Awesome-Technologies/synapse-admin) verwenden.*


## Aufgabe

Es sollte ein Matrix-Server aufgebaut werden, über den ein Teil der Öffentlichkeitsarbeit
abgewickelt werden kann.

Struktur siehe [Raum-Konfiguration](#raum-konfiguration) weiter unten.

## Installation

### Allgemein

Ein Matrix-Server besteht aus einem *homeserver*, welcher verschiedene Komponenten nutzt wie
- Datenbank
- HTTP-Proxy (für Verschlüsselung und Zertifikate)
- Optionale Dienste wie Bridges zu anderen Netzerken wie *Telegram*, *Signal*, etc.
- Optionale Bridges zu externen Diensten wie Amazon S3, Authentifikations-/Autorisierungs-Providern wie LDAP und REST, etc.

### Ansible Playbook

(Gefunden auf [Try-Matrix-Now](https://matrix.org/docs/projects/try-matrix-now/) unter 'Other'.)

Links:
- [Ansible Playbook](docs/configuring-playbook.md)
- [Docs](https://github.com/spantaleev/matrix-docker-ansible-deploy/tree/master/docs#table-of-contents)
- insbesondere [Configuring the Ansible playbook](docs/configuring-playbook.md)

Für den *relativ schnellen* Aufbau einer solchen Infrastruktur gibt es ein [Ansible Playbook 'matrix-docker-ansible-deploy'](https://github.com/spantaleev/matrix-docker-ansible-deploy).
In diesem Playbook kann eine Matrix-Installation konfiguriert und installiert werden.
Die Aktivierung und Konfiguration der verschiedenen Dienste kann dabei - wie für Ansible üblich -
über Konfigurationsparameter gesteuert werden.

Auch einige Infrastruktur-Aufgaben wie das Anlegen von Benutzern ist über dieses Playbook möglich.

Dabei wird eine **Docker-Infrastruktur** aufgebaut und konfiguriert.

**Hinweis**: Die Firewall-Konfiguration erfolgt in eigenen Chains. Einsicht erhält man über `sudo ufw show raw | less`.

### DNS-Konfiguration

Insbesondere zur Federation ist es notwendig, dass die DNS-Einträge korrekt sind. Siehe [Configuring your DNS server](docs/configuring-dns.md).

### Playbook-Konfiguration

Die Konfiguration ist in diesem Repostiory hinterlegt im Verzeichnis [inventory](inventory).
- [hosts](inventory/hosts)
- [vars.yml](inventory/host_vars/matrix.io99.de/vars.yml)
- **Achtung**: nicht eingecheckt [vars.secret.yml](inventory/host_vars/matrix.io99.de/vars.secret.yml)

### Scripts

**Warnung**:
- Manche Parameter dürfen nach dem initialen Setup nicht mehr angepasst werden. Beispielsweise das Datenbank-Passwort.
- Manche Fehlkonfigurationen führen zu einem Scheitern des Server-Starts. Beispielsweise 'fehlendes "#"' bei der Konfiguration der `matrix_synapse_auto_join_rooms`.

#### Initiales Setup

`ansible-playbook -i inventory/hosts setup.yml --tags=setup-all`

#### Start

`ansible-playbook -i inventory/hosts setup.yml --tags=start`

#### Konfigurations-Änderung

`ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start`

#### Benutzer anlegen

`ansible-playbook -i inventory/hosts setup.yml  --extra-vars='username=<your-username> password=<your-password> admin=<yes|no>' --tags=register-user`

## Federation

Da es die Aufgabe war, einen Server aufzusetzen, über den auch andere Personen an Kommunikationen teilhaben dürfen
wurde auch die *Federation* konfiguriert.

Somit können Nutzer, welche bereits einen Matrix-Account 'anderswo' haben die Spaces und Räume des Servers einfach durch 'beitreten' nutzen.

**Hinweis**: Für einen 'internen' Server wäre eine solche Freigabe vermutlich weniger angebracht.

## <a name="#raum-konfiguration"></a> Raum-Konfiguration

Links:
- [Moderation in Matrix](https://matrix.org/docs/guides/moderation)

Die Spaces und Räume wurden 'per Hand' über einen Client angelegt.

Die öffentlichen Spaces wurden anschließend über die Admin-UI ins Raumverzeichnis eingetragen.
Dadurch kann von extern ganzen Spaces begetreten werden.

Sturktur:

- Infopunkt (Space, öffentlich)
	- Anleitung (Room)
		- öffentlich
		- moderiert
	- Hilfe (Room)
		- öffentlich
		- unmoderiert
- Neuigkeiten & Infogruppe (Space, öffentlich)
	- Neuigkeiten (Room)
	- Infogruppe (Room)
- Regionale Vernetzung (Space, öffentlich)
	- Baden-Württemberg (Room)
		- öffentlich
		- unmoderiert
	- Bayern (Room)
		- öffentlich
		- unmoderiert
	- ...
- Regionale Vernetzung (Space, intern)
	- Baden-Württemberg (Room)
		- intern
		- unmoderiert
	- Bayern (Room)
		- intern
		- unmoderiert
	- ...
- Mitfahrbörse (Space)
	- Suche (Room)
		- öffentlich
		- unmoderiert
	- Biete (Room)
		- öffentlich
		- unmoderiert

## Nächste Schritte

### Installation / Integration

- Backup der Datenbank (siehe [Backing up PostreSQL](docs/maintenance-postgres.md#backing-up-postgresql))
- Eventuell [Borg Backup](docs/configuring-playbook-backup-borg.md)
- Eventuell 'externe Datenbank'
- Evaluation von *Versions-Upgrades* (siehe [Upgrading the Matrix services](docs/maintenance-upgrading-services.md))

### User-Management Automatisierung

- Eventuell [Matrix Corporal](https://github.com/devture/matrix-corporal)

### Spam-Protection

- Eventuell [Matrix Mjolnir](https://github.com/matrix-org/mjolnir)

### Help-Desk

- Eventuell [Honoroit Setup](/docs/configuring-playbook-bot-honoroit.md)
- Eventuell [Email2Matrix Setup](docs/configuring-playbook-email2matrix.md)

### Weitere '*Inspirationen*'

Das [README](README.md) bietet ettliche Inspirationen.

- [Dimension Setup](docs/configuring-playbook-dimension.md) [Dimension](https://dimension.t2bot.io/)
- [Jitsi Setup](docs/configuring-playbook-jitsi.md)
- [synapse-simple-antispam Setup](docs/configuring-playbook-synapse-simple-antispam.md)
- [Mjolnir Spam Protection Setup](docs/configuring-playbook-bot-mjolnir.md)
- [Email2Matrix Setup](docs/configuring-playbook-email2matrix.md)

### API-Zugriff

- Siehe [Client-Server API](https://matrix.org/docs/guides/client-server-api)
- Siehe [Matrix SDKs](https://matrix.org/sdks/)
- [Admin API](https://matrix-org.github.io/synapse/latest/usage/administration/admin_api/index.html)

### Eigene Module

- Siehe [Modules](https://matrix-org.github.io/synapse/latest/modules/index.html)
- Siehe [Workers](https://matrix-org.github.io/synapse/latest/workers.html)
