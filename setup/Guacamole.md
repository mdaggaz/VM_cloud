# Apache Guacamole Installation und Konfiguration für VM-Zugriff (REMnux) via RDP

**Datum der Dokumentation:** 14. Mai 2025

**Ziel:** Installation von Apache Guacamole auf einem Ubuntu-Server, um auf eine virtuelle Maschine (REMnux VM auf VirtualBox) zuzugreifen, die auf einem anderen Windows-PC gehostet wird, unter Verwendung des RDP-Protokolls über einen Webbrowser.

**Umgebungsübersicht (Beispielkonfiguration):**

* **PC1 (Guacamole Server):**
    * Betriebssystem: Ubuntu 22.04.5 LTS (Bare-Metal-Installation)
    * IP-Adresse: `192.168.0.139` (Beispiel, ersetzen durch Ihre tatsächliche IP)
* **PC2 (Host der VM):**
    * Betriebssystem: Windows
    * IP-Adresse: `192.168.0.170` (Beispiel, ersetzen durch Ihre tatsächliche IP)
* **Ziel-VM (Virtuelle Maschine):**
    * Typ: REMnux (läuft auf VirtualBox auf PC2)
    * Verbindungsprotokoll: RDP (aktiviert über VRDE in VirtualBox)
    * VRDE-Port auf PC2: `3392` (Beispiel, anpassen falls abweichend)
    * VM-Benutzername: `remnux` (Beispiel)
    * VM-Passwort: `malware` (Beispiel)

---

## Detaillierte Schritte:

### Schritt 1: Abhängigkeiten installieren (auf PC1 - Ubuntu)

1.  Paketliste aktualisieren:
    ```bash
    sudo apt update
    ```
2.  Build-Werkzeuge und Kernabhängigkeiten für `guacd` (Guacamole Backend-Server), RDP-Unterstützung, Tomcat 9 (Webserver) und Java JDK installieren:
    ```bash
    sudo apt install -y build-essential libcairo2-dev libjpeg-turbo8-dev \
    libpng-dev libtool-bin libossp-uuid-dev libavcodec-dev libavformat-dev \
    libavutil-dev libswscale-dev libpango1.0-dev libssh2-1-dev \
    libtelnet-dev libvncserver-dev libwebsockets-dev libssl-dev \
    libvorbis-dev libwebp-dev ghostscript freerdp2-dev \
    tomcat9 tomcat9-admin tomcat9-user default-jdk
    ```

---

### Schritt 2: `guacamole-server` (guacd) herunterladen, kompilieren und konfigurieren (auf PC1)

1.  Guacamole-Version festlegen (wir haben 1.5.4 verwendet):
    ```bash
    export GUAC_VERSION=1.5.4
    ```
2.  Quellcode für `guacamole-server` herunterladen:
    * **Wichtiger Hinweis zur Kompatibilität:** Es wird dringend empfohlen, den Quellcode (`guacamole-server-X.Y.Z.tar.gz`) von der primären Apache-Download-Seite (`downloads.apache.org`) zu beziehen. Die offizielle SHA256-Summe für die Version 1.5.4 ist `e9313305de281973096828ad32e709497788582397062830007f017e888158a0`.
    ```bash
    wget "[https://dlcdn.apache.org/guacamole/$](https://dlcdn.apache.org/guacamole/$){GUAC_VERSION}/source/guacamole-server-${GUAC_VERSION}.tar.gz"
    # Optional: SHA256-Summe der heruntergeladenen Datei überprüfen
    # sha256sum guacamole-server-${GUAC_VERSION}.tar.gz 
    ```
3.  Archiv entpacken und in das Verzeichnis wechseln:
    ```bash
    tar -xzf guacamole-server-${GUAC_VERSION}.tar.gz
    cd guacamole-server-${GUAC_VERSION}/
    ```
4.  Konfigurationsskript ausführen (vor der Kompilierung):
    ```bash
    ./configure --with-init-dir=/etc/init.d
    ```
5.  Quellcode kompilieren (kann einige Zeit dauern):
    ```bash
    make
    ```
6.  Software installieren:
    ```bash
    sudo make install
    ```
7.  Cache der Shared Libraries aktualisieren:
    ```bash
    sudo ldconfig
    ```
8.  Installierte `guacd`-Version und unterstützte Protokolle überprüfen:
    ```bash
    /usr/local/sbin/guacd -v 
    ```
    (Sollte Version 1.5.4 anzeigen und RDP als unterstützt auflisten).
9.  Guacamole-Konfigurationsverzeichnis erstellen:
    ```bash
    sudo mkdir /etc/guacamole
    ```
10. `guacd`-Konfigurationsdatei erstellen (`/etc/guacamole/guacd.conf`):
    ```bash
    sudo nano /etc/guacamole/guacd.conf
    ```
    Inhalt der Datei:
    ```ini
    [daemon]
    pid_file = /var/run/guacd.pid
    log_level = info

    [server]
    bind_host = 127.0.0.1
    bind_port = 4822
    ```
11. Systemd-Dienstdatei für `guacd` erstellen (`/etc/systemd/system/guacd.service`):
    ```bash
    sudo nano /etc/systemd/system/guacd.service
    ```
    Inhalt der Datei:
    ```ini
    [Unit]
    Description=Guacamole Server Daemon
    After=network.target

    [Service]
    ExecStart=/usr/local/sbin/guacd -f -L info
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    ```
12. Systemd-Konfiguration neu laden, `guacd`-Dienst aktivieren und starten:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable guacd
    sudo systemctl start guacd
    sudo systemctl status guacd 
    ```
    (Sicherstellen, dass der Status `active (running)` ist).

---

### Schritt 3: `guacamole.war` Webanwendung in Tomcat installieren (auf PC1)

1.  **Sehr wichtiger Hinweis zu `guacamole.war` und Kompatibilität:**
    * Du hast eine `guacamole.war`-Datei (für Version 1.5.4) heruntergeladen, deren SHA256-Summe `5728b563911bd64bce0a0b81c74ea8ccb2190d1785bff34030fc6885a8273d3e` ist. Diese Summe ist auf `archive.apache.org` gelistet.
    * Die `guacamole-1.5.4.war`-Datei auf der primären Apache-Download-Seite (`downloads.apache.org`) hat eine andere SHA256-Summe: `b82589373835518ea50222695f96572181766725010551596847877966869534`.
    * **Wenn du später `Guacamole protocol violation`-Fehler in den `guacd`-Logs feststellst, deutet dies stark darauf hin, dass die von dir verwendete `guacamole.war`-Datei (`5728b...`) nicht vollständig mit dem von dir kompilierten `guacd` kompatibel ist (welches vermutlich aus dem Quellcode stammt, der zur primären `.war`-Datei mit der Summe `b8258...` gehört). Die Lösung wäre dann, die `.war`-Datei mit der Summe `b8258...` zu beschaffen und zu verwenden.**
    * Fürs Erste dokumentieren wir die Schritte mit der Datei, die du erfolgreich beziehen konntest.

2.  `guacamole.war` herunterladen (angenommen, du hast die Datei mit der Summe `5728b...` heruntergeladen und als `guacamole.war` gespeichert):
    ```bash
    # export GUAC_VERSION=1.5.4 # Falls du eine Variable verwenden möchtest
    # wget -O guacamole.war "LINK_ZU_DEINER_GEWÄHLTEN_WAR_DATEI" 
    # Stelle sicher, dass dies die Datei ist, die du verwenden möchtest.
    ```
3.  Unterverzeichnisse für Guacamole erstellen (für Erweiterungen und Bibliotheken, falls später verwendet):
    ```bash
    sudo mkdir /etc/guacamole/lib
    sudo mkdir /etc/guacamole/extensions 
    ```
    (Das `extensions`-Verzeichnis war bei unseren Tests leer, was für die Basis-Authentifizierung per Datei gut ist).
4.  `guacamole.war` in Tomcats `webapps`-Verzeichnis verschieben:
    ```bash
    # Angenommen, guacamole.war befindet sich im aktuellen Verzeichnis
    sudo mv guacamole.war /var/lib/tomcat9/webapps/
    sudo chown tomcat:tomcat /var/lib/tomcat9/webapps/guacamole.war # (Empfohlen)
    ```
5.  Umgebungsvariable `GUACAMOLE_HOME` für Tomcat setzen:
    Bearbeite die Datei `/etc/default/tomcat9`:
    ```bash
    sudo nano /etc/default/tomcat9
    ```
    Füge folgende Zeile am Ende der Datei hinzu:
    ```bash
    GUACAMOLE_HOME=/etc/guacamole
    ```
6.  Konfigurationsdatei für die Guacamole-Webanwendung erstellen (`/etc/guacamole/guacamole.properties`):
    ```bash
    sudo nano /etc/guacamole/guacamole.properties
    ```
    Inhalt der Datei:
    ```properties
    guacd-hostname: 127.0.0.1
    guacd-port: 4822
    auth-provider: org.apache.guacamole.auth.file.FileAuthenticationProvider
    ```
    (Die Zeile `auth-provider` hat uns bei der Fehlersuche geholfen; Guacamole könnte auch ohne diese Zeile funktionieren, wenn keine anderen Authentifizierungs-Erweiterungen vorhanden sind und `user-mapping.xml` existiert).
7.  Berechtigungen für das Guacamole-Konfigurationsverzeichnis anpassen:
    ```bash
    sudo chown -R root:tomcat /etc/guacamole
    sudo chmod -R g+rX,o-rwx /etc/guacamole
    ```
8.  Tomcat neu starten und Status überprüfen:
    ```bash
    sudo systemctl restart tomcat9
    sudo systemctl status tomcat9
    ```
    (Sicherstellen, dass der Status `active (running)` ist und die `catalina.out`-Logs keine schwerwiegenden Fehler beim Start der Guacamole-Webanwendung zeigen).

---

### Schritt 4: `user-mapping.xml` erstellen (auf PC1) - Die von dir gefundene funktionierende Lösung

1.  `user-mapping.xml`-Datei erstellen und bearbeiten:
    ```bash
    sudo nano /etc/guacamole/user-mapping.xml
    ```
2.  Inhalt der Datei (dies ist die Struktur, die bei dir funktioniert hat, um den `Unexpected element: 'connection'`-Fehler zu vermeiden, mit "inline" Definition der Verbindung innerhalb von `<authorize>`):
    ```xml
    <user-mapping>
        <authorize username="DEIN_GUACAMOLE_BENUTZERNAME" password="DEIN_GUACAMOLE_PASSWORT">
            <connection name="REMnuxVM">
                <protocol>rdp</protocol>
                <param name="hostname">192.168.0.170</param> <param name="port">3392</param>             <param name="username">remnux</param>       <param name="password">malware</param>      <param name="ignore-cert">true</param>
                <param name="security">any</param>
            </connection>
        </authorize>
    </user-mapping>
    ```
    **Hinweis:** Ersetze `DEIN_GUACAMOLE_BENUTZERNAME` und `DEIN_GUACAMOLE_PASSWORT` durch die gewünschten Daten (z.B. `md` und `md`). Passe die Verbindungsdetails (`hostname`, `port`, `username`, `password` für die VM) entsprechend an.

3.  Berechtigungen für `user-mapping.xml` anpassen:
    ```bash
    sudo chmod 640 /etc/guacamole/user-mapping.xml
    sudo chown root:tomcat /etc/guacamole/user-mapping.xml
    ```
4.  Tomcat neu starten (damit die neue `user-mapping.xml` berücksichtigt wird):
    ```bash
    sudo systemctl restart tomcat9
    ```
    (Überprüfe den Tomcat-Status und `catalina.out`, um sicherzustellen, dass keine XML-Parsing-Fehler auftreten).

---

### Schritt 5: Firewall auf PC2 (Windows) konfigurieren

* Denke daran, eine **eingehende Regel (Inbound Rule)** in der Windows-Firewall auf **PC2** zu erstellen.
* Diese Regel muss eingehende **TCP**-Verbindungen auf Port `3392` (dem VRDE-Port deiner REMnux VM) erlauben.
* Idealerweise sollte die Quelle der erlaubten Verbindungen auf die IP-Adresse von PC1 (`192.168.0.139` in unserem Beispiel) beschränkt werden, um die Sicherheit zu erhöhen.

---

### Schritt 6: Zugriff auf Guacamole und die VM

1.  Öffne einen Webbrowser auf einem beliebigen Computer in deinem Netzwerk.
2.  Gehe zur Adresse: `http://IHRE_IP_PC1:8080/guacamole/` (ersetze `IHRE_IP_PC1` durch die tatsächliche IP-Adresse von PC1, z.B. `192.168.0.139`).
3.  Melde dich mit `DEIN_GUACAMOLE_BENUTZERNAME` und `DEIN_GUACAMOLE_PASSWORT` an, die du in `user-mapping.xml` festgelegt hast.
4.  Du solltest die Verbindung namens `REMnuxVM` (oder den von dir gewählten Namen) in der Liste sehen. Klicke darauf.
5.  Wenn alles korrekt konfiguriert ist (einschließlich der Firewall auf PC2 und der Kompatibilität der Guacamole-Versionen), sollte die Verbindung zur REMnux VM hergestellt werden.


