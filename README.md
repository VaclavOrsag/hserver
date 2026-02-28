# HOME CONTROL SERVER

## O CO SE POKOUŠÍME
Vyvíjíme domácí server, který běží na Linux stroji (aktuálně T480 s Pop!_OS) a slouží jako centrální řídicí a monitorovací bod domácí sítě.

Nejde o běžnou webovou aplikaci. Jde o domácí infrastrukturu.

**Server:**
- monitoruje zařízení v domácí síti
- umožňuje jejich probouzení přes Wi-Fi (Wake-on-Wireless-LAN / WoWLAN)
- ukládá historii stavů a běhu
- poskytuje webový dashboard a REST API
- je navržen tak, aby šel později přesunout na Raspberry Pi

T480 je aktuálně vývojový i produkční stroj. Později bude T480 pouze vývojový stroj a Raspberry Pi poběží 24/7 jako produkční server.

---

## JAK SE SYSTÉM BUDE CHOVAT (FUNKČNÍ POPIS)
Server běží nepřetržitě na T480 a plní čtyři hlavní role:

### A) MONITOR ZAŘÍZENÍ
Server pravidelně (např. každých 5–10 sekund):
- pingne definovaná zařízení (např. gaming PC)
- zjistí, zda jsou ONLINE nebo OFFLINE
- pokud dojde ke změně stavu, uloží event do databáze

**Ukládají se tyto informace:**
- čas změny
- typ změny (online, offline)
- zařízení, kterého se to týká

Tím vzniká historie dostupnosti zařízení.

### B) WoWLAN (PROBUZENÍ PC PŘES WI-FI)
Server umožňuje probudit desktop PC přes Wi-Fi síť. Kvůli chybějícímu ethernetovému připojení se PC nevypíná úplně, ale využívá se režim spánku (S3).

**Proces:**
1. Uživatel klikne na tlačítko „Wake PC“ na dashboardu nebo zavolá API endpoint.
2. Server odešle tzv. magic packet na MAC adresu PC.
3. PC se probudí ze spánku (vyžaduje správně nastavený BIOS a správce zařízení ve Windows).
4. Server začne kontrolovat, zda PC reaguje na ping.

Jakmile je PC dostupné, uloží event:
- `wake_sent`
- `pc_online`

Tato funkce bude reálně používaná každý den.

### C) SESSION TRACKING
Server automaticky vytváří session běhu zařízení.

**Logika:**
- OFFLINE → ONLINE = start session
- ONLINE → OFFLINE = end session

**Ukládá se:**
- `start_time`
- `end_time`
- `duration`

**Díky tomu lze zobrazit:**
- jak dlouho běželo PC dnes
- poslední session
- počet session za týden

### D) WEB DASHBOARD
Server poskytuje webové rozhraní přístupné v lokální síti. 
Například: `http://192.168.0.14:8000`

**Dashboard zobrazuje:**
- seznam zařízení
- jejich aktuální stav (ONLINE / OFFLINE)
- tlačítko pro probuzení (WoWLAN)
- poslední eventy
- poslední session

UI je v první fázi jednoduché. Důležitá je funkčnost a stabilita backendu.

### E) REST API
Kromě dashboardu server poskytuje API.

**Příklad endpointů:**
- `GET /api/status` – Vrátí aktuální stav zařízení.
- `POST /api/wake/desktop` – Odesílá Magic Packet pro WoWLAN.
- `GET /api/sessions` – Vrátí historii session.
- `GET /api/events` – Vrátí poslední eventy.

API umožní pozdější rozšíření systému nebo napojení dalších služeb.

---

## ARCHITEKTURA (MVP)
**Na T480 běží:**
- FastAPI server (API + web dashboard)
- Monitorovací služba (background loop)
- SQLite databáze
- WoWLAN modul

**Později:**
- oddělení do systemd služeb
- běh 24/7
- přesun na Raspberry Pi

---

## ROADMAPA (VĚTŠÍ KROKY)
- **FÁZE 1 – WoWLAN základ**
  - nastavit PC pro probouzení z režimu spánku přes Wi-Fi (BIOS, Windows)
  - implementovat odeslání magic packetu v Pythonu
  - otestovat probuzení PC ze skriptu
- **FÁZE 2 – API server**
  - vytvořit FastAPI aplikaci
  - přidat `/health` a `/wake` endpoint
  - ověřit funkčnost přes HTTP request
- **FÁZE 3 – Monitoring**
  - vytvořit periodický ping loop
  - detekovat změnu stavu
  - logovat změny
- **FÁZE 4 – Databáze**
  - vytvořit tabulky `devices`, `events`, `sessions`
  - ukládat historii běhu
- **FÁZE 5 – Dashboard**
  - zobrazit aktuální stav zařízení
  - přidat Wake tlačítko
  - zobrazit historii
- **FÁZE 6 – Production režim**
  - systemd služby
  - automatický start po rebootu
  - stabilní běh 24/7

---

## HLAVNÍ DESIGN PRINCIPY
- žádné hardcoded IP adresy
- konfigurace přes `.env`
- oddělení logiky (API / monitor / wol / db)
- přenositelnost na Raspberry Pi
- jednoduchý, stabilní backend před hezkým UI

---

## CÍL PROJEKTU
Cílem není jen „naprogramovat aplikaci“.

**Cílem je:**
- pochopit fungování domácí infrastruktury
- naučit se backend + Linux + networking
- vybudovat reálně používaný systém
- vytvořit projekt, který má hodnotu i do CV