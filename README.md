# DevOps Project – Freelancing Platform Project

## Description

Αυτό το repository περιέχει playbooks για να κάνουμε deploy την spring boot εφαρμογή που φτιάξαμε στο μάθημα `Κατανεμημένα Συστήματα`.

Περιέχει 3 διαφορετικούς φακέλους, όπου ο καθένας περιέχει τα αρχεία που χρειάζονται για να γινει deploy η εφαρμογή με τους 3 ακόλουθους τρόπους:

- Deploy σε 2 VM (Ένα για springboot application και ένα για την postgres βάση δεδομένων)
- Deploy σε 1 VM με χρήση docker (δύο containers, ένα για spring και ένα για postgres)
- Deploy σε 1 VM με microk8s (δύο deployments, ένα για spring boot και ένα για postgres)

Επίσης έχουμε ένα φάκελο για το CI/CD (jenkins) όπου περιέχει ένα αρχείο `Jenkinsfile` όπου εκτελέιτε σε έναν jenkins server που έχουμε σε ένα άλλο VM.

## Playbooks

### VM
- Δύο VM
- `postgres.yaml`: εγκατάσταση PostgreSQL, δημιουργία βάσης/χρήστη, ρύθμιση service
- `spring.yaml`: έλεγχος Java/Maven, clone του κώδικα, build, setup Spring service, Nginx

---

### Docker
- Ένα VM με `docker`, `docker-compose` και 2 containers:
  - `spring` (eclipse-temurin:21)
  - `postgres` (postgres:16)

- Χρήση Ansible για copy και εκτέλεση `docker-compose.yml`
- Volume για να μην διαγράφεται η δεδομένων στη βάση

---

### Kubernetes (Microk8s)

- Ένα VM με Microk8s
- Deployments:
  - `spring-deployment`
  - `postgres-deployment`
- Services: ClusterIP
- Ingress controller για external access (SSL-enabled)
- Persistent Volumes για Postgres

Σε αυτό το deploy φτιάξαμε dns:
https://colabworks.ip-ddns.com

---

### CI/CD (Jenkins)

- Job: Clone DS-Project με webhook σε νέο push (Clone Spring)

**Pipeline**

1. Stage 1: Καλούμε το job ansible-playbooks, το οποίο κάνει clone το repository με τα playbooks.
2. Stage 2: Τρέξιμο test σε H2 database.
3. Stage 3: Build image και push στο github.
4. Stage 4: Update Kubernetes cluster με νέο image

---

## Demo Videos

[Google Drive Folder](https://drive.google.com/drive/folders/1PNUpmBhnm171zlQU_d59r0eacApiBOww)

Περιλαμβάνει:
- VM deployment
- Docker deployment
- Kubernetes deployment
- Jenkins CI/CD Pipeline
