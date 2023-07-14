# ASP.NET Hosting in Linux

Dies ist das Repository für den **LinkedIn Learning** Kurs `[COURSENAME]`. Den gesamten Kurs finden Sie auf [LinkedIn Learning][lil-course-url].

![COURSENAME][lil-thumbnail-url] 

[COURSEDESCRIPTION]

## sudo oder root?
Enablen des root-Accounts:
```
sudo passwd root
```
Disablen des root-Accounts:
```
sudo passwd -l root
```
Auch wenn ich meine Systeme mit root betreibe, versuche ich, im Folgenden die Anweisungen mit sudo anzugeben. Sollte es irgendwo einmal fehlen, merken Sie es daran, dass Sie die Berechtigung für eine bestimmte Aktion nicht haben. Manchmal kommen auch etwas merkwürdige Fehlermeldungen, weil ein Lock nicht gesetzt werden kann, oder ähnliches. In dem Fall: Wenn sudo fehlt, ergänzen Sie es einfach und schauen, ob der Befehl dann funktioniert.

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




### Autor

Mirko Matytschak

Software-Entwickler und -Berater

Sehen Sie sich andere Kurse des Autors auf [LinkedIn Learning](https://www.linkedin.com/learning/instructors/name_des_autors) an.

[0]: # (Replace these placeholder URLs with actual course URLs)
[lil-course-url]: https://www.linkedin.com
[lil-thumbnail-url]: https:
