# Benutzer- und Administratorrechte im VM Cloud System

## Rollenübersicht

In unserem VM Cloud System gibt es zwei Hauptrollen:

- **Benutzer (User)**: Normale Studierende, die eigene virtuelle Maschinen verwalten.
- **Administrator (Admin)**: Verantwortliche, die das gesamte System und alle Benutzer verwalten.

Jede Rolle hat genau definierte Rechte, um einen sicheren und klar strukturierten Betrieb zu gewährleisten.

---

## Rechte der Benutzer (User)

Ein Benutzer hat folgende Möglichkeiten:

- **Anmelden (Login)**  
  Der Benutzer authentifiziert sich mit Benutzername und Passwort.

- **Eigene virtuelle Maschinen auflisten**  
  Der Benutzer kann nur seine eigenen VMs sehen, nicht die von anderen Benutzern.

- **Eigene VMs starten**  
  Der Benutzer kann seine eigenen VMs einschalten.

- **Eigene VMs stoppen**  
  Der Benutzer kann seine eigenen VMs herunterfahren.

- **Eigene VMs neu starten**  
  Der Benutzer kann seine eigenen VMs neu starten.

- **Eigene VMs löschen (optional)**  
  Je nach Systemkonfiguration darf der Benutzer seine eigene VM löschen.

- **Eigene VMs konfigurieren (optional)**  
  Der Benutzer kann Einstellungen wie RAM, CPU und Netzwerkadapter seiner eigenen VMs anpassen.

- **Fernzugriff auf eigene VMs über noVNC**  
  Der Benutzer kann sich über den Browser auf seine eigene VM verbinden.

- **Neue VM aus Vorlage erstellen**  
  Der Benutzer kann aus bereitgestellten OVA-Templates neue eigene VMs erstellen.

---

## Rechte der Administratoren (Admin)

Ein Administrator hat erweiterte Rechte:

- **Anmelden (Login Adminbereich)**  
  Der Administrator authentifiziert sich mit Admin-Zugangsdaten.

- **Alle VMs im System auflisten**  
  Der Administrator kann alle existierenden VMs aller Benutzer sehen.

- **Beliebige VMs starten, stoppen und neu starten**  
  Der Administrator kann jede VM im System verwalten.

- **Beliebige VMs löschen**  
  Der Administrator kann jede VM löschen, unabhängig vom Besitzer.

- **Neue VMs für beliebige Benutzer erstellen**  
  Der Administrator kann neue VMs erzeugen und bestimmten Benutzern zuordnen.

- **Benutzerliste verwalten**  
  Der Administrator kann alle Benutzer einsehen.

- **Benutzer erstellen und deaktivieren**  
  Der Administrator kann neue Benutzerkonten erstellen oder bestehende deaktivieren.

- **Passwörter für Benutzer zurücksetzen**  
  Der Administrator kann vergessene oder verlorene Passwörter zurücksetzen.

- **Vorlagen (OVA-Dateien) verwalten**  
  Der Administrator kann die Liste der verfügbaren VM-Templates aktualisieren.

- **Systemlogs einsehen (optional)**  
  Der Administrator kann Aktivitäten- und Ereignisprotokolle einsehen, um das System zu überwachen.

---

## Zusammenfassung

- Benutzer : Verwaltung und Zugriff auf eigene VMs |
- Administrator : Komplette Verwaltung des Systems, aller VMs und aller Benutzer |

Diese klare Trennung der Aufgabenbereiche stellt sicher, dass jeder Benutzer nur auf seine Ressourcen zugreift und das System sicher und effizient bleibt.

      +---------+                      +---------+
      |  User   |                      |  Admin  |
      +---------+                      +---------+
           |                                |
    +----------------+              +----------------------+
    | Login          |              | Admin Login          |
    +----------------+              +----------------------+
           |                                |
    +----------------+              +----------------------+
    | View Own VMs   |              | View All VMs         |
    +----------------+              +----------------------+
           |                                |
    +----------------+              +----------------------+
    | Start/Stop VMs |              | Manage Any VM        |
    +----------------+              +----------------------+
           |                                |
    +----------------+              +----------------------+
    | Delete/Config  |              | Manage Users         |
    +----------------+              +----------------------+
           |                                |
    +----------------+              +----------------------+
    | Access via     |              | Manage Templates     |
    | noVNC          |              +----------------------+
    +----------------+                      |
           |                                |
    +----------------+              +----------------------+
    | Create New VM  |              | View Logs (Optional) |
    +----------------+              +----------------------+
