# DevOps Project

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

---

# Οδηγίες Εγκατάστασης

### Προαπαιτούμενα

- Ένα ή δύο VM (ανάλογα με το deployment)
- Ubuntu 22.04/24.04
- Ansible εγκατεστημένο στο τοπικό μηχάνημα

```bash
sudo apt update
sudo apt install ansible -y
```

### Clone τα Repositories

```sh
git clone https://github.com/KaratziasK/DS-project.git
```

```sh
git clone https://github.com/KaratziasK/Devops.git
```


### Ρύθμιση SSH πρόσβασης

Για να μπορέσει το Ansible να επικοινωνήσει με τα VM, πρέπει να έχουμε SSH πρόσβαση.

Ρυθμίζουμε τα κλειδία (public και private) και φτίαχνουμε το `.ssh/config`, ετσι ώστε η πρόσβαση στο vm να είναι όσο το δυνατό πιο εύκολη.

Τα Host ονόματα στο αρχείο `.ssh/config` πρέπει να ταιριάζουν με τα ονόματα στο αρχείο `hosts.yaml`

---

### VM deployment

Σε αυτή την περίπτωση θα χρειαστούμε 2 VM. Το ένα για το application και το άλλο για την βάση δεδομένων.

- **Πρέπει το VM που θα χρησιμοποιήσουμε για το application να έχει ανοιχτό το port `80` για να επικοινωνεί με nginx**

- **Πρέπει το VM που θα χρησιμοποιήσουμε για την βάση δεδομένων να έχει ανοιχτό το port `5432` για να επικοινωνεί με το spring**

- **Θα χρειαστεί επίσης να γίνουν καποιες αλλαγές στα αρχεία μεταβλητών που περιέχουν `IP διευθύνσεις`, ώστε να μπουν οι σωστές**

**Εκτέλεση των playbooks**

Πρώτα τρέχουμε το playbook `postgres.yaml` και μετά το `spring.yaml`.

```sh
cd project
ansible-playbook playbooks-vm/postgres.yaml 
ansible-playbook playbooks-vm/spring.yaml -e "EMAIL_PASSWORD=email_password"
```

---

### Docker deployment

Σε αυτή την περίπτωση θα χρειαστούμε ένα VM στο οποίο θα εγκαταστήσουμε μέσω ansible το docker στο vm και θα φτιάξουμε δύο containers. Το ένα για το application και το άλλο για την βάση δεδομένων.

**Πρέπει το VM που θα χρησιμοποιήσουμε για το docker να έχει ανοιχτό το port `8080`**

**Εκτέλεση των playbooks**

```sh
cd project
ansible-playbook playbooks-docker/spring-docker.yaml -e "EMAIL_PASSWORD=email_password"
```

---

### Kubernetes Deployment

Σε αυτή την περίπτωση θα χρειαστούμε ένα VM. Η εφαρμογή και η βάση τρέχουν ως δύο ξεχωριστά deployments σε ένα private cluster.


**Πρέπει το VM που θα χρησιμοποιήσουμε για το microk8s να έχει ανοιχτό το port `80` για HTTP, `443` για HTTPS , `16443` για πρόσβαση στο Kubernetes API**

**Προετοιμασία (στο VM):** Θα πρέπει να εγκαταστήσουμε manually microk8s στο vm και να μεταφέρουμε το `~/.kube/config` στο τοπικό μηχάνημα ώστε να τρέξουμε τα αρχεία από το τοπικό μηχάνημα (θα χρειαστεί παραμετροποίηση το αρχείο `config`). 

Έγινε map το domain colabworks.ip-ddns.com στη public IP του VM μέσω του https://www.cloudns.net

**Εκτέλεση deployments**

- Φτίαχτηκε `alias k='kubectl'` για ευκολία.

```sh
k apply -f playbooks-microk8s/postgres-pvc.yaml
k apply -f playbooks-microk8s/postgres-deployment.yaml
k apply -f playbooks-microk8s/postgres-svc.yaml
```

Φτιάχνουμε secret για τα credentials του email:

```sh
kubectl create secret generic email-secret \
  --from-literal=EMAIL_USERNAME=... \
  --from-literal=EMAIL_PASSWORD=...
```

```sh
k apply -f playbooks-microk8s/spring-deployment.yaml
k apply -f playbooks-microk8s/spring-svc.yaml
k apply -f playbooks-microk8s/spring-ingress.yaml
```

Για τις παρακάτω δύο εντολές χρειάζεται να έχουμε εγκατεστημένο το `helm`.

```sh
k apply -f playbooks-microk8s/Cluster-issuer.yaml
k apply -f playbooks-microk8s/spring-ingress-ssl.yaml
```

Τώρα η εφαρμογή είναι διαθέσιμη στο https://colabworks.ip-ddns.com


### CI/CD (Jenkins)

Για το CI/CD setup χρησιμοποιήθηκε ένας Jenkins server σε ξεχωριστό VM.

**Πρέπει το VM που θα χρησιμοποιήσουμε για το Jenkins να έχει ανοιχτό το port `8080`**

Το playbook `install-jenkins-docker.yaml` κάνει εγκατάσταση Docker και στήνει τον Jenkins server.

```sh
cd project
ansible-playbook playbooks-jenkins/install-jenkins-docker.yaml
```

Τώρα έχουμε πρόσβαση στο http://<jenkins-vm-ip>:8080

---

**Χρήση του Jenkins στο microk8s:**

Στον Jenkins server που φτιάξαμε στο jenkins-vm δημιουργήσαμε δύο jobs. Το ένα κάνει clone 
το DS-project με webhook. Οπότε έχουμε πάντα την νεότερη έκδοση του κώδικα στον Jenkins 
server κάτω από τον κατάλογο /var/lib/workspace/jenkins. Το δεύτερο job κάνει clone το 
repository απο το github, όπου έχει μέσα όλα τα ansible playbooks. Αυτό το job, το 
ενσωματώσαμε μέσα στο pipeline `playbooks-jenkins/Jenkinsfile` που φτιάξαμε. Όπότε τώρα για να χειριστούμε κάποια αλλαγή, το μόνο που πρέπει να τρέξουμε είναι το βασικό pipeline.
Το pipeline, πέραν απο την εκτέλεση του job που κάνει clone το repository με τα ansible 
playbooks, εκτελέι άλλα 3 stages. Το ένα stage τρέχει τα tests του spring σε μια δοκιμαστική 
βάση δεδομένων (h2database). Το επόμενο stage δημιουργεί ένα TAG με το head commit και 
το build id, κάνει build το νέο image και το κάνει push στο github. Το τρίτο και τελευταίο stage, κάνει update το νέο image στο microk8s-vm.
