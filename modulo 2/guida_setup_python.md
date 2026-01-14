# Guida Setup Ambiente - Modulo 2

**NOTA**: Questa guida descrive l'installazione diretta. Per un setup più pulito e isolato, **consigliamo di usare Docker**. Vedi `guida_setup_docker.md` per la guida Docker (raccomandato).

Questa guida ti accompagnerà nell'installazione diretta di tutto il necessario per seguire il corso pratico.

---

## 1. Installazione Python

### Verifica Python Installato

Apri un terminale (Linux/Mac) o PowerShell (Windows) e verifica:

```bash
python --version
# oppure
python3 --version
```

**Richiesto**: Python 3.10 o superiore

### Se Python Non è Installato

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

#### Mac
```bash
# Con Homebrew
brew install python3

# Oppure scarica da python.org
```

#### Windows
1. Scarica da https://www.python.org/downloads/
2. Durante installazione, **seleziona "Add Python to PATH"**
3. Verifica con `python --version`

### Verifica pip

```bash
pip --version
# oppure
pip3 --version
```

Se non installato:
```bash
python -m ensurepip --upgrade
```

---

## 2. Installazione Jupyter Notebook

### Installazione Base

```bash
pip install jupyter notebook
```

### Verifica Installazione

```bash
jupyter --version
```

### Avvio Jupyter

```bash
jupyter notebook
```

Si aprirà il browser con l'interfaccia Jupyter. Per fermare: `Ctrl+C` nel terminale.

**Alternativa**: JupyterLab (interfaccia più moderna)
```bash
pip install jupyterlab
jupyter lab
```

---

## 3. Installazione Ollama (LLM Locali)

Ollama permette di eseguire LLM localmente senza configurazione complessa.

### Linux

```bash
curl -fsSL https://ollama.ai/install.sh | sh
```

### Mac

```bash
# Con Homebrew
brew install ollama

# Oppure scarica da https://ollama.ai/download
```

### Windows

1. Scarica installer da https://ollama.ai/download
2. Esegui installer
3. Ollama si avvierà automaticamente

### Verifica Installazione

```bash
ollama --version
```

### Download Modello LLM

Per iniziare, scarichiamo Llama 3 8B (bilanciato qualità/risorse):

```bash
ollama pull llama3.2:3b
```

**Nota**: Il download richiede ~4.7GB. Per modelli più piccoli:
- `llama3.2:3b-instruct-q4_0` (quantizzato, ~4.7GB)
- `mistral:7b` (alternativa, ~4.1GB)
- `phi3:mini` (molto leggero, ~2.3GB, per hardware limitato)

### Test Ollama

```bash
ollama run llama3.2:3b "Ciao, come stai?"
```

Dovresti vedere una risposta generata. Premi `Ctrl+D` per uscire.

### Avvio Server Ollama

Ollama deve essere in esecuzione per essere usato da Python. Di solito parte automaticamente, ma se necessario:

```bash
ollama serve
```

Lascia questo terminale aperto. Ollama sarà disponibile su `http://localhost:11434`

---

## 4. Installazione Dipendenze Python

### Creazione Ambiente Virtuale (Consigliato)

Creare un ambiente virtuale isola le dipendenze del progetto:

```bash
# Crea ambiente virtuale
python -m venv venv_chatbot

# Attiva ambiente virtuale
# Linux/Mac:
source venv_chatbot/bin/activate
# Windows:
venv_chatbot\Scripts\activate
```

Quando attivo, vedrai `(venv_chatbot)` nel prompt.

### Installazione Pacchetti

```bash
# LangChain versione 1.0+ (rilasciata ottobre 2024)
pip install "langchain>=1.0.0"
pip install "langchain-community>=0.3.0"
pip install "langchain-core>=1.0.0"
pip install "langchain-ollama>=0.1.0"  # API moderna per Ollama (richiesto per LangChain 1.0+)
pip install jupyter
pip install python-dotenv  # Per gestione variabili ambiente
```

**Oppure crea file `requirements.txt`**:

```txt
langchain>=1.0.0
langchain-community>=0.3.0
langchain-core>=1.0.0
langchain-ollama>=0.1.0
jupyter>=1.0.0
python-dotenv>=1.0.0
```

E installa con:
```bash
pip install -r requirements.txt
```

### Verifica Installazione

Apri Python e testa:

```python
import langchain
print(langchain.__version__)

from langchain_ollama import OllamaLLM
print("LangChain installato correttamente!")
```

---

## 5. Configurazione Progetto

### Struttura Cartelle

Crea una cartella per il progetto:

```bash
mkdir corso_chatbot
cd corso_chatbot
```

### File di Configurazione

Crea file `.env` (opzionale, per configurazioni):

```bash
# .env
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=llama3.2:3b
```

### Repository GitHub (Opzionale)

Se vuoi versionare il codice:

```bash
git init
git add .
git commit -m "Initial setup"
```

---

## 6. Test Completo Setup

Crea file `test_setup.py`:

```python
#!/usr/bin/env python3
"""Test setup ambiente"""

print("1. Test import LangChain...")
from langchain_ollama import OllamaLLM
print("   ✅ LangChain importato")

print("2. Test connessione Ollama...")
try:
    llm = OllamaLLM(model="llama3.2:3b")
    response = llm.invoke("Rispondi solo: OK")
    print(f"   ✅ Ollama funziona: {response[:50]}")
except Exception as e:
    print(f"   ❌ Errore Ollama: {e}")
    print("   Verifica che Ollama sia in esecuzione: ollama serve")

print("3. Test Jupyter...")
try:
    import IPython
    print("   ✅ Jupyter/IPython disponibile")
except:
    print("   ⚠️  Jupyter non importabile (non critico)")

print("\n✅ Setup completato! Sei pronto per il corso.")
```

Esegui:
```bash
python test_setup.py
```

---

## 7. Troubleshooting Comune

### Problema: "python: command not found"

**Soluzione**:
- Linux/Mac: Usa `python3` invece di `python`
- Windows: Rinstalla Python selezionando "Add to PATH"
- Verifica PATH: `echo $PATH` (Linux/Mac) o `echo %PATH%` (Windows)

### Problema: "pip: command not found"

**Soluzione**:
```bash
python -m ensurepip --upgrade
python -m pip install --upgrade pip
```

### Problema: Ollama non risponde

**Soluzione**:
1. Verifica che Ollama sia in esecuzione: `ollama list`
2. Riavvia: `ollama serve` (in terminale separato)
3. Verifica porta: `curl http://localhost:11434/api/tags`
4. Su Windows: Verifica servizio Ollama in Task Manager

### Problema: Modello non trovato

**Soluzione**:
```bash
# Lista modelli installati
ollama list

# Se manca, scarica
ollama pull llama3.2:3b
```

### Problema: Performance LLM molto lente

**Soluzione**:
- Usa modello più piccolo (phi3:mini)
- Verifica RAM disponibile (serve almeno 8GB)
- Se hai GPU NVIDIA, Ollama la userà automaticamente
- Chiudi altre applicazioni pesanti

### Problema: Errori import LangChain

**Soluzione**:
```bash
# Reinstalla con versioni aggiornate (1.0+)
pip uninstall langchain langchain-community langchain-core
pip install "langchain>=1.0.0" "langchain-community>=0.3.0" "langchain-core>=1.0.0"

# Verifica versione Python
python --version  # Deve essere 3.10+
```

### Problema: Jupyter non si apre nel browser

**Soluzione**:
```bash
# Avvia con URL esplicito
jupyter notebook --ip=127.0.0.1 --port=8888

# Oppure copia URL dal terminale
```

---

## 8. Alternative e Opzioni Avanzate

### Alternative a Ollama

**LM Studio** (Windows/Mac con GUI):
- Download: https://lmstudio.ai/
- Interfaccia grafica per gestione modelli
- API compatibile con Ollama

**vLLM** (Server production):
- Per deployment più avanzato
- Migliori performance

### Editor Alternativi

- **VS Code**: Con estensione Jupyter
- **PyCharm**: IDE completo Python
- **Google Colab**: Jupyter cloud (ma non per LLM locali)

### GPU Acceleration

Se hai GPU NVIDIA:
1. Installa CUDA: https://developer.nvidia.com/cuda-downloads
2. Ollama userà automaticamente GPU se disponibile
3. Verifica: `nvidia-smi` durante esecuzione

---

## 9. Checklist Pre-Corso

Prima di iniziare il Modulo 2, verifica:

- [ ] Python 3.10+ installato e funzionante
- [ ] pip installato e aggiornato
- [ ] Jupyter installato e avviabile
- [ ] Ollama installato e funzionante
- [ ] Modello LLM scaricato (llama3.2:3b o alternativo)
- [ ] LangChain installato e importabile
- [ ] Test setup eseguito con successo
- [ ] Cartella progetto creata
- [ ] Notebook di test funzionante

---

## 10. Supporto

Se incontri problemi:

1. **Rivedi questa guida**: Molti problemi sono comuni
2. **Verifica versioni**: Python 3.10+, LangChain 1.0+ (rilasciato ottobre 2024)
3. **Controlla log**: Ollama mostra errori nel terminale
4. **Test incrementale**: Testa ogni componente separatamente
5. **Chiedi aiuto**: Durante il corso, il formatore aiuterà con setup

---

**Buon lavoro! 🚀**

