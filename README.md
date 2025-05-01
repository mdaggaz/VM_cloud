# Masterprojekt – VM Cloud (änderbar je nach Bedarf)

Nach einer theoretischen Analyse des Projekts **VM Cloud** wurde ein technischer Plan erstellt, der als Grundlage für die Implementierung dient. Einige Entscheidungen können sich im Verlauf des Projekts ändern, je nach Anforderungen, neuen Erkenntnissen oder Rückmeldungen. Diese Datei dokumentiert den aktuellen Stand und die geplante Richtung.

## 1. Projektziel

Ziel ist die Entwicklung einer zentralen Webanwendung zur Verwaltung virtueller Maschinen (VMs), die auf einem VirtualBox-Server laufen. Benutzer können sich über den Browser anmelden, ihre eigenen VMs sehen, neue VMs aus `.ova`-Vorlagen erstellen, bestehende VMs starten, stoppen, konfigurieren und löschen. Der Zugriff auf den Desktop der VM erfolgt direkt im Browser über eine noVNC-Integration.

Das System stellt sicher, dass:
- jede Person nur Zugriff auf ihre eigenen VMs hat,
- die Kommunikation und Verwaltung durch moderne Sicherheitsmechanismen geschützt ist,
- die gesamte Verwaltung über eine benutzerfreundliche Weboberfläche erfolgt.

---

## 2. Vorgesehene Technologien

### 🖥️ Backend
- Sprache: **Python**
- Framework: **FastAPI**
- Authentifizierung: **JWT** mit Rollenprüfung
- VM-Steuerung: über `VBoxManage`
- VNC-Zugriff: über **websockify** (dynamisch gestartet)
- VM-Erstellung: aus `.ova`-Dateien im Ordner `OS_VM_CLOUD/`
- VMs gespeichert unter: `VMS_PER_USER/<username>/`

### 🌐 Frontend
- Framework: **React** (Vite)
- Styling: **TailwindCSS**
- Token-Speicherung: `localStorage`
- Zugriffsschutz: `PrivateRoute`
- Anzeige der VMs mit Aktionen: Start, Stop, Visualisieren
- Visualisierung über `vnc.html`

---
## 3.Benutzerrollen

[Benutzer- und Administratorrechte im VM Cloud System](Benutzer_und_Administratorrechte.md)

---
## 4. Sicherheit

- Authentifizierung mit JWT + Rollentrennung
- Jeder Benutzer kann nur seine eigenen VMs sehen und verwalten
- Überprüfung des Benutzernamens bei jeder API-Anfrage
- Dynamisches Port-Forwarding für VNC:  [Verwaltung der VNC-Ports für virtuelle Maschinen mit VirtualBox](Verwaltung_VNC-Ports_VMs.md)

  
---
## 5. Wie benutzt ein Benutzer das System
Ein Benutzer meldet sich über die Weboberfläche mit seinem Benutzernamen und Passwort an. Nach erfolgreicher Authentifizierung erhält er ein JSON Web Token (JWT), das im Browser gespeichert wird und für alle weiteren Anfragen erforderlich ist. Auf dem Dashboard sieht der Benutzer nur seine eigenen virtuellen Maschinen. Er kann bestehende VMs starten, stoppen, löschen oder konfigurieren.

Wenn der Benutzer eine neue virtuelle Maschine erstellen möchte, muss er kein eigenes Image hochladen. Stattdessen wählt er eine vorhandene .ova-Vorlage aus, die bereits zentral auf dem Server im Ordner OS_VM_CLOUD/ gespeichert ist. Diese Vorlagen beinhalten z. B. Ubuntu, Windows10 oder REMnux. Die ausgewählte Vorlage wird automatisch importiert und in einem persönlichen Verzeichnis unter VMS_PER_USER/<benutzername>/ gespeichert.

Nach dem Start der VM wird ein freier VNC-Port automatisch zugewiesen. Der Benutzer kann dann über die Weboberfläche den Desktop der VM direkt im Browser mit noVNC anzeigen lassen – ohne zusätzliche Software installieren zu müssen. Alle Schritte sind über eine intuitive Benutzeroberfläche ausführbar.

---
## 6. Zukünftige Erweiterungen

### 6.1 Datenbank

Von Anfang an wird dieses System mit einer Datenbank umgesetzt, zum Beispiel mit ***PostgreSQL*** oder ***MongoDB***, je nach technischer Passung. Diese Entscheidung kann in Zukunft angepasst werden, falls bestimmte Anforderungen seitens der Hochschule oder des Projekts dies notwendig machen.

### 6.2 Finaler Deployment-Prozess

Die Anwendung soll auf einem zentralen Hochschulserver gehostet werden. Der Zugriff erfolgt ausschließlich über das interne Netz oder VPN. Ein Reverse-Proxy (z. B. Nginx) mit SSL-Zertifikat (Let’s Encrypt) ist möglich. Die genaue Deployment-Methode hängt von den endgültigen Anforderungen ab.

---
