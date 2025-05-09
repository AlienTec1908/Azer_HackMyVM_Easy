# Azer - HackMyVM Writeup

![Azer VM Icon](Azer.png)

Dieses Repository enthält das Writeup für die HackMyVM-Maschine "Azer" (Schwierigkeitsgrad: Easy), erstellt von DarkSpirit. Ziel war es, initialen Zugriff auf die virtuelle Maschine zu erlangen und die Berechtigungen bis zum Root-Benutzer zu eskalieren.

## VM-Informationen

*   **VM Name:** Azer
*   **Plattform:** HackMyVM
*   **Autor der VM:** DarkSpirit
*   **Schwierigkeitsgrad:** Easy
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Azer](https://hackmyvm.eu/machines/machine.php?vm=Azer)

## Writeup-Informationen

*   **Autor des Writeups:** Ben C.
*   **Datum des Berichts:** 30. April 2024
*   **Link zum Original-Writeup (GitHub Pages):** [https://alientec1908.github.io/Azer_HackMyVM_Easy/](https://alientec1908.github.io/Azer_HackMyVM_Easy/)

## Kurzübersicht des Angriffspfads

Der Angriff auf die Azer-Maschine umfasste die folgenden Schritte:

1.  **Reconnaissance & Port Scanning:**
    *   Identifizierung der Ziel-IP (`192.168.2.114`) mittels `arp-scan`.
    *   Ein `nmap`-Scan offenbarte zwei offene Ports: HTTP (80, Apache 2.4.57, Titel "LÖSEV") und HTTP (3000, Node.js Express, Titel "Login Page").
2.  **Web Enumeration:**
    *   `nikto` und `gobuster` auf Port 80 zeigten eine statische Webseite (HTTrack-Kopie) ohne relevante Schwachstellen für den Einstieg.
    *   Der Fokus verlagerte sich auf die Node.js-Anwendung (Login-Seite) auf Port 3000.
3.  **Initial Access (Command Injection):**
    *   Eine Fehlermeldung bei einem Login-Versuch auf Port 3000 ("Error executing bash script: Command failed: /home/azer/get.sh admin admin...") offenbarte, dass Benutzereingaben an ein Shell-Skript (`/home/azer/get.sh`) übergeben werden und der Benutzer `azer` existiert.
    *   Eine Command Injection-Schwachstelle wurde im `Username`-Feld des Login-Formulars ausgenutzt. Durch Einfügen von `;` gefolgt von einem Python-Reverse-Shell-Payload wurde eine interaktive Shell als Benutzer `azer` erlangt.
    *   Die User-Flag (`/home/azer/user.txt`) wurde gelesen.
4.  **Post-Exploitation Enumeration (as azer):**
    *   `sudo -l` zeigte, dass `sudo` nicht verfügbar war. Andere Enumerationsschritte (SUID, Capabilities, Crontab) ergaben keine direkten Eskalationspfade.
    *   `ip a` zeigte die Existenz eines internen Docker-Netzwerks (`10.10.10.0/24`).
5.  **Privilege Escalation (Password Guessing via Internal Service):**
    *   Ein Webdienst wurde auf einem internen Container unter `10.10.10.10:80` entdeckt (mittels `curl http://10.10.10.10`).
    *   Der Dienst antwortete mit `.:.AzerBulbul.:.`, was als Hinweis auf das Root-Passwort interpretiert wurde.
    *   Mit `su root` und dem Passwort `AzerBulbul` wurde erfolgreich Root-Zugriff erlangt.
6.  **Flags:**
    *   Die User-Flag wurde als `azer` gelesen.
    *   Die Root-Flag wurde aus `/root/root.txt` gelesen.

## Verwendete Tools

*   `arp-scan`
*   `vi`
*   `nmap`
*   `nikto`
*   `gobuster`
*   `curl`
*   `python3`
*   `nc` (netcat)
*   `find`
*   `ss`
*   `getcap`
*   `cat`
*   `ip`
*   `su`

## Identifizierte Schwachstellen (Zusammenfassung)

*   **Command Injection in Node.js-Anwendung:** Benutzereingaben aus dem Login-Formular auf Port 3000 wurden unsicher an ein Backend-Shell-Skript (`/home/azer/get.sh`) übergeben, was die Ausführung beliebiger Befehle als Benutzer `azer` ermöglichte.
*   **Preisgabe von Informationen durch Fehlermeldungen:** Eine detaillierte Fehlermeldung deckte den Pfad des Skripts, die Übergabe von Argumenten und den Benutzernamen `azer` auf.
*   **Schwaches, aus Hinweis ableitbares Root-Passwort:** Ein interner Dienst im Docker-Netzwerk gab einen direkten Hinweis (`AzerBulbul`) auf das Root-Passwort.

## Flags

*   **User Flag (`/home/azer/user.txt`):** `0d2856d69dc348b3af80a0eed67c7502`
*   **Root Flag (`/root/root.txt`):** `b5d96aec2d5f1541c5e7910ccab527d8`

---

**Wichtiger Hinweis:** Dieses Dokument und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Techniken und Werkzeuge sollten nur in legalen und autorisierten Umgebungen (z.B. eigenen Testlaboren, CTFs oder mit expliziter Genehmigung) angewendet werden. Das unbefugte Eindringen in fremde Computersysteme ist eine Straftat und kann rechtliche Konsequenzen nach sich ziehen.

---
*Bericht von Ben C. - Cyber Security Reports*
