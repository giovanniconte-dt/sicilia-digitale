# Guida Setup con Docker

Questa guida ti accompagna nell'installazione di Ollama e Open WebUI usando Docker, mantenendo l'ambiente pulito e isolato.

---

## 1. Prerequisiti: Docker e Docker Compose

### Verifica Docker Installato

Apri un terminale e verifica:

```bash
docker --version
docker-compose --version
```

**Richiesto**: Docker 20.10+ e Docker Compose 2.0+

### Se Docker Non è Installato

#### Linux (Ubuntu/Debian)
```bash
# Installa Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Aggiungi utente al gruppo docker (per evitare sudo)
sudo usermod -aG docker $USER

# Riloggia o esegui:
newgrp docker
```

#### Mac
1. Scarica Docker Desktop: https://www.docker.com/products/docker-desktop
2. Installa e avvia Docker Desktop
3. Docker Compose è incluso

#### Windows
1. Scarica Docker Desktop: https://www.docker.com/products/docker-desktop
2. Installa e avvia Docker Desktop
3. Docker Compose è incluso
4. Assicurati che WSL2 sia attivo

### Verifica Installazione

```bash
docker run hello-world
```

Dovresti vedere un messaggio di benvenuto.

---

## 2. Setup Ollama e Open WebUI con Docker Compose

### 2.1 File Docker Compose

Crea il file `docker-compose.yml` nella cartella del progetto (già fornito).

Il file contiene:
- **Ollama**: Server LLM locale, porta 11434
- **Open WebUI**: Interfaccia web per gestire modelli e chat, porta 3000

### 2.2 Avvio Servizi

Nella cartella con `docker-compose.yml`:

```bash
# Avvia servizi in background
docker-compose up -d

# Verifica che siano in esecuzione
docker-compose ps
```

Dovresti vedere:
```
NAME                   STATUS
ollama_chatbot         Up
open_webui_chatbot     Up
```

### 2.3 Accesso Interfacce

- **Ollama API**: http://localhost:11434
- **Open WebUI**: http://localhost:3000

Apri il browser su http://localhost:3000 per accedere a Open WebUI.

---

## 3. Download Modelli LLM

### Opzione 1: Via Open WebUI (Raccomandato)

1. Apri http://localhost:3000
2. Vai su "Settings" → "Models"
3. Clicca "Pull a model from Ollama.com"
4. Inserisci nome modello (es. `llama3.2:3b`)
5. Clicca "Pull Model"
6. Attendi download (mostra progresso)

**Modelli consigliati**:
- `llama3.2:3b` - Bilanciato qualità/risorse (~4.7GB)
- `llama3.2:1b` - Minimal
- `mistral:7b` - Alternativa efficiente (~4.1GB)
- `phi3:mini` - Molto leggero, per hardware limitato (~2.3GB)

### Opzione 2: Via Terminale Docker

```bash
# Accedi al container Ollama
docker exec -it ollama_chatbot ollama pull llama3.2:3b

# Verifica modelli installati
docker exec -it ollama_chatbot ollama list
```

### Opzione 3: Via API Ollama

```bash
curl http://localhost:11434/api/pull -d '{
  "name": "llama3.2:3b"
}'
```

---

## 4. Verifica Setup

### 4.1 Test Ollama API

```bash
# Lista modelli
curl http://localhost:11434/api/tags

# Test generazione
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2:3b",
  "prompt": "Ciao! Rispondi brevemente.",
  "stream": false
}'
```

### 4.2 Test Open WebUI

1. Apri http://localhost:3000
2. Crea account (primo accesso)
3. Seleziona modello (es. llama3.2:3b)
4. Prova una chat semplice

### 4.3 Test da Python

Crea file `test_docker_setup.py`:

```python
from langchain_ollama import OllamaLLM

# Connessione a Ollama in Docker (API moderna LangChain 1.0+)
llm = OllamaLLM(
    model="llama3.2:3b",
    base_url="http://localhost:11434"
)

# Test
response = llm.invoke("Ciao! Rispondi brevemente.")
print(response)
```

Esegui:
```bash
python test_docker_setup.py
```

---

## 5. Gestione Container

### Comandi Utili

```bash
# Avvia servizi
docker-compose up -d

# Ferma servizi
docker-compose down

# Ferma e rimuovi volumi (ATTENZIONE: cancella modelli!)
docker-compose down -v

# Visualizza log
docker-compose logs -f ollama
docker-compose logs -f open-webui

# Riavvia un servizio
docker-compose restart ollama

# Verifica status
docker-compose ps

# Verifica uso risorse
docker stats
```

### Backup Modelli

I modelli sono salvati nel volume Docker. Per backup:

```bash
# Backup volume
docker run --rm -v ollama_chatbot_ollama_data:/data -v $(pwd):/backup \
  alpine tar czf /backup/ollama_backup.tar.gz /data

# Restore volume
docker run --rm -v ollama_chatbot_ollama_data:/data -v $(pwd):/backup \
  alpine tar xzf /backup/ollama_backup.tar.gz -C /
```

---

## 6. Configurazione Avanzata

### 6.1 GPU Support (NVIDIA)

Per usare GPU NVIDIA, modifica `docker-compose.yml`:

```yaml
services:
  ollama:
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

**Prerequisiti**:
- NVIDIA GPU con driver installato
- nvidia-container-toolkit:
  ```bash
  # Ubuntu
  distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
  curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
  curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
  sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
  sudo systemctl restart docker
  ```

### 6.2 Cambiare Porte

Modifica `docker-compose.yml`:

```yaml
services:
  ollama:
    ports:
      - "11435:11434"  # Cambia porta esterna
  open-webui:
    ports:
      - "3001:8080"  # Cambia porta esterna
```

Aggiorna i notebook per usare la nuova porta:
```python
llm = OllamaLLM(model="llama3.2:3b", base_url="http://localhost:11435")
```

### 6.3 Persistenza Dati

I dati sono salvati in volumi Docker:
- `ollama_data`: Modelli e cache Ollama
- `open-webui_data`: Database e configurazione Open WebUI

I volumi persistono anche quando i container sono fermati.

Per vedere dove sono i volumi:
```bash
docker volume ls
docker volume inspect modulo2_ollama_data
```

---

## 7. Troubleshooting

### Problema: "Port already in use"

**Causa**: Porte 11434 o 3000 già in uso.

**Soluzione**:
1. Trova processo che usa la porta:
   ```bash
   # Linux/Mac
   lsof -i :11434
   lsof -i :3000
   
   # Windows
   netstat -ano | findstr :11434
   ```
2. Ferma processo o cambia porta in `docker-compose.yml`

### Problema: Container non si avvia

**Soluzione**:
```bash
# Visualizza log errori
docker-compose logs ollama
docker-compose logs open-webui

# Verifica spazio disco
df -h

# Riavvia
docker-compose down
docker-compose up -d
```

### Problema: "Cannot connect to Ollama" da Python

**Soluzione**:
1. Verifica che Ollama sia in esecuzione:
   ```bash
   docker-compose ps
   curl http://localhost:11434/api/tags
   ```
2. Verifica che stai usando porta corretta (default: 11434)
3. Su alcuni sistemi, prova `http://127.0.0.1:11434` invece di `localhost`

### Problema: Modello non trovato

**Soluzione**:
```bash
# Verifica modelli installati
docker exec -it ollama_chatbot ollama list

# Se manca, scarica
docker exec -it ollama_chatbot ollama pull llama3.2:3b
```

### Problema: Performance lente

**Soluzione**:
1. Verifica risorse disponibili:
   ```bash
   docker stats
   ```
2. Se hai GPU NVIDIA, abilita GPU support (vedi sezione 6.1)
3. Usa modello più piccolo (phi3:mini)
4. Aumenta RAM Docker (Docker Desktop → Settings → Resources)

### Problema: Out of Memory

**Soluzione**:
1. Riduci numero modelli scaricati
2. Aumenta RAM Docker (Docker Desktop → Settings → Resources)
3. Usa modelli quantizzati più piccoli:
   ```bash
   docker exec -it ollama_chatbot ollama pull llama3.2:3b-instruct-q4_0
   ```

---

## Checklist Pre-Corso

- [ ] Docker installato e funzionante
- [ ] Docker Compose installato
- [ ] Servizi Ollama e Open WebUI avviati (`docker-compose up -d`)
- [ ] Open WebUI accessibile su http://localhost:3000
- [ ] Modello LLM scaricato (llama3.2:3b o alternativo)
- [ ] Test connessione da Python funzionante
- [ ] Python 3.10+ installato (su macchina host)
- [ ] LangChain installato (`pip install langchain>=1.0.0 langchain-community>=0.3.0 langchain-core>=1.0.0 langchain-ollama>=0.1.0`)

---

## NOTE:

1. **Python sul Host**: I notebook Jupyter girano sul tuo PC, non in Docker. Docker serve solo per Ollama/Open WebUI.

2. **Porte**: Assicurati che porte 11434 e 3000 siano libere, o modifica `docker-compose.yml`.

3. **Persistenza**: I modelli sono salvati in volumi Docker. Se fai `docker-compose down -v`, i modelli vengono cancellati.

4. **Performance**: Se hai GPU NVIDIA, abilita GPU support per performance migliori.

5. **Network**: I container Docker sono accessibili da `localhost` sul host.

---

**Setup completato! 🚀**

