# Verwaltung der VNC-Ports für virtuelle Maschinen mit VirtualBox

## Warum dynamisches Port Forwarding erforderlich ist

In unserer Umgebung verwenden alle virtuellen Maschinen denselben VNC-Port intern (5900).  
Da mehrere VMs gleichzeitig laufen können, ist es unmöglich, auf dem Hostsystem denselben Port 5900 mehrfach zu öffnen.  
Ohne eine spezielle Konfiguration würden Benutzer auf falsche VMs zugreifen, was ein erhebliches Sicherheitsproblem darstellen würde.

Um dies zu verhindern, verwenden wir das dynamische Port Forwarding von VirtualBox.

Jede VM erhält auf dem Hostserver einen eigenen externen Port, der auf den internen Port 5900 der jeweiligen VM weitergeleitet wird.

Dies ermöglicht es:
- Mehrere VMs gleichzeitig zu betreiben
- Einen klaren Zugriff von noVNC auf die richtige VM zu gewährleisten
- Benutzer voneinander strikt zu isolieren

## Wie Port Forwarding umgesetzt wird

Beim Erstellen oder Starten einer VM wird eine Portweiterleitungsregel mit `VBoxManage` hinzugefügt.

Für jede VM wird ein einzigartiger externer Port (zum Beispiel 5901, 5902, 5903, usw.) auf den internen Port 5900 weitergeleitet.

Die Regel wird wie folgt gesetzt:
- Name der Regel: beliebig (z.B. "vnc")
- Protokoll: TCP
- Host-Port: eindeutig für jede VM (5901, 5902, 5903, ...)
- Guest-Port: 5900 (immer gleich)

Durch diese Technik wird bei einem Zugriff über noVNC direkt der richtige VM-Desktop geladen, ohne dass Benutzer Fehler machen können.

## Beispiel:

- VM1 (z.B. REMnuxVM-User1) ➔ Hostport 5901 ➔ Gastport 5900
- VM2 (z.B. WinXPVM-User2) ➔ Hostport 5902 ➔ Gastport 5900
- VM3 (z.B. KaliVM-User3) ➔ Hostport 5903 ➔ Gastport 5900

Wenn ein Benutzer sich verbindet, wird er durch noVNC auf die richtige VM und den richtigen Port geleitet.

## Umsetzung im Backend

Das Backend übernimmt folgende Aufgaben:
- Reservierung eines freien Hostports für jede neue VM
- Anwendung der Portweiterleitungsregel bei der VM-Erstellung
- Speicherung der Host-Port-Zuordnung in der Datenbank
- Überprüfung beim Verbinden über noVNC, dass der Benutzer nur seine eigene VM sehen kann

Durch diese Struktur ist ein sicherer und stabiler Betrieb von mehreren virtuellen Maschinen über eine zentrale Webanwendung möglich.
