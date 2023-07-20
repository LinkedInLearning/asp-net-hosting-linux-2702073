# ASP.NET Hosting in Linux

Dies ist das Repository für den **LinkedIn Learning** Kurs `[COURSENAME]`. Den gesamten Kurs finden Sie auf [LinkedIn Learning][lil-course-url].

![COURSENAME][lil-thumbnail-url] 

[COURSEDESCRIPTION]

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

```
ssh-keygen -f ./ihrname -t ecdsa -b 521
```
Das erzeugt ihrname und ihrname.pub. Natürlich können Sie statt ihrname Ihren Namen verwenden...

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





### Autor

Mirko Matytschak

Software-Entwickler und -Berater

Sehen Sie sich andere Kurse des Autors auf [LinkedIn Learning](https://www.linkedin.com/learning/instructors/name_des_autors) an.

[0]: # (Replace these placeholder URLs with actual course URLs)
[lil-course-url]: https://www.linkedin.com
[lil-thumbnail-url]: https:
