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
