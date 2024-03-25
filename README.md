# ASP.NET: Hosting auf Linux

Dies ist das Repository für den **LinkedIn Learning** Kurs `ASP.NET: Hosting auf Linux`. Den gesamten Kurs finden Sie auf [LinkedIn Learning][lil-course-url].
[![COURSENAME][lil-thumbnail-url] 

ASP.NET-Anwendungen auf Linux zu hosten gilt als elegante Lösung und bietet viele Vorteile, die in erster Linie auf die Flexibilität, Anpassbarkeit, Skalierbarkeit und Sicherheit des offenen und kostenlosen Betriebssystems zurückzuführen sind. Zudem unterstützt Linux neben ASP.NET auch viele andere Webentwicklungstechnologien, wie Perl, PHP, MySQL, Python oder Ruby.<br><br>

Mirko Matytschak erläutert in seinem LinkedIn Learning-Kurs alle Schritte, die beim Aufbau eines solchen Hosting-Systems eine Rolle spielen: Er beginnt bei den Vorüberlegungen zur Auswahl der Komponenten, zeigt die komplette Installation eines Linux-Servers und die Einrichtung der Applikation inklusive Backup- und Security-Strategien.

## Inhaltsverzeichnis
Ein Inhaltsverzeichnis finden Sie, wenn Sie oben neben dem Dateinamen README.md auf das Menü-Symbol klicken:

![grafik](https://github.com/LinkedInLearning/asp-net-hosting-linux-2702073/assets/43751391/881ca078-c37b-4743-94de-d59499d01336)

## sudo oder root?
Enablen des root-Accounts:
```
sudo passwd root
```
Disablen des root-Accounts:
```
sudo passwd -l root
```
Auch wenn ich meine eigenen Systeme mit root betreibe, versuche ich, im Folgenden die Anweisungen mit sudo anzugeben. Das vorangestellte sudo schadet nicht, wenn Sie als root eingeloggt sind.

## Zeitzone
Zu diesem Text gibt es kein Video.

Mit

```plaintext
timedatectl
```

kann man sich anzeigen lassen, wie die Zeitzone eingestellt ist. Wenn sie nicht auf CET eingestellt ist, dann kann man das mit

```plaintext
dpkg-reconfigure tzdata
```

einstellen. Die LocalTime ist wichtig für Umbraco, damit im Backoffice die richtige Zeit steht.

## SSH einrichten

Erst einmal überprüfen, ob SSH installiert ist:
```
systemctl status ssh
```
Die Wahrscheinlichkeit, dass der SSH-Server bereits läuft, ist sehr hoch. Sonst:

```
sudo apt install openssh-server
```
Nach dem Install läuft der Server gleich los, was man wiederum mit `systemctl status ssh` überprüfen kann.

### Client-Software

[Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/)

[WinScp](https://winscp.net/eng/docs/lang:de)

[Notepad++](https://notepad-plus-plus.org/)

### Schlüssel erzeugen

Im Video zeige ich noch die folgenden Parameter:
```
ssh-keygen -f ./ihrname -t ecdsa -b 521
```
Und die Angabe der Schlüssellänge 521 Bytes war falsch. Es muss natürlich Bits heißen. Wir benutzen in der Firma
```
ssh-keygen -f ./ihrname -t ed25519
```
Das erzeugt eine Elliptic Curve nach neuestem Stand der Technik.

```
cd ~
mkdir .ssh
echo "ecdsa-sha2-nistp521 GAAAAAAANZ_VIELE_ZEICHEN== mirko matytschak@W10" > .ssh/authorized_keys
```
Dann den privaten Schlüssel mittels PuttyGen in das .ppk-Format konvertieren.
Der private Schlüssel kann dann in Putty und WinScp unter SSH/Authorization eingegeben werden. Es erfolgt dann ein Login via Key.

### Passwort-Login deaktivieren
Datei /etc/ssh/sshd_config editieren. Dort nach PasswordAuthentication suchen. Das sieht etwa so aus:
```
#PasswordAuthentication yes
```
Das ändern wir in
```
PasswordAuthentication no
```
## ufw installieren

```
sudo apt-get install ufw
sudo ufw allow ssh
sudo ufw show added
```
Das Resultat sollte so aussehen:
```
Added user rules (see 'ufw status' for running firewall):
ufw allow 22/tcp
```
**Achtung**: Wenn das Resultat anders aussieht, die Firewall nicht anschalten. Sonst wird Ihr System unter Umständen unbrauchbar.

Einschalten, wenn alles OK ist:
```
sudo ufw enable
```
## Automatische Security-Patches

Installation:
```
sudo apt install unattended-upgrades apt-listchanges
```

Datei `/etc/apt/apt.conf.d/50unattended-upgrades`, die ist eigentlich schon korrekt auf dem frischen System installiert, wir prüfen nur nach, ob der Abschnitt über Unattended-Upgrade auch tatsächlich die gewünschten Informationen enthält:
```
Unattended-Upgrade::Allowed-Origins {
	"${distro_id}:${distro_codename}";
	"${distro_id}:${distro_codename}-security";
	// Extended Security Maintenance; doesn't necessarily exist for
	// every release and this system may not have it installed, but if
	// available, the policy for updates is such that unattended-upgrades
	// should also install from here by default.
	"${distro_id}ESMApps:${distro_codename}-apps-security";
	"${distro_id}ESM:${distro_codename}-infra-security";
//	"${distro_id}:${distro_codename}-updates";
//	"${distro_id}:${distro_codename}-proposed";
//	"${distro_id}:${distro_codename}-backports";
};
```

Datei `/etc/apt/apt.conf.d/10periodic`:

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "0";
APT::Periodic::AutocleanInterval "0";
APT::Periodic::Unattended-Upgrade "1";
```

## Mail-Notifications

```
sudo apt install msmtp ca-certificates
```
Zur Konfiguration muss man eine Datei /etc/msmtprc anlegen und folgendes einfügen:
(Alles nach den Pfeilen <- darf nicht in die Datei mit übernommen werden.
```
# Allgemeine Optionen
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt  <- das wurde mit ca-certificates installiert. Überprüfen, ob die Datei existiert
logfile        ~/.msmtp.log
tls_starttls   off                                 <- Anpassen
tls_certcheck  off                                 <- Zertifikat selbstsigniert = off, von einer öffentlichen CA = on

# Konfiguration für benutzer@anbieter.com
account        unattendedupgrades                  <- Der Account wurde mit UnattendedUpgrade angelegt
host           mail.ihredomain.de                  <- Ihr Mailserver
port           465                                 <- SMTP/SSL = 465, STARTTLS = 578
from           <info@ihredomain.de>                <- Ihre Absender-Adresse
user           serversupport@ihredomain.de         <- Ihr BenutzerAccount
password       ihrpasswort                         <- Ihr Passwort

# Standard Account für E-Mails
account default : unattendedupgrades
```
### Test der Mailanbindung

Statt empfaenger@mailserver.com muss Ihre Mailadresse rein.

```
echo -e "Subject: Testmail \n\n Nachricht kommt hier hin \n\n" |sendmail empfaenger@mailserver.com
```

## Nginx

Dazu braucht es nur zwei Befehle:

```
sudo apt install nginx
sudo ufw allow 'Nginx Full'
```

Danach sollte die Default-Seite von Nginx erscheinen, wenn man die IP-Adresse im Browser eingibt.

Dann den Ordner `/var/www/html` komplett löschen. Das ist die Default-Seite. Dann /etc/nginx/sites-enabled/default öffnen und mit folgendem Code ersetzen:

```
# Default server configuration
#
server {
	listen 80 default_server;
	listen [::]:80 default_server;
return 404;
}
```
Das heißt: jeder Request ohne Host Header wird einen 404 zurückliefern.

Die folgenden Abschnitte über Nginx sind eine Ergänzung zum Lehrvideo und werden dort nicht besprochen.

### Verhindern der Server-Header mit nginx-Versionsnummer

In /etc/nginx/nginx.conf:

```
http { 
  ...
  server_tokens off;
  ...
```
### Nginx-Debugging

Im Server-Block folgende zwei Zeilen:

```
rewrite_log on;
error_log /var/log/nginx/deinesite.error_log debug;
```

**Achtung!**

Der Log erzeugt extrem viel Output (200-500 Zeilen) pro Request. Auf keinen Fall auf einem System mit hoher Last ausprobieren. Und wenn doch, dann absolut kurzfristig.

### Fehler wegen zu großer Header

Wir hatten für eine Asp.net-Site folgenden Fehler im Debug-Log: 

```
upstream sent too big header while reading response header from upstream
```

Korrektur im Server-Block:

```
proxy_busy_buffers_size   512k;
proxy_buffers   4 512k;
proxy_buffer_size   256k;
```

## Docker

[Dokumentation der Docker-Installation](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

### Tools installieren

```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

### Key anlegen

```
wget https://download.docker.com/linux/ubuntu/gpg
sudo gpg -o /etc/apt/keyrings/docker.gpg --dearmor ./gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
rm gpg
```
### Paketquelle installieren

```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
### Installation der Docker Engine und Tools

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Test

```
sudo docker ps
```
Gegenwärtigen Benutzer zum Docker-User machen:

```
sudo usermod -aG docker $USER
```

Abschließender Test:

```
docker ps
```

## Sql Server

Erst einen Ordner anlegen:
```
sudo mkdir /var/opt/mssql
sudo chmod 777 /var/opt/mssql 
```
Dann den Container laden:

```
docker run --name sqlexpress \
 -e 'ACCEPT_EULA=Y' \
 -e 'SA_PASSWORD=Hola$qlServer123' \
 -e 'MSSQL_PID=Express' \
 -v /var/opt/mssql:/var/opt/mssql \
 --restart unless-stopped \
 --network=host \
 -d mcr.microsoft.com/mssql/server:2019-latest
```

Test mit 

```
docker ps
```

## MariaDb

```
sudo apt update
sudo apt install mariadb-server
sudo mysql_secure_installation
```
mysql_secure-installation startet einen Assistenten. Die Beantwortung der Fragen sehen Sie im Video.

Anlegen des admin-Accounts:

```
sudo mariadb
GRANT ALL ON *.* TO 'admin'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit
```

Die Zeilen müssen Sie nacheinander eingeben, weil mariadb eine Db-Konsole öffnet.

## Remote-Administration
Wenn Sie später ein VPN für die Administration einrichten, benötigen Sie einen Benutzer, der aus dem Netzwerk stammt. Angenommen, Sie haben ein VPN mit dem Subnetz 10.1.1.0/24 eingerichtet. Dann legt man den remote Admin so an:

```
GRANT ALL ON *.* TO 'admin'@'10.1.1.%' IDENTIFIED BY 'ihrpassword' WITH GRANT OPTION;
```

Nun kann sich der admin aus diesem Subnetz anmelden.

Dazu müssen Sie die ensprechenden Ports innerhalb des Subnetzes freigeben:

```
sudo ufw allow from 10.1.1.0/24 to any port 3306
sudo ufw allow from 10.1.1.0/24 to any port 1433
```

Port 3306 ist für MariaDb, Port 1433 für SqlServer.

### OpenVPN

Im Video empfehle ich noch die Verwendung von OpenVPN. Allerdings sind wir mit unserer Firma mittlerweile auf WireGuard umgestiegen - siehe die [Installations-Anleitung](https://github.com/LinkedInLearning/asp-net-hosting-linux-2702073/blob/main/README.md#wireguard). Der Grund dafür ist, dass WireGuard Teil des Linux-Kernels ist und daher vom Linux-Team gewartet wird. Laut Kommentar von Linus Thorvalds besticht WireGuard durch seine Einfachheit und einer sehr geringen Angriffsoberfläche.

[Ausführliche Beschreibung](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04-de)

Wenn Sie OpenVPN schon auf Windows-Servern genutzt haben: die Konfiguration funktioniert auch auf Linux (Zeilenenden in der Konfigurationsdatei auf Linux umstellen). 

In der Datei /etc/sysctl.conf einfügen:
```
net.ipv4.ip_forward=1
```

Der ipv4 forward ist notwendig, um die Requests im lokalen Netzwerk zu sehen.

Ufw-Konfiguration ergänzen:
```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

dann das OpenVPN-Port öffnen:

```
sudo ufw allow 1194/udp
```


Neustart von ufw ist mit 

```
sudo ufw disable
sudo ufw enable
```

möglich.

### Konfigurationsdateien
Die Konfigurationsdatei auf dem Server heißt /etc/openvpn/xxxx.conf, wobei xxxx der Name der Konfiguration ist, also zum Beispiel _server_.

Ein funktionierendes Beispielpaar für eine Server- und Client-Konfiguration finden Sie hier:

[server.conf](https://github.com/LinkedInLearning/asp-net-hosting-linux-2702073/blob/main/client.ovpn)

[client.ovpn](https://github.com/LinkedInLearning/asp-net-hosting-linux-2702073/blob/main/client.ovpn)

### Easy-Rsa

Das ist ein Script, die Sie unter Linux ausführen können. Zum Einrichten folgen Sie [dieser Beschreibung](#anlegen-einer-ca) und dem Kapitel "Certificate Authority einrichten". Sie benötigen das Zertifikat der CA (ca.crt) auf der Client- und Serverseite sowie die beiden Dateien des Client-Zertifikats auf der Client-Seite.

### OpenVPN Start

```
systemctl start openvpn@server
```

Die Endung @server matcht mit dem Namen der .conf-Datei.

Überprüfen:

```
systemctl status openvpn@server
ip addr show tun0
```

### Wireguard
Wireguard ist in Ubuntu vorinstalliert. Was man braucht, sind die Tools:

```
apt install wireguard-tools
```

Dann braucht es im Verzeichnis /etc/wireguard für jede Verbindung eine Datei. Eine Verbindung ist ein virtuelles Device, die haben Namen wie wg0, wg1 etc. Die dafür nötige Datei muss heißen wie das Interface, z.B. `wg0.conf`. Eine solche Konfigurationsdatei sieht wie folgt aus:

```
[Interface]
PrivateKey = base64codierterPrivateKey.
Address = 10.8.2.1/32
ListenPort = 51821
 
#
# Reload Config:
#
# wg syncconf wg0 <(wg-quick strip wg0)
#
 
# Mirko
[Peer]
PublicKey = base64codierterPublicKey des Teilnehmers
AllowedIPs = 10.8.2.2/32
# Weitere Peers (=Clients)
```

Um die Keys anzulegen:

```
wg genkey > private
wg pubkey < private > public
```

Die Dateien können hinterher weggeworfen werden, wenn man die Keys entsprechend verteilt hat. Wenn sie nicht weggeworfen werden, muss die private-Datei mit chmod auf 600 gesetzt werden, sodass die nur vom Benutzer selbst gelesen und geschrieben werden kann.

Der PrivateKey wird nun an entsprechender Stelle in die Config eingefügt und der PublicKey wird in der Config-Datei auf der Gegenseite eingefügt.

Wenn Sie ein Subnetz mit 10.8.2.0/24 anleen, dann können Sie dem Server die Address 10.8.2.1/32 geben. Für den Peer vergeben Sie die 10.8.2.2, für mehrere Peers geht's dann nach oben weiter mit 3,4 etc.

Das Port 51821 muss in ufw für Udp freigegeben werden (Tcp ist nicht nötig):

```
sudo ufw allow 51821/udp
```

Mit 

```
wg-quick up wg01
```
wird das Interface gestartet und müsste dann einen reboot überstehen.

#### WireGuard Windows

Unter Windows gibt es den WireGuard-Client. Sehr komfortabel. Sie legen einen Tunnel mit "Einen leeren Tunnel hinzufügen" an:

![grafik](https://github.com/LinkedInLearning/asp-net-hosting-linux-2702073/assets/43751391/cbd8b38c-ca20-481b-b0a7-1719c04320b6)

Den kann man dann bearbeiten und dabei sehen Sie eine Datei, die der Datei auf dem Server ähnelt und die Sie editieren, sodass sie am Ende so aussieht:

```
[Interface]
PrivateKey = EinNeuerBase64CodierterPrivateKey
Address = 10.8.2.2/32

[Peer]
PublicKey = DerPublicKeyDesServers
AllowedIPs = 10.8.2.1/32
Endpoint = 88.198.22.19:51821
```

Der Unterschied zur Server-Konfiguration ist, dass wir für den Peer einen Endpoint definieren, das ist die im Internet öffentliche Adresse des Servers mit dem Port, das wir in der Server-Config definiert haben, in dem Fall 51821.

Die eigene Adresse ist dann die 2, 3, 4 etc., wie im Server dem eigenen Public Key zugewiesen. Die Adresse des Servers ist die 1, das muss matchen mit der Adresse in der Server-Config.

## Certbot

### Installation

```
sudo apt-get update 
sudo apt-get install certbot python3-certbot-nginx
```
### Einrichten eines Zertifikats
Es muss eine Site mit Http erreichbar sein. Die Server-Datei für Nginx muss entsprechend eingerichtet sein. Minimalkonfiguration aus dem Beispiel:

```
server
{
    server_name ihre.domaene.de;
    root /var/www/testsite/app;
	index index.html;

    listen 80;
}
```
Dann certbot aufrufen:

```
sudo certbot --nginx -d ihre.domaene.de
```

### Zertifikat zurückziehen

```
certbot revoke --cert-name example.com
```

Sie werden gefragt, ob Sie die Zertifikatsdateien löschen wollen. Das können Sie bestätigen.

Die Zertifikate liegen unter /etc/letsencrypt.

Renew-Anweisung liegt in /etc/cron.d/certbot:

```
0 \*/12 \* \* \* root test -x /usr/bin/certbot -a <span dir="">!</span> -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew
```

Der Cron-Eintrag wurde vom certbot selbst eingetragen. Die renewals liegen in /etc/letsencrypt/renewal

## ASP.NET

Installation:
```
sudo apt install dotnet6
```
Test

```
dotnet --version
```

## Installation der Anwendung

Zum Entpacken können wir das .7z-Format (7zip), .zip (gzip) oder tar.gz verwenden. Bei den ersteren muss man das entsprechende Tool installieren, also 7zip oder gzip. Ich verwende 7zip, das ist extrem schnell. Das wird folgendermaßen installiert:

```
sudo apt install p7zip-full
```
Dann kann man mit 

```
7z x <dateiname.7z>
```
die Datei in den gegenwärtigen Ordner extrahieren.

## Server-Datei und Service-Konfiguration

Für die Server-Datei für Nginx können Sie folgendes Template verwenden:

```
server {
    listen        80;
    server_name   www.meinhost.de;                   <-- Anpassen
    location / {
        proxy_pass         http://127.0.0.1:5000;    <-- Port muss dem Port in der .service-Datei matchen
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

Ein Symbolic Link auf diese Datei muss im Verzeichnis `/etc/nginx/sites-enabled` angelegt werden. Dann Nginx testen und Konfiguration neu laden:

```
sudo nginx -t
sudo nginx -s reload
```

Test, welches Port im 50xx-Bereich noch frei ist:

```
lsof -i -P -n | grep LISTEN | grep :50
```

Template für die Service-Datei:

```
[Unit]
Description=IhreApp          <-- Name anpassen
[Service]
WorkingDirectory=/var/www/ihreapp/app <-- Pfad anpassen
#ExecStartPre=/usr/bin/dotnet /var/www/waitforsqlserver/WaitForSqlServer.dll <-- Wenn Sql Server gebraucht wird
ExecStart=/usr/bin/dotnet /var/www/dotnet01/app/IhreApp.dll   <-- Name anpassen
Restart=always
# Restart service after 10 seconds if the dotnet service crashes
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=ihreapp <-- anpassen
User=ihrusername                 <-- anpassen, User vorher anlegen
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
Environment=ASPNETCORE_URLS=http://+:5000                 <-- Port anpassen.
[Install]
WantedBy=multi-user.target
```

Dann aufrufen:

```
sudo systemctl enable /pfad/auf/Ihre/.service-datei/ihreapp.service
sudo systemctl start ihreapp.service
```

Nach Änderungen an der .service-Datei:

```
systemctl daemon-reload
```

Nach Änderungen der Anwendung (Deployment):

```
systemctl restart ihreapp.service
```

Überprüfen der Startphase der Anwendung:

```
journalctl -xeu ihreapp
```

Der zweite Parameter muss dem Parameter SyslogIdentifier in der .service-Datei entsprechen.

## Verzeichnisberechtigungen

Dazu benötigen Sie unter Umständen Access Control Lists. Die Tools dafür installieren Sie mit

```
sudo apt-get install acl
```

1. Benutzer anlegen
```
sudo useradd -s /usr/sbin/nologin aspnethosting
```

2. www-data Ihrer Benutzergruppe hinzufügen

```
sudo usermod -a -G aspnethosting www-data
```

3. Den neuen Benutzer zum Owner des Verzeichnisbaums machen

```
sudo chown -R aspnethosting:aspnethosting /var/www/aspnethosting/app
```

4. Datei- und Verzeichnismodi setzen
```
#!/bin/bash
find $1 -type d -exec chmod 750 {} \;
find $1 -type f -exec chmod 640 {} \;
```

5. Access Control Lists für non-root Accounts anlegen

```
sudo setfacl -R -m u:ihrusername:rwx /var/www/aspnethosting/app
sudo setfacl -Rd -m u:ihrusername:rwx /var/www/aspnethosting/app
sudo setfacl -R -m m:rwx /var/www/aspnethosting/app   
```

Das kann mit in den Batch für die Verzeichnismodi:

```
setfacl -R -m u:ihrusername:rwx $1
setfacl -Rd -m u:ihrusername:rwx $1
setfacl -R -m m:rwx /$1  
```
Die Batch-Datei ausführbar machen:

```
sudo chmod +x /var/www/aspnethosting/setperm
```

### Test der Site nach dem Einrichten

```
sudo -u aspnethosting /bin/bash
dotnet /var/www/aspnethosting/app/AspNetHosting.dll &
```

Prozess beenden:

```
ps
```
Zeigt die laufenden Prozesse mit Nummer an

```
kill xxxx
```

Prozess mit Nummer abschießen.

## Locations

```
location = /robots.txt {
    alias /var/www/aspnethosting/robots.txt;
}
```
### Zugriff auf Locations begrenzen
Die Idee ist, die Verwaltungsoberfläche /umbraco nicht über den öffentlich bekannten Host zuzulassen, sondern nur über technische Hosts, die dann zum Beispiel nur über das lokale Netzwerk erreichbar sind.
Das geht so, dass man zwei Server-Dateien anlegt, eine für die technische Domäne und eine für die öffentliche Domäne. Die legt man mit Port 80 an und lässt Certbot die SSL-Konfiguration eintragen. Für die technische Domäne trägt man dann folgendes ein:

```
    location = /robots.txt {
        alias /var/www/disallow/robots.txt ;
    }	
```
Das ist eine Version der robots.txt, die die Indizierung ablehnt. Die kann geladen werden, weil das von www-data ausgeliefert wird und www-data ein Mitglied der Benutzergruppe der Site ist.

Für die öffentliche Domäne trägt man folgendes ein:

```
    location = /robots.txt {
        alias /var/www/aspnethosting/app/wwwroot/robots.txt ;    <-- erlaubt die Indizierung
    }
    location = /umbraco {                                        <-- Backoffice wird nicht gefunden
        return 404;
    }
```

Hier ist ebenfalls eine gute Idee:

```
    location /umbraco {
        # Erlaube Zugriff von einer IP und zusätzlich von einem IP-Bereich:
        allow 1.2.3.4;
        allow 192.168.0.0/24;
        deny all;
    }
```

[Mehr Info über Locations](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)

## Backup

```
#!/bin/bash
docker exec -t sqlexpress /opt/mssql-tools/bin/sqlcmd -S localhost -U aspnethosting -P aspnethosting12 -Q "BACKUP DATABASE AspNetHosting TO DISK = '/var/opt/mssql/data/aspnethosting.bak';"
```

S = Servername
U = Benutzername im SqlServer
P = Passwort
Q = Befehl der ausgeführt werden soll

### Cron-Job

Eine Datei mit folgendem Inhalt anlegen:

```
0 3	* * *	root	/var/www/aspnethosting/config/do-backup > /var/www/aspnethosting/backup/backup-log.txt 2>&1
```

Uhrzeit, Batch-Name, Pfad für die Log-Datei anpassen.
Einen Link im Verzeichnis `/etc/cron.d` anlegen. 
**Achtung:** Der verwendete Link-name im Video wird nicht funktionieren. Dateien / Links in cron.d dürfen keinen Punkt im Namen haben. Sonst werden sie ignoriert. Siehe die [man-page zu diesem Thema](https://manpages.ubuntu.com/manpages/lunar/en/man8/cron.8.html):

_The file names must conform to the filename requirements of run-parts:  they  must  be  entirely  made  up  of letters,  digits  and  can  only  contain  the special signs underscores ('\_') and hyphens ('-'). Any file that does not conform to these requirements will not be executed by  run-parts._

## Basic Authentication für Staging Sites

Server-Datei ergänzen:

```
    auth_basic "Aspnet Staging Site";
    auth_basic_user_file /var/www/aspnethosting/config/.htpasswd;
```

Schritt 1:

```
sudo echo -n 'ihrusername:' >> /var/www/aspnethosting/config/.htpasswd
```

Schritt 2:

```
sudo openssl passwd -apr1 >> /var/www/aspnethosting/config/.htpasswd
```

Test und Reload nicht vergessen:

```
sudo nginx -t
sudo nginx -s reload
```

## Neue Sudo-User anlegen

1. User anlegen

```
sudo useradd -m -d /home/bob bob
```

2. Zu den Sudoern hinzufügen

```
sudo usermod -aG sudo bob
```

3. authorized_keys-Datei in `/home/bob/.ssh/authorized_keys` anlegen und Schlüssel _in eine Zeile_ pasten

```
ecdsa-sha2-nistp521 AAAAE2VjZHgaaanzviiiiieleZeichen== bob@aspnethosting
```

4. Dateirechte setzen

```
sudo chown -R bob:bob /home/bob
sudo chmod 600 /home/bob/.ssh/authorized_keys
```

5. Initiales Passwort für Sudo

```
sudo passwd bob
```
Das Passwort sollte bob sofort nach dem ersteb Einloggen ändern. Niemand kann eine SSH-Sitzung mit dem Passwort beginnen, wenn der Passwort-Login disabled ist, dennoch ist das sauberer.

### Löschen eines Users

```
sudo userdel -r bob
```

## Anlegen einer CA

[Link zur Easy-RSA-Site](https://github.com/OpenVPN/easy-rsa)

Link zu .tgz-Datei kopieren.

Download

```
wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.1.5/EasyRSA-3.1.5.tgz
```

Entpacken

```
tar -xvzf EasyRSA-3.1.5.tgz
```

Im Easy-RSA-Verzeichnis: vars.sample in vars umbenennen, Defaults anpassen und dann:

```
sudo chmod +x easyrsa
```

Initialisieren

```
./easyrsa init-pki
```

CA-Zertifikat anlegen:

```
./easyrsa build-ca
```

Certificate Request anlegen:

```
./easyrsa gen-req mirko
```

Signieren:

```
./easyrsa sign-req client mirko
```

Für Windows eine .p12-Datei erzeugen:

```
openssl pkcs12 -export -out mirko.p12 -inkey /home/mirko/EasyRSA-3.1.5/pki/private/mirko.key -in /home/mirko/EasyRSA-3.1.5/pki/issued/mirko.crt
```

Passen sie die Pfade auf die Schlüssel- und Zertifikatsdateien an.

Server-Zertifikate können mit Easy-Rsa ebenso angelegt werden. Der Common Name (CN) muss beim Anlegen des Requests dem Hostnamen entsprechen. 

```
./easyrsa sign-req server yourservername
```

Das Schlüsselpaar kann dann in Nginx verwendet werden:

```
    ssl_certificate /pfad/zu/Ihrer/.crt-Datei/yourservername.crt;
    ssl_certificate_key /pfad/zu/Ihrer/.key-Datei/yourservername.key;
    listen 443 ssl http2;
    # Wenn Ihr Schlüssel passwortgeschützt ist,
    # das Passwort in eine Datei legen und diese Zeile auskommentieren:
    # ssl_password_file /var/lib/nginx/ssl_passwords;
    # Zugriffsschutz für die Datei nicht vergessen!
```

### Zurückziehen von Zertifikaten

Das dürfte beim Ausscheiden von Mitarbeitern von Bedeutung sein. Die ausgestellten Zertifikate müssen als .crt-Datei unbedingt aufgehoben werden!

./easyrsa revoke xxxxx      <- Name des Zertifikats = Dateiname ohne Endung
./easyrsa gen-crl

Dann liegt eine crl.pem im Verzeichnis pki. In der Nginx-Serverdatei muss die zusätzliche Zeile rein:

```
	ssl_client_certificate /var/www/aspnethosting/config/ca.crt;
	ssl_crl /var/www/aspnethosting/config/crl.pem;
    ...
```

Nach Auslieferung der crl.pem Nginx reloaden.

## Autor

Mirko Matytschak

Mirko Matytschak programmiert schon seit den Vorversionen im Jahr 2000 mit .NET. Seit diesen ersten Tagen nutzt er als Software-Entwickler und -Berater Architekturen auf Basis von .NET und entwickelt Software auf Basis von C#.
Er ist CEO und CTO der [FORMFAKTEN GmbH](https://www.formfakten.de) und Entwickler des relationalen Mapping-Tools [.NET Data Objects (NDO)](https://www.netdataobjects.de).

Sehen Sie sich andere Kurse des Autors auf [LinkedIn Learning](https://www.linkedin.com/learning/instructors/mirko-matytschak) an.

[0]: # (Replace these placeholder URLs with actual course URLs)
[lil-course-url]: https://www.linkedin.com/learning/asp-dot-net-hosting-auf-linux
[lil-thumbnail-url]: https://media.licdn.com/dms/image/D560DAQEDgZA1OT6Msg/learning-public-crop_675_1200/0/1698145672690?e=2147483647&v=beta&t=qdKkKLDIjxCN38r1lP3hG1ckA7v2UUIYY7lMdIUQaQE
