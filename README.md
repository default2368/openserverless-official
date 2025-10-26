Guida Completa a OpenServerless
Indice

Introduzione
Architettura del Sistema
Componenti e Strumenti
Installazione e Setup
Gestione Utenti
Operazioni Quotidiane
Troubleshooting


1. Introduzione {#introduzione}
OpenServerless è una piattaforma open source per il serverless computing che offre un'alternativa flessibile e portabile rispetto alle soluzioni proprietarie dei cloud provider (AWS Lambda, Azure Functions, Google Cloud Functions).
Caratteristiche Principali

Open Source Completo: Codice completamente aperto e modificabile
Portabilità: Evita il vendor lock-in, deploy ovunque
Basato su Kubernetes: Sfrutta l'ecosistema cloud-native
Self-Hosted: Controllo completo su dati e infrastruttura
Compatibilità Multi-Runtime: Supporta diversi linguaggi di programmazione


2. Architettura del Sistema {#architettura}
Stack Tecnologico
┌─────────────────────────────────────────┐
│         CLI Tools (ops, kubectl)        │
│         Interfaccia Utente              │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│         Docker Desktop                  │
│    (Container Runtime + Kubernetes)     │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│      Kubernetes Cluster                 │
│   • Orchestrazione Container            │
│   • Gestione Risorse                    │
│   • Networking e Storage                │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│      OpenServerless Platform            │
│   • Controller                          │
│   • API Gateway                         │
│   • Function Runtime                    │
│   • Database (CouchDB)                  │
│   • Optional: Redis, MongoDB, MinIO     │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│      Servizi Esposti                    │
│   • REST API                            │
│   • Web UI                              │
│   • Autenticazione                      │
└─────────────────────────────────────────┘
Flusso di Dipendenze
Senza Docker Desktop:

❌ Kubernetes non disponibile
❌ OpenServerless non può partire
❌ API REST non accessibili
❌ Web UI non funzionante
✅ Solo CLI tools disponibili (comandi locali)

Con Docker Desktop Attivo:

✅ Kubernetes operativo
✅ OpenServerless deployato
✅ API REST disponibili
✅ Web UI accessibile
✅ Autenticazione funzionante


3. Componenti e Strumenti {#componenti}
kubectl
Cosa fa:

Client a riga di comando per Kubernetes
Gestisce risorse native K8s (pods, services, secrets, namespaces)
Opera a livello infrastrutturale basso

Comandi Essenziali:
bash# Verifica stato cluster
kubectl get nodes

# Lista namespace
kubectl get namespaces

# Vedi tutti i pod
kubectl get pods --all-namespaces

# Vedi risorse in un namespace specifico
kubectl get all -n <namespace>

# Vedi i secrets
kubectl get secrets -n <namespace>

# Vedi i logs di un pod
kubectl logs <pod-name> -n <namespace>

# Edita una risorsa
kubectl edit secret <secret-name> -n <namespace>

# Configurazione kubectl
kubectl config view
kubectl config current-context
kubectl config get-contexts
ops CLI
Cosa fa:

CLI specifica per OpenServerless
Semplifica operazioni complesse
Opera a livello applicativo alto
Usa kubectl dietro le quinte

Comandi Essenziali:
bash# Info sulla CLI
ops -version
ops -info
ops -help
ops -tasks

# Setup e configurazione
ops setup prereq          # Valida prerequisiti
ops setup cluster         # Deploy su cluster esistente
ops setup devcluster      # Crea dev cluster locale
ops setup mini            # Deploy versione slim
ops setup status          # Verifica stato
ops setup uninstall       # Disinstalla

# Configurazione
ops -config               # Gestione configurazione

# Autenticazione
ops -login <apihost> <username>

# Gestione utenti (admin)
ops admin adduser <username> <email> <password> [opzioni]
ops admin deleteuser <username>
ops admin listuser [<username>]
ops admin usage

# Gestione actions (serverless functions)
ops action list
ops action create <name> <file>
ops action update <name> <file>
ops action delete <name>
ops action invoke <name>

# Package e trigger
ops package list
ops trigger list
ops rule list

# Logs e debugging
ops activations list
ops logs <activation-id>
ops result <activation-id>
Relazione tra kubectl e ops
ops admin adduser mario email@test.com Pass123 --all
           ↓
    (internamente esegue)
           ↓
kubectl create namespace mario
kubectl create secret generic mario-auth ...
kubectl apply -f mario-redis-deployment.yaml
kubectl apply -f mario-mongodb-deployment.yaml
kubectl apply -f mario-storage-pvc.yaml
...
Quando usare cosa:

ops: Operazioni standard e semplificate su OpenServerless
kubectl: Debug, modifiche manuali, operazioni non supportate da ops (es. cambio password)


4. Installazione e Setup {#installazione}
Prerequisiti
Sistema:

Docker Desktop con almeno 6GB RAM
20GB+ spazio disco disponibile

Verifica installazioni:
bash# Verifica Docker
docker --version
docker ps

# Verifica Kubernetes
kubectl version --client
kubectl get nodes

# Verifica ops
ops -version
ops -info
Setup OpenServerless
Opzione 1: Mini Setup (Locale Leggero)
bash# Deploy versione slim
ops setup mini

# Accesso: http://devel.miniops.me
Opzione 2: Cluster Setup (Docker Desktop)
bash# 1. Avvia Docker Desktop e aspetta che Kubernetes sia pronto
kubectl get nodes
# Deve mostrare: docker-desktop   Ready

# 2. Valida prerequisiti
ops setup prereq

# 3. Deploy OpenServerless
ops setup cluster

# 4. Verifica installazione
ops setup status

# 5. Verifica pod attivi
kubectl get pods --all-namespaces
Opzione 3: Dev Cluster
bash# Crea e configura un dev cluster locale
ops setup devcluster
Verifica Installazione
bash# Vedi namespace creati
kubectl get namespaces
# Dovrebbe mostrare: nuvolaris o simile

# Vedi pod OpenServerless
kubectl get pods -n nuvolaris

# Verifica servizi
kubectl get services -n nuvolaris

# Test configurazione ops
ops -config

5. Gestione Utenti {#gestione-utenti}
Concetto Base
In OpenServerless: Utenti = Namespace

Ogni utente ha il proprio namespace Kubernetes isolato
I servizi dell'utente girano nel suo namespace
Isolamento completo tra utenti

Creare un Utente
bash# Sintassi base
ops admin adduser <username> <email> <password>

# Con tutti i servizi
ops admin adduser mario mario@example.com Pass123! --all --storagequota=auto

# Con servizi specifici
ops admin adduser luigi luigi@example.com Pass456! --redis --mongodb --minio

# Con quota storage specifica
ops admin adduser peach peach@example.com Pass789! --all --storagequota=10G
Opzioni servizi disponibili:

--all: Abilita tutti i servizi
--redis: Database key-value in-memory
--mongodb: Database NoSQL document-oriented
--minio: Object storage compatibile S3
--postgres: Database relazionale
--milvus: Vector database per AI/ML
--storagequota=<size>: Quota storage (es. 10G) o auto

Listare Utenti
bash# Lista tutti gli utenti
ops admin listuser

# Dettagli utente specifico
ops admin listuser mario

# Con kubectl
kubectl get namespaces
Eliminare un Utente
bash# Attenzione: elimina tutti i dati dell'utente!
ops admin deleteuser mario

# Verifica eliminazione
kubectl get namespace mario
# Dovrebbe dare errore: "not found"
Modificare Password Utente
Problema: ops non ha comando diretto per cambiare password.
Soluzione 1: Tramite kubectl (Consigliata)
bash# 1. Trova il secret dell'utente
kubectl get secrets -n mario

# 2. Visualizza i secrets
kubectl get secrets -n mario -o yaml

# 3. Identifica il secret con le credenziali (es. "mario-auth")
kubectl get secret mario-auth -n mario -o yaml

# 4. Codifica la nuova password in base64
echo -n "NuovaPassword123!" | base64
# Output: TnVvdmFQYXNzd29yZDEyMyE=

# 5. Edita il secret
kubectl edit secret mario-auth -n mario
# Sostituisci il campo password con il nuovo valore base64

# 6. Salva e esci
Soluzione 2: Ricreare utente (Perdita Dati!)
bash# Salva configurazioni importanti prima!

# Elimina utente
ops admin deleteuser mario

# Ricrea con nuova password
ops admin adduser mario mario@example.com NuovaPass! --all --storagequota=auto
Monitorare Risorse Utente
bash# Vedi utilizzo storage
ops admin usage

# Debug dettagliato
ops admin usage --debug

# Vedi tutte le risorse di un utente
kubectl get all -n mario

# Vedi pod specifici
kubectl get pods -n mario

# Logs di un servizio utente
kubectl logs <pod-name> -n mario

6. Operazioni Quotidiane {#operazioni}
Login e Autenticazione
bash# Login standard
ops -login http://miniops.me mario
# Inserisci password quando richiesta

# Login con URL custom
ops -login https://mio-openserverless.com mario

# Con variabili d'ambiente (per script)
export OPS_APIHOST=http://miniops.me
export OPS_USER=mario
export OPS_PASSWORD=Pass123!
ops -login
Gestione Actions (Funzioni Serverless)
Creare una Action
bash# Action JavaScript inline
ops action create hello <(echo 'function main() { return {body: "Hello World!"}; }') --kind nodejs:default

# Action da file
echo 'function main(params) { 
  return {greeting: "Hello " + params.name}; 
}' > hello.js

ops action create hello hello.js --kind nodejs:default

# Action Python
echo 'def main(args):
    name = args.get("name", "stranger")
    return {"greeting": f"Hello {name}"}
' > hello.py

ops action create hello-py hello.py --kind python:default
Invocare una Action
bash# Invocazione semplice
ops invoke hello

# Con parametri
ops invoke hello name=Mario

# Invocazione bloccante (aspetta risultato)
ops invoke hello --result

# Invocazione asincrona
ops invoke hello --async
Gestire Actions
bash# Lista tutte le actions
ops action list

# Dettagli action
ops action get hello

# Aggiorna action
ops action update hello hello-v2.js

# Elimina action
ops action delete hello

# Ottieni URL pubblico
ops url hello
Logs e Debugging
bash# Lista attivazioni recenti
ops activations list

# Logs di una specifica attivazione
ops logs <activation-id>

# Risultato di un'attivazione
ops result <activation-id>

# Logs in tempo reale (se disponibile)
ops activations poll
Package (Raggruppamento Actions)
bash# Crea package
ops package create mypackage

# Lista package
ops package list

# Crea action in un package
ops action create mypackage/hello hello.js

# Invoca action in package
ops invoke mypackage/hello

# Elimina package
ops package delete mypackage
Trigger e Rule
bash# Crea trigger
ops trigger create mytrigger

# Crea rule (collega trigger ad action)
ops rule create myrule mytrigger hello

# Lista trigger e rule
ops trigger list
ops rule list

# Fire trigger
ops trigger fire mytrigger name=Mario

# Elimina
ops rule delete myrule
ops trigger delete mytrigger

7. Troubleshooting {#troubleshooting}
Problemi Comuni
1. Connection Refused
Sintomo:
error: dial tcp 127.0.0.1:6443: connect: connection refused
Causa: Docker Desktop / Kubernetes non avviato
Soluzione:
bash# Avvia Docker Desktop
# Aspetta che l'icona diventi verde
# Verifica Kubernetes
kubectl get nodes
2. OpenServerless Non Risponde
Sintomo:
error: Post "http://miniops.me/api/v1/...": connection refused
Causa: OpenServerless non deployato o non attivo
Soluzione:
bash# Verifica pod OpenServerless
kubectl get pods --all-namespaces

# Se non ci sono pod di OpenServerless
ops setup cluster

# Verifica status
ops setup status
3. Utente Non Può Accedere
Causa possibili:

Password errata
Utente non creato correttamente
Servizi utente non attivi

Soluzione:
bash# Verifica utente esiste
ops admin listuser mario

# Verifica namespace
kubectl get namespace mario

# Verifica pod utente
kubectl get pods -n mario

# Verifica secrets
kubectl get secrets -n mario

# Ricrea utente se necessario
ops admin deleteuser mario
ops admin adduser mario mario@example.com NewPass! --all
4. Action Non Si Avvia
Debug:
bash# Verifica action esiste
ops action list

# Vedi dettagli
ops action get <action-name>

# Verifica logs ultimi errori
ops activations list
ops logs <last-activation-id>

# Verifica pod nel namespace
kubectl get pods -n <username>
kubectl logs <pod-name> -n <username>
5. Storage Quota Superata
Verifica:
bash# Vedi utilizzo storage
ops admin usage

# Debug dettagliato
ops admin usage --debug
Soluzione:

Aumenta quota utente (ricrea utente con quota maggiore)
Pulisci dati vecchi
Compatta database: ops admin compact

Reset Completo
Se nulla funziona:
bash# 1. Reset ops
ops -reset

# 2. Disinstalla OpenServerless
ops setup uninstall

# 3. Ferma Docker Desktop
# 4. Cancella dati Docker (opzionale ma efficace)
#    Settings → Troubleshoot → Clean/Purge data

# 5. Riavvia Docker Desktop

# 6. Reinstalla
ops setup prereq
ops setup cluster
Comandi Diagnostici Utili
bash# Info sistema
ops -info
kubectl version
docker version

# Stato cluster
kubectl get nodes
kubectl get namespaces
kubectl get pods --all-namespaces
kubectl get services --all-namespaces

# Stato OpenServerless
ops setup status
ops -config

# Logs sistema
kubectl logs <pod-name> -n nuvolaris
kubectl describe pod <pod-name> -n nuvolaris

# Eventi recenti
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Risorse cluster
kubectl top nodes
kubectl top pods --all-namespaces

Riepilogo Comandi Rapidi
Setup Iniziale
bashops setup prereq          # Valida prerequisiti
ops setup cluster         # Installa OpenServerless
ops setup status          # Verifica installazione
Gestione Utenti
bashops admin adduser <user> <email> <pass> --all
ops admin listuser
ops admin deleteuser <user>
Login e Uso
bashops -login http://miniops.me <username>
ops action create <name> <file>
ops invoke <name>
ops action list
Debug
bashkubectl get pods --all-namespaces
kubectl logs <pod> -n <namespace>
ops activations list
ops logs <activation-id>

Risorse

Documentazione ufficiale: https://openserverless.apache.org
Repository GitHub: https://github.com/apache/openserverless
Task Repository: https://github.com/apache/openserverless-task
Supporto Community: GitHub Issues


Guida creata in base all'esperienza pratica con OpenServerless 0.1.0
