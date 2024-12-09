# Dokumentation M346-Ticketsystem

Dieses Repository wurde in Zusammenarbeit von Levin Fritz, Noé Messmer und Janis Mora erstellt. Diese Dokumentation beschreibt das gesamte Projekt von der Planung bis zum fertigen Skript. Ebenfalls folgt eine Anleitung, wie das Ticketsystem anhand unseres Skriptes zu installieren ist.

[**1. Projektinformationen**](#anker)  
[**1.1 Aufgabenstellung**](#anker1)  
[**1.2 Wahl Ticketsystem**](#anker2)  
[**1.3 Aufgaben und Zuständigkeit**](#anker3)  
[**2. Installation und Konfiguration**](#anker4)  
[**2.1 Erklärung des Codes**](#anker8)  
[**2.2 Begründung für Terraform statt Cloud-Init**](#anker9)  
[**3. Anleitung**](#anker5)  
[**4. Testfälle**](#anker6)  
[**5. Reflexion**](#anker7)

<a name="anker"></a>
## 1. Projektinformationen

In diesem Abschnitt werden grundlegende Informationen zum Projekt wie die gegebene Aufgabe, Wahl des Ticketsystems und die Aufgabenverteilung in der Gruppe aufgezählt.

<a name="anker1"></a>
### 1.1 Aufgabenstellung

Für das Projekt musste ein Ticketsystem auf einer AWS-Instanz erstellt werden. Die Installation der Instanzen und des CMS sollte schlussendlich automatisiert werden.

<a name="anker2"></a>
### 1.2 Wahl Ticketsystem

Wir haben uns für osTicket entschieden, da es eine stabile Open-Source-Lösung ist, die alle wesentlichen Funktionen eines Ticketsystems abdeckt. Es bietet eine benutzerfreundliche Oberfläche, flexible Konfigurationsmöglichkeiten und Unterstützung für mehrere Kommunikationskanäle.

<a name="anker3"></a>
### 1.3 Aufgaben und Zuständigkeit

Für das Projekt musste das Ticketsystem umgesetzt sowie eine ausführliche Dokumentation gestaltet werden. Grundsätzlich war jedes Gruppenmitglied bei allen Aufgaben beteiligt. Levin und Noé konzentrierten sich jedoch stärker auf die Einrichtung der AWS-Umgebung, während Janis die Dokumentation ausführlich gestaltete.

<a name="anker4"></a>
## 2. Installation und Konfiguration

Für die Umsetzung haben wir verschiedene Skripte verwendet:

- [`main.tf`](https://github.com/levinfritz/M346-Levin-Noe-Janis/blob/main/Konfiguration/main.tf): Definiert die gesamte Infrastruktur auf AWS, einschließlich EC2-Instanzen, Sicherheitsgruppen und Schlüsselpaaren.
- [`web-init.sh`](https://github.com/levinfritz/M346-Levin-Noe-Janis/blob/main/Konfiguration/web-init.sh): Installiert und konfiguriert osTicket auf einer Apache-Instanz.
- [`db-init.sh`](https://github.com/levinfritz/M346-Levin-Noe-Janis/blob/main/Konfiguration/db-init.sh): Installiert und konfiguriert eine MariaDB-Instanz für osTicket.
- [`deploy.sh`](https://github.com/levinfritz/M346-Levin-Noe-Janis/blob/main/Konfiguration/deploy.sh): Automatisiert die Installation von Terraform und die Ausführung der Terraform-Skripte.


<a name="anker8"></a>
### 2.1 Erklärung des Codes

#### Terraform (`main.tf`)

- Erstellung eines neuen SSH-Schlüsselpaares mit `tls_private_key`.
- Definition der Sicherheitsgruppen für Web- und Datenbankzugriff.
- Einrichtung von zwei EC2-Instanzen:
  - Webserver mit osTicket
  - Datenbankserver mit MariaDB

#### Webserver-Initialisierung (`web-init.sh`)

- Installiert Apache, PHP und osTicket.
- Konfiguriert Berechtigungen für die Webserver-Dateien.
- Lädt osTicket von GitHub herunter und entpackt es.

#### Datenbank-Initialisierung (`db-init.sh`)

- Installiert MariaDB und erstellt die notwendige Datenbank und Benutzer für osTicket.
- Konfiguriert MariaDB für den Zugriff durch den Webserver.

#### Deploy-Skript (`deploy.sh`)

- Installiert Terraform, initialisiert das Terraform-Projekt und führt die Infrastruktur-Bereitstellung aus.


<a name="anker9"></a>
### 2.2 Begründung für Terraform statt Cloud-Init

Wir haben uns bewusst dafür entschieden, Terraform-Skripte anstelle von Cloud-Init-Skripten zu verwenden. Der Grund dafür ist, dass Terraform eine leistungsstarke und deklarative Methode zur Verwaltung der gesamten Infrastruktur bietet, die es uns ermöglicht, die Ressourcen wie EC2-Instanzen, Sicherheitsgruppen und Schlüsselpaare konsistent und wiederholbar bereitzustellen. Hier einige der Hauptgründe:

1. **Wiederholbarkeit und Konsistenz**: Terraform ermöglicht es uns, Infrastruktur als Code zu definieren, sodass die Bereitstellung in verschiedenen Umgebungen immer auf die gleiche Weise erfolgt. Dies stellt sicher, dass keine Abweichungen zwischen den Deployments auftreten.
  
2. **Zustandsmanagement**: Terraform speichert den Zustand der Infrastruktur, sodass wir leicht feststellen können, ob und wie die Infrastruktur geändert wurde. Dies erleichtert das Management und hilft uns, Änderungen nachzuvollziehen und zu testen.

3. **Komplexität der Infrastruktur**: Das Projekt erfordert die Erstellung und Verwaltung mehrerer Ressourcen wie EC2-Instanzen und Sicherheitsgruppen. Terraform ist besser geeignet, um diese Infrastruktur effizient zu verwalten, da Cloud-Init-Skripte hauptsächlich für die Konfiguration der Instanzen gedacht sind und nicht die gesamte Infrastrukturabstraktion übernehmen.

4. **Erweiterbarkeit**: Mit Terraform können wir die Infrastruktur in einer strukturierten Weise aufbauen und zukünftige Änderungen oder Erweiterungen problemlos umsetzen. Beispielsweise könnten wir zusätzliche EC2-Instanzen, S3-Buckets oder RDS-Datenbanken hinzufügen, ohne die gesamten Infrastrukturskripte neu zu schreiben.

5. **Integration mit anderen Tools**: Terraform lässt sich leicht in CI/CD-Pipelines integrieren, was uns erlaubt, die Infrastruktur automatisch bei jeder Änderung zu aktualisieren.

Cloud-Init ist eine großartige Lösung für die erste Konfiguration von Instanzen, jedoch eignet sich Terraform besser für das Management und die Automatisierung der gesamten Infrastruktur. Daher haben wir uns entschieden, Terraform als primäres Tool zu verwenden, um die Infrastruktur bereitzustellen.

<a name="anker5"></a>
## 3. Anleitung 

### 1. Voraussetzungen

- AWS CLI ist installiert und konfiguriert.
- Eine Ubuntu Maschine ist verfügbar und mit den notwendigen Berechtigungen ausgestattet.
- `git` ist installiert

### 2. Installation

1. Klone das Repository:
   ```bash
   git clone https://github.com/levinfritz/M346-Levin-Noe-Janis.git
   cd M346-Levin-Noe-Janis/Konfiguration
2. Mache das  deploy.sh Skript ausführbar und führe es aus. 
    ```bash
   chmod u+x deploy.sh
   ./deploy.sh

3. Greife auf den Webserver zu: Öffne die IP-Adresse des Webservers in deinem Browser. 

<a name="anker6"></a>
## 4. Testfälle  
**Testfall 1** <br>
Titel: Überprüfe, ob der Webserver erreichbar ist.<br>
Datum: 09.12.2024<br>
Testperson: Levin Fritz<br>

![image2](https://github.com/davidbuerge1/M346-CMS/blob/main/server-setup/207bb161-4192-4b38-9554-fa0202a65119.jpg)

Die IP-Adressen werden mithilfe der unten stehenden Befehle ausgelesen. Der Fehler entstand, da auf dem AWS, auf dem die Instanzen installiert werden sollten bereits Instanzen mit dem gleichen Namen existierten. Daher konnte die IP-Adresse des Servers nicht ausgelesen werden. Im jetztigen Script haben wir dies korrigiert. Nun wurden keine weiteren fehler gefunden.
  
**Testfall 2**  
Titel: Teste die Verbindung zur Datenbank.<br>
Datum: 09.12.2024<br>
Testperson: Levin Fritz<br>

Bei diesem Beispiel fehlt vor WordPressCMS das "/". Daher wurde das docker-compose auch nicht richtig ausgeführt
```
cd WordPressCMS/server-setup/docker
```
<a name="anker7"></a>
## 5. Reflexion 

**Levin Fritz**  
Ich konnte viel über die Automatisierung von Cloud-Infrastrukturen lernen. Besonders das Debuggen war lehrreich. Insgesamt hat das Projekt mein Verständnis für AWS und Automatisierung deutlich verbessert.

**Noé Messmer**  
Das Projekt war sehr spannend, und ich konnte mein Wissen über die Cloud und Linux-Administration erweitern. Für das nächste Projekt würde ich die Kommunikation und Aufgabenteilung in der Gruppe verbessern.

**Janis Mora**  
Ich fand das Projekt eine gute praktische Übung. Die Verbindung von Theorie und Praxis war sehr lehrreich. Für zukünftige Projekte sollten wir die Planungsphase intensiver gestalten.
