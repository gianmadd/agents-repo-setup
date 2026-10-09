# Agents Repo Setup

Uno strumento che, lanciato dentro un repo, gli dà una base fissa di istruzioni per gli agenti di coding, così non vanno ripetute a ogni sessione.

## Language

**Setup**:
La skill che si lancia una volta in un repo e scrive le Istruzioni del repo dopo aver verificato i Prerequisiti.
_Avoid_: installer, bootstrap, init

**Istruzioni del repo**:
Il blocco scritto dal Setup in `AGENTS.md`, letto da ogni agente; in un repo locale con un `AGENTS.md` del team va invece in `CLAUDE.local.md`.
_Avoid_: prompt, config, regole globali

**Configurazione consigliata**:
L'insieme predefinito di skill, plugin e regole che il Setup propone e che l'utente accetta, toglie o integra repo per repo; ogni skill o plugin indica quando usarlo, come si attiva, come si installa e il fallback per gli agenti che non ce l'hanno.
_Avoid_: catalogo, default, preset

**Prerequisito**:
Una skill o un plugin che le Istruzioni del repo nominano e che deve essere installato perché funzionino.
_Avoid_: dipendenza, requirement
