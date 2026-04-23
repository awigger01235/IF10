# Instalation der benötigten Software
## Windows-Subsystem für Linux (WSL) mit Ubuntu
```
wsl.exe --install --no-distribution
```
```
wsl --install ubuntu
```

## Instalation von Docker
```
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```
sudo systemctl status docker
```

# n8n-Container starten
```
# Erstellt einen dauerhaften Speicher für n8n-Daten
sudo docker volume create n8n_data

# Hiermit wird ein Docker Container gestartet, in welchem n8n läuft
sudo docker run -it --rm -d \
 --name n8n \
 -p 5678:5678 \
 -v n8n_data:/home/node/.n8n \
 docker.n8n.io/n8nio/n8n \
 start --tunnel
```

# n8n einrichten
- **Zugriff:** Öffne n8n in deinem Browser unter `http://localhost:5678`.
- **Anmeldung:** Nutze idealerweise deine **IServ-E-Mail-Adresse**.
- **Personalisierung:** Die Angaben beim "Customizing" sind optional – wähle einfach beliebige Optionen.
- **Lizenz:** Klicke bei der entsprechenden Aufforderung auf **"Send me a free license key"**.
- **Aktivierung:**
    1. Klicke unten links auf das **Zahnrad-Symbol** (Einstellungen).
    2. Navigiere zum Bereich **"Usage and Plan"**.
    3. Gib deinen erhaltenen Key unter **"Enter activation key"** ein.

# Ersten Workflow erstellen
Klicke auf der Startseite auf **Start from scratch**, um einen neuen Workflow zu eröffnen. Folge dann diesen Schritten:
1. Füge den Trigger **On chat message** hinzu
2. Füge an diesen den Ollama Node **Message a model** (Ist in der Kategorie **Ollama**) an
3. In dem Ollama-Node müssen als erstes die Credentials gesetzt werden (**Set up credentials**)
     - **Base URL:** http://10.255.41.159:11434 
4. Nun kann unter Model das Large-Language-Model **gemma4** ausgewählt werden
5. Wenn du eine Nachricht in den Chat gesendet hast (der Chat öffnet sich, wenn du über das **When chat message received** hoverst und auf **Open Chat** klickst) siehst du im Ollama-Menü links den **Input** Hier gibt es einen Bereich **chatInput**, welchen du in das Content-Feld ziehen musst, damit an das LLM die eingegebene Nachricht übergeben wird
6. Nun kannst du im Chat mit dem LLM **"gemma4"** chatten 

# Zweiter Workflow
In diesem Abschnitt wirst du einen Workflow erstellen, welcher dir eine Nachricht auf deine IServ-Email sendet, wenn du an diesem Tag Informatik-Unterricht hast

1. **Hinzufügen des WebUntis-Nodes:**
    1. Klicke unten links auf das **Zahnrad-Symbol** (Einstellungen).
    2. Navigiere zum Bereich **"Community nodes"**.
    3. Unter **Install a community node** als Namen **n8n-nodes-webuntis** eingeben, den Haken bei den Risiken setzen und auf **Install** drücken
2. Füge den Trigger **Trigger manually** hinzu
3. Füge an diesen den Node **Get Current Date** aus **Date & Time** (Nach Node Date & Time suchen)
4. An diesen Node kann nun ein **Get timetable for timeframe** von **WebUntis** angehangen werden
    - Credentials:
        - Username: WebUntis-Nutzername
        - Password: WebUntis-Passwort
    - Date: \[Die Ausgabe des **Get Current Time** Nodes]
    - School Name: ahg-ahaus
    - Base URL: https://ahg-ahaus.webuntis.com
6. Um nun aus den ganzen Daten nurnoch die einzelnen Stunden herauszunehmen, kann der Node **Split Out** genutzt werden
    - Um nur die Unterrichtsstunden einzeln weiterzugeben kann hier in **Fields To Split Out** der Bereich **lessons** hineingezogen werden (nicht **lessons\[...]**)
8. Die einzelnen Unterrichtsstunden lassen sich nun mit dem Node **Filter** filtern sodass bspw. nur Unterrichtsstunden ausgegeben werden, welche Informatik sind. Hierzu stellt man den Filter bspw. so ein, dass **subject** gleich **IF10** (Fachkürzel muss der Liste entnommen werden) ist.
    - Möchte man nach demm Datum filtern, muss man zudem noch nach dem **Get Current Date** Node noch einen **Format Current Time** Node anhängen, welcher das Datum, welches man mit dem **Get Current Date** Node erhält in folgendes Format umformt: **dd.MM.yyyy** und im **Filter** Node **Convert types where required aktiviert sein
    - **Wichtig zu beachten:** Wenn man dies macht, müssen zudem in dem **Get timetable for timeframe** Node das Datum so eingestellt werden, dass es immernoch das originale Datum nutzt
9. Nun kann man, an den Filter widerum den Ollama Node **Message a model** anhängen um sich eine Nachricht formulieren zu lassen
       - Hierzu werden die bereits erstellten Credentials genutzt und es muss nur ein Prompt formuliert sowie das Model auf gemma4 gestellt werden  
10. Die von Ollama generierte Nachricht kann dann mit dem **Send an Email** (In der Kategorie **Send Email**) an seine IServ-Email schicken lassen
    - Als Credentials:
        - User: IServ-Nutzername
        - Password: IServ-Passwort
        - Host: ah-ahg.de
        - Der Rest kann so bleiben, wie er ist
    - From Email: \[irgendetwas]@ah-ahg.de
    - To Email: Deine IServ Email
    - Subject: Betreff der Email
    - Email Format: Text
    - Im Text feld kann dann die von der KI generierte Nachricht hineingezogen werden
