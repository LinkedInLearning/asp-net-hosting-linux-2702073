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

### Autor

Mirko Matytschak

Software-Entwickler und -Berater

Sehen Sie sich andere Kurse des Autors auf [LinkedIn Learning](https://www.linkedin.com/learning/instructors/name_des_autors) an.

[0]: # (Replace these placeholder URLs with actual course URLs)
[lil-course-url]: https://www.linkedin.com
[lil-thumbnail-url]: https:
