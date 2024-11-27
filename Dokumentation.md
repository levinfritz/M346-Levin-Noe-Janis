# Dokumentation M346-CMS

Dieses Repository wurde in Zusammenarbeit von Fabian Peter, Romeo Davatz und David Bürge erstellt. Diese Dokumentation beschreibt das gesamte Prokjekt von der Planung bis zum fertigen Script. Ebenfalls folgt eine Anleitung, wie das CMS anhand unseres Scriptes zu installieren ist.

[**1. Projektinformationen**](#anker)  
[**1.1 CMS**](#anker1)  
[**1.2 Aufgaben und Zuständigkeit**](#anker2)  
[**1.3 Aufgaben und Zuständigkeit**](#anker3)  
[**2. Installation und Konfiguration**](#anker4)  
[**2.1 Erklärung des Codes**](#anker8)  
[**3. Anleitung**](#anker5)  
[**4. Testfälle**](#anker6)  
[**5. Reflexion**](#anker7)

<a name="anker"></a>
## 1. Projektinformationen
In diesem Abschnitt werden grundlegende Informationen zum Projekt wie die gegebene Aufgabe, Wahl des CMS und die Aufgabenverteilung in der Gruppe aufgezählt.

<a name="anker1"></a>
### 1.1 Aufgabenstellung
Für das Prokjekt, musste ein CMS auf einer AWS instanz erstellt werden. Der installation der Instanzen und dem CMS sollte schlussendlich Automatisiert werden.. 

<a name="anker2"></a>
### 1.2 Wahl CMS  
Ein Content-Management-System (CMS) ist eine Softwareanwendung, die es Benutzern ermöglicht, Inhalte auf Websites zu erstellen, zu bearbeiten und zu verwalten, ohne umfangreiche Programmierkenntnisse zu benötigen. Es ist eine effektive Lösung für die Verwaltung von digitalen Inhalten, sei es Texte, Bilder, Videos oder andere Medien.

Das CMS bietet eine benutzerfreundliche Oberfläche, die es Benutzern ermöglicht, Inhalte direkt im Webbrowser zu erstellen und zu bearbeiten. Es ermöglicht die Organisation von Inhalten in einer hierarchischen Struktur, um eine einfache Navigation zu gewährleisten. Ein CMS erleichtert auch die Zusammenarbeit verschiedener Benutzer, indem es die Berechtigungen und Zugriffslevel verwaltet.

Als CMS haben wir uns für WordPress entschieden, da uns dies bereits bekannt war. Ausserdem ist es eines der bekanntesten CMS, daher findet man man ausreichent Informationen und Dokumentationen im Internet, was uns die Arbeit erleichtern konnte.

<a name="anker3"></a>
### 1.3 Aufgaben und Zuständigkeit
Für das Prokelt musste das CMS umgesetzt werden, sowie eine ausführliche Dokumentation gestaltet werden. Grundsätzlich war jeder in der gruppe bei allem beteiligt. Trotzdem konzentirerten sich  Fabian und David eher auf die Installation des CMS, wobei sich Romeo ausführlicher mit der Dokumentation befasste. 

<a name="anker4"></a>
## 2. Installation und Konfiguration
  Für die Umsetzung haben wir 3 verschiedene FIles verwendet. Das [setup-wordpress-aws.sh](https://github.com/davidbuerge1/M346-CMS/blob/main/setup-wordpress-aws.sh) diente dabei als die "grundlegende" Datei, inder die Sicherheitsgruppen, Rules sowie die Schlüsselpaare definiert wurden. Das [DB-server-setup.sh](https://github.com/davidbuerge1/M346-CMS/blob/main/server-setup/DB-server-setup.sh), wurde dabei als Konfigurationsdatei für die Datenbank verwendet. Das [CMS-server-setup.sh](https://github.com/davidbuerge1/M346-CMS/blob/main/server-setup/CMS-server-setup.sh) wurde als Konfigurationsdatei für die Instanz verwendet, auf welcher das CMS läuft. Zum Code finden Sie weitere Informationen unter [Code erklärt](#anker8).

  Am Anfang waren wir uns unschlüssig, wie wir das Projekt umsetzen könnten. Nach reichlichem informieren, haben wir uns dazu entschieden Docker zu verwenden, da es sehr Effizient ist und auch in Professionellen Umgebungen öfters verwendet wird. Ausserdem haben wir zwei verschiedene EC2 Instanzen verwendet, wobei die Instanz der Datenbank durch eine Sicherheitsgruppe geschützt ist so, dass nicht aus dem Internet direkt darauf zugegriffen werden kann. Die CMS Instanz, kommuniziert über die Interne IP-Adresse mit der Datenbank und mit der externen Adresse mit dem Endbenutzer.

  

<a name="anker8"></a>
### 2.1 Erklärung des Codes
### [setup-wordpress-aws.sh](https://github.com/davidbuerge1/M346-CMS/blob/main/setup-wordpress-aws.sh)

Hier wird ein zufälliges Passwort generiert, welches von den Instanzen übernommen wird, um die Sicherheit zu steigern.
```
password=$(openssl rand -base64 36 | tr -dc 'a-zA-Z0-9' | head -c 54)
```
Dieser Code erstellt das Init-file für die Datenbank. Dabei werden die benötigten Packages installiert sowie befehle aufgeschrieben, die später auf der Instanz ausgeführt werden sollen. Beispielsweise das ausühren des Datenbank Setups. Ausserdem wird die Setupdatei der Datenbank direkt aus unserem Repository heruntergeladen um Fehler zu verhindern und mithilfe des zuvor erstellten Passworts ausgeführt. Daher ist das Repository Public und für alle zugänglich, da ansonsten der download von den Instanzen nicht funktionieren würde
```
cat <<END > init.yaml
#cloud-config
package_update: true
packages:
  - curl
  - mariadb-server
  - git
runcmd:
  - git clone "https://github.com/davidbuerge1/M346-CMS.git" setup
  - cd setup/server-setup
  - chmod +x DB-server-setup.sh
  - sudo bash DB-server-setup.sh $password
END
```
Hier wird ein Keypair für den Zugriff auf die Instanz erstellt.
```
aws ec2 create-key-pair --key-name WordPress-AWS-Key --key-type rsa --query 'KeyMaterial' --output text > ./WordPress-AWS-Key.pem
```
Mit diesem Code werden die beiden Sicherheitsgruppen erstellt. Eine für die Interne Kommunikation zwischen den Instanzen und die andere für die Kommunikation mit externen Netzwerken.
```
aws ec2 create-security-group --group-name WordPress-net-Intern --description "Internes-Netzwerk-fuer-WordPressDB"
aws ec2 create-security-group --group-name WordPress-net-Extern --description "Externes-Netzwerk-fuer-WordPressCMS"
```
Hier wird die Instance der Datenbank gestartet.
```
aws ec2 run-instances --image-id ami-08c40ec9ead489470 --count 1 --instance-type t2.micro --key-name WordPress-AWS-Key --security-groups WordPress-net-Intern --iam-instance-profile Name=LabInstanceProfile --user-data file://init.yaml --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WordPressDB}]'
```
Mit folgendem Code wird die Interne sowie die Externe IP-Adresse des DB-Servers ermittelt, um später die Kommunikation zu gewährleisten.
```
WPDBInstanceId=$(aws ec2 describe-instances --query 'Reservations[0].Instances[0].InstanceId' --output text --filters "Name=tag:Name,Values=WordPressDB")
WPDBPrivateIpAddressip=$(aws ec2 describe-instances --query 'Reservations[0].Instances[0].PrivateIpAddress' --output text --filters "Name=tag:Name,Values=WordPressDB")
```
Hier wird die Id der Security-group ausgelesen, um später Regeln den entsprechenden Regeln den Gruppen zuzuweisen.
```
SecurityGroupId=$(aws ec2 describe-security-groups --group-names 'WordPress-net-Extern' --query 'SecurityGroups[0].GroupId' --output text)
```
Hier werden die Regeln für die Security-Groups definiert.
```
aws ec2 authorize-security-group-ingress --group-name WordPress-net-Intern --protocol tcp --port 3306 --source-group $SecurityGroupId
aws ec2 authorize-security-group-ingress --group-name WordPress-net-Intern --protocol tcp --port 22 --source-group $SecurityGroupId
aws ec2 authorize-security-group-ingress --group-name WordPress-net-Extern --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name WordPress-net-Extern --protocol tcp --port 443 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name WordPress-net-Extern --protocol tcp --port 22 --cidr 0.0.0.0/0
```
Mithilfe dieses Codes wird das Init-file für die CMS-instanz erstellt. Wie auch bei der Datenbank, wird die Setupdatei direkt aus dem Repository heruntergeladen und ausgeführt
```
cat <<END > init.yaml
#cloud-config
package_update: true
packages:
  - git
  - ca-certificates
  - curl
  - gnupg
  - software-properties-common
  - apt-transport-https
  - cron
  - snapd
runcmd:
  - git clone "https://github.com/davidbuerge1/M346-CMS.git" WordPressCMS
  - cd WordPressCMS/server-setup
  - chmod +x CMS-server-setup.sh
  - sudo bash CMS-server-setup.sh $WPDBPrivateIpAddressip $password WordPressDB
END
```
Hier wird die zweite Instanz  mithilfe des Init-files erstellt.
``` 
aws ec2 run-instances --image-id ami-08c40ec9ead489470 --count 1 --instance-type t2.micro --key-name WordPress-AWS-Key --security-groups WordPress-net-Extern --iam-instance-profile Name=LabInstanceProfile --user-data file://init.yaml --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WordPressCMS}]'
```

### [DB-server-setup.sh](https://github.com/davidbuerge1/M346-CMS/blob/main/server-setup/DB-server-setup.sh)
Dieser Befehl führt eine Änderung in der Datei "50-server.cnf" durch, die sich im Verzeichnis "/etc/mysql/mariadb.conf.d/" befindet. Dabei wird die Option "bind-address" so modifiziert, dass sie auf "127.0.0.1" festgelegt wird.
```
sudo sed -i 's/bind-address\s*=.*/bind-address = 127.0.0.1/' /etc/mysql/mariadb.conf.d/50-server.cnf
```
Der erste Befehl weist dem Benutzer 'root' alle Privilegien für alle Datenbanken zu, ermöglicht den Zugriff von jedem beliebigen Host aus und setzt das Passwort, das als Parameter '$1' übergeben wird. Der zweite Befehl aktualisiert die Berechtigungen, um die Änderungen wirksam zu machen. Der dritte Befehl erstellt eine neue Datenbank mit dem Namen "WordPressDB" unter Verwendung des angegebenen Benutzernamen und Passworts. Die Befehle werden alle mit Root-Berechtigungen ausgeführt, die durch sudo verliehen werden, und erfordern eine Passwortabfrage, um sich als Benutzer 'root' anzumelden.
```
echo "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '$1' WITH GRANT OPTION;" | sudo mysql -u root -p"$1"
echo "FLUSH PRIVILEGES;" | sudo mysql -u root -p"$1"
echo "create database WordPressDB;" | sudo mysql -u root -p"$1"
```
Mithilfe dieses Codes werden die beiden Ports freigegeben, um die Kommunikation zu ermöglichen.
```
ufw allow 3306
ufw allow 22
```
Anpassung der Mariadb conf
```
sed -i '/^bind-address/ s/^/#/' /etc/mysql/mariadb.conf.d/50-server.cnf
```

### [CMS-server-setup.sh](https://github.com/davidbuerge1/M346-CMS/blob/main/server-setup/CMS-server-setup.sh)
Installation von Docker
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" -y
apt update -y
apt-get install docker-ce docker-ce-cli containerd.io -y
curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
Anpassung der des docker-compose.yml
```
cd /WordPressCMS/server-setup/docker
sed -i "s/<DB-Host>/$1/g" docker-compose.yml
sed -i "s/<DB-User>/root/g" docker-compose.yml
sed -i "s/<DB-Password>/$2/g" docker-compose.yml
sed -i "s/<DB-Name>/$3/g" docker-compose.yml
```
    

<a name="anker5"></a>
## 3. Anleitung zur Installation 
### 1. Schritt 
Hier sind die Voraussetungen, bevor die Instanzen installiert werden können.
 
- [X] Auf der Ubuntumaschine muss das AWS bereits konfiguriert sein.
- [X] Vollständige Konfiguration vom AWS Client auf einer Ubuntu-maschine.
- [X] Es darf kein Key, auf dem lokalen PC oder auf dem AWS, mit dem Namen "WordPress-AWS-Key" geben.
- [X] Es darf keinen Ordner mit dem Namen WordPressCMS existieren.
- [X] Auf dem AWS darf es keine Instanzen mit den Namen "WordPressDB" und "WordPressCMS" geben.
- [X] Auf dem AWS darf es keine Sicherheitsgruppe mit dem Namen "WordPress-net-Intern" oder "WordPress-net-Extern" geben.
  
### 2. Schritt
Das Script muss zuerst mithilfe dieses Befehls ausgeführt heruntergeladen werden.
```
git clone "https://github.com/davidbuerge1/M346-CMS.git" WordPressCMS
```

### 3. Schritt  
Nun muss in das zuvor heruntergeladene verzeichnis gewechselt werden. Die Berechtigungen der Datei müssen so angepasst werden, dass die Datei executable ist.

### 4. Schritt  
Skrip muss mithilfe folgendes Befehls ausgeführt werden.

### 5. Schritt
Das Programm bleibt bei der Erstellung der beiden Instanzen stehen und bietet eine Übersicht. Um das Programm weiter auszuführen, muss in der Konsole mit q weitergefahren werden. 

![image](https://github.com/davidbuerge1/M346-CMS/blob/main/server-setup/Screenshot%202023-12-22%20221931.png)

Nachdem das Programm fertig durchgelaufen ist, dauert es einige Minuten, bis die Webseite zugänglich ist. Danach kann die Webseite mithilfe der öffentlichen IP-Adresse geöffnet werden.

<a name="anker6"></a>
## 4. Testfälle  
**Testfall 1** 
Wir hatten lange die Prtoblematik, dass die CMS-Instanz keine Verbindung mit der DB-Instanz herstellen konnte. Dabei erschien folgende Fehlermeldung.

![image2](https://github.com/davidbuerge1/M346-CMS/blob/main/server-setup/207bb161-4192-4b38-9554-fa0202a65119.jpg)

Die IP-Adressen werden mithilfe der unten stehenden Befehle ausgelesen. Der Fehler entstand, da auf dem AWS, auf dem die Instanzen installiert werden sollten bereits Instanzen mit dem gleichen Namen existierten. Daher konnte die IP-Adresse des Servers nicht ausgelesen werden. Im jetztigen Script haben wir dies korrigiert. Nun wurden keine weiteren fehler gefunden.
  
**Testfall 2**  
Ausserdem hatten wir lange probleme mit den Speicherorten der verschiedenen Files. Oft wurde nicht richtig auf das File verwiesen. So konnten die Scripts auch nicht richtig ausgeführt werden.

Bei diesem Beispiel fehlt vor WordPressCMS das "/". Daher wurde das docker-compose auch nicht richtig ausgeführt
```
cd WordPressCMS/server-setup/docker
```
<a name="anker7"></a>
## 5. Reflexion 

**David Bürge**  
Ich denke ich konnte in diesem Projekt sehr viel lernen. Besonders im Bezug auf die Fehlerbehebung. Das im Unterricht gelernte konnte nun auch praktisch angewendet werden. Anfangs hatten wir noch einige schwierigkeiten, da wir nicht richtig wussten wo und wie wir am Projekt beginnen sollten. Daher gibt es auch das Script [InstallInstances.sh](https://github.com/davidbuerge1/M346-CMS/blob/main/InstallInstances.sh). Nach einigem Informieren und ein wenig Starthilfe einer anderen Gruppe, konnte das Projekt jedoch erfolgreich abgeschlossen werden. Ich denke für ein nächstes Projekt, wäre vor allem die aufteilung der verschiedenen Arbeiten ein wichtiger Punkt. Insgesamt denke ich, dass das Projekt eine sehr interessante und lehrreiche Erfahrung war und denke, dass wir das Projekt erfolgreich abschliessen konnten.

**Fabian Peter**  
Für mich war das Projekt sehr interessant, ich habe viel neues daraus gelernt. Auch konnte ich schon gelerntes sehr gut anwenden. Da ich im Basislehrjahr als abschluss Projekt Nextcloud auf Debian installieren musste, waren mir vieles schon bekannt. Beispielsweise wusste ich genau wofür eine .conf-Datei verwendet wird und in welchem Verzeichnis sie liegt. Auch sehr Hilfreich war das Vorwissen zu allen Linux-Commands. Ich finde, dass uns in der Schule sehr gut die Verbindung zum AWS erklärt wurde, was somit keine grosse Herausforderung mehr war. Jedoch war das Skript ansich dafür viel schwerer. Es war nicht ganz einfach ein Skript von 0 auf zu schreiben. Das nächste mal würde ich die Kommunikation untereinander besser machen, damit nicht immer nur jemand daran arbeitet. Im grossen und ganze war aber sehr viel spannendes im Projekt dabei und ich denke wir haben es gut abgeschlossen. 


**Romeo Davatz**  
Für mich war das Projekt eine gute Erfahrung. Ich konnte das, was ich im Unterricht gelernt habe praktisch anwenden. Wir konnten die Probleme gut angehen und sind wie ich denke zu einem guten ergebniss gekommen. Für ein späteres Projekt, könnten wir die Aufgabenverteilung in der Gruppe verbessern. 