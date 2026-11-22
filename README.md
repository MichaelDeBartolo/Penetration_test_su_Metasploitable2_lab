# Cybersecurity Metasploitable2 Lab
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
