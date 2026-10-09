# Agents Repo Setup

Uno strumento che, lanciato dentro un repo, gli dà una base fissa di istruzioni per gli agenti di coding, così non vanno ripetute a ogni sessione.

## Language

**Setup**:
La skill che si lancia una volta in un repo e scrive le Istruzioni del repo dopo aver verificato i Prerequisiti.
_Avoid_: installer, bootstrap, init

**Istruzioni del repo**:
Il blocco scritto dal Setup in `AGENTS.md`, letto da ogni agente; in un repo locale con un `AGENTS.md` del team va invece in `CLAUDE.local.md`.
_Avoid_: prompt, config, regole globali

**Valori personali**:
Gli account e le identità dell'utente (account GitHub, tenant e utente Azure) che le Istruzioni del repo citano ma non contengono; stanno in `AGENTS.local.md`, mai committato.
_Avoid_: segreti, credenziali, config locale

**Configurazione di default**:
L'insieme di skill, plugin e regole che il Setup usa come risposta di default alle sue domande, e che l'utente tiene, cambia o integra repo per repo; ogni skill o plugin indica quando usarlo, come si attiva, come si installa e come si spegne nel repo.
_Avoid_: configurazione consigliata, catalogo, preset

**Prerequisito**:
Una skill o un plugin che le Istruzioni del repo nominano e che deve essere installato perché funzionino.
_Avoid_: dipendenza, requirement
