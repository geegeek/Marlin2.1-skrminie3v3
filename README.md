<p align="center"><img src="buildroot/share/pixmaps/logo/marlin-outrun-nf-500.png" height="250" alt="MarlinFirmware's logo" /></p>

# Firmware Marlin personalizzato per Ender 3 V2 (SKR Mini E3 V3 + Micro Swiss NG + BLTouch)

Questo repository contiene la mia configurazione di Marlin 2.1 pensata per una Creality Ender 3 V2 aggiornata con scheda madre BTT SKR Mini E3 V3.0, estrusore Micro Swiss NG Direct Drive, sensore BLTouch 3.1 e driver TMC2209. Il README è una memoria storica delle modifiche applicate rispetto ai valori di default di Marlin così da poterle replicare velocemente in caso di upgrade futuri.

## Panoramica hardware

| Componente | Dettagli configurati in Marlin |
|------------|--------------------------------|
| Scheda madre | `BOARD_BTT_SKR_MINI_E3_V3_0` con porta seriale principale 2 a 115200 baud e porta secondaria USB (`-1`). |
| Cinematica | Volume utile 235×235×250 mm, endstop agli angoli minimi, direzione motori X/Y invertita, Z e asse estrusore diretti. |
| Estrusione | Un solo estrusore diretto Micro Swiss NG con e-step fissati a 400 step/mm. |
| Sensori | BLTouch 3.1 fissato a sinistra dell'ugello (offset -44 mm in X, -9 mm in Y, +3.5 mm in Z). |
| Interfaccia utente | Display Creality/CR-10 a 128×64 e voce macchina impostata su "Ender-3 Pro". |
| Driver | TMC2209 su tutti gli assi con corrente 580 mA (650 mA su E0) e StealthChop attivo. |

## Configurazione principale (`Configuration.h`)

### Comunicazione e identificazione
- `SERIAL_PORT` 2 per la porta UART integrata sulla SKR Mini e `SERIAL_PORT_2` a `-1` per l'USB virtuale.
- Baud rate fisso a 115200.
- Nome macchina mostrato sul display: `Ender-3 Pro`.

### Geometria e movimento
- Volume dichiarato: 235 mm per X e Y, 250 mm per Z, con software endstop minimi attivi (Z esclusa) e massimi attivati.
- Direzione motori: X e Y invertiti (`true`), Z normale (`false`), estrusore diretto invertito (`true`).
- Passi per mm: `{80, 80, 400, 400}` (nota: 400 step/mm per E0 tarato sul Micro Swiss NG).
- Velocità massime: `{500, 500, 5, 25}` mm/s per X/Y/Z/E.
- Accelerazioni massime: `{500, 500, 100, 5000}` con accelerazioni operative 500 mm/s² (stampa e travel) e 100 mm/s² in retrazione.
- Dinamica avanzata: Junction Deviation a 0.08 mm con gestione dei segmenti corti abilitata e accelerazione S-Curve attiva.

### Homing, probing e livellamento
- Utilizzo obbligatorio della sonda per l'homing Z e funzione Z Safe Homing al centro del piatto.
- BLTouch abilitato con margine di probing di 10 mm e velocità di spostamento 133 mm/s tra i punti.
- Offset sonda/nozzle `{-44, -9, 3.5}` mm.
- Altezze di sicurezza: 10 mm per deploy, 5 mm tra i punti e per il multi probing.
- Probing rapido a 4 mm/min, seconda passata a metà velocità, profondità minima -2 mm.
- Livellamento automatico bilineare 5×5, con estrapolazione oltre la griglia e fade-out fino a 10 mm; movimenti suddivisi in segmenti da 5 mm.

### Termica e gestione temperature
- Termistori tipo 1 su hotend e piano.
- PID attivo per hotend (`Kp 21.73 / Ki 1.54 / Kd 76.55`) e piano (`Kp 50.71 / Ki 9.88 / Kd 173.43`).
- Preheat rapidi: profilo PLA 185 °C / 45 °C / ventola 255 e ABS 240 °C / 110 °C / ventola 255.
- Ventola estrusore collegata al pin FAN1 con attivazione automatica sopra i 50 °C.

### Memoria non volatile e sicurezza
- EEPROM attiva con inizializzazione automatica e messaggi `M500/M501` verbosi.
- Keepalive host ogni 2 secondi e watchdog attivato.
- Funzione `EMERGENCY_PARSER` per comandi critici immediati.

### Qualità di vita e interfaccia
- Menu LCD `CR10_STOCKDISPLAY` con wizard per l'offset della sonda, info menu e babystepping.
- Babystepping su Z con doppio click rapido (1250 ms) e combinazione con M851 per salvare l'offset.
- Linear Advance abilitato (K attuale 0.0 da calibrare in base al materiale).
- Supporto alle archi G2/G3 e movimenti manuali con beep feedback.

### Pause, cambio filamento e runout
- Funzione `ADVANCED_PAUSE_FEATURE` con ritrazioni corte (2 mm), unload da 40 mm e load veloce 60 mm, purge finale 50 mm, mantenendo motori attivi e parcheggio automatico della testina.
- G-code rapidi `M701/M702` e comando `M603` per configurare i cambi filamento.
- Script runout di default `M600` (sensore disabilitato di default ma flusso già predisposto).

### Driver stepper (TMC2209)
- Corrente RMS 580 mA su X/Y/Z (ridotta a metà in homing) e 650 mA su E0, microstepping 1/16.
- StealthChop abilitato su tutti gli assi e timing chopper `CHOPPER_DEFAULT_24V`.

### Report e integrazione host
- Auto report stato SD (`M27 S`) e temperature (`M155 S`) + posizione (`M154 S`).
- Comandi host action/prompt abilitati con supporto `M76`.
- Parser G-code esteso (`CAPABILITIES_REPORT`) con report ventola su cambio velocità.

## Configurazione avanzata (`Configuration_adv.h`)

Oltre ai punti elencati sopra, in `Configuration_adv.h` sono stati attivati:

- Ventola hotend automatica su FAN1 e velocità massima (255).
- Wizard LCD per calibrare l'offset della sonda (`PROBE_OFFSET_WIZARD`).
- Watchdog hardware abilitato (`USE_WATCHDOG`).
- Babystepping con doppio click, collegamento automatico all'offset della sonda e opzione di visualizzazione totale disabilitata per mantenere il menu pulito.
- Linear Advance pronto all'uso (K da tarare).
- Funzioni di pausa avanzate con parametri personalizzati per il Micro Swiss NG.
- Supporto agli archi G2/G3 per percorsi più fluidi.
- Corrente e modalità dei driver TMC come riportato in tabella hardware.
- Auto-report SD/temperature/posizione e comandi host (pause, prompt) abilitati.

## Come replicare/aggiornare la configurazione

1. **Partire dai file di configurazione**: copiare `Configuration.h` e `Configuration_adv.h` in un nuovo sorgente Marlin e verificare prima i blocchi relativi a scheda, driver, sonde e volume di stampa.
2. **Ricontrollare gli offset del BLTouch** dopo eventuali modifiche meccaniche (carrelli, staffe ecc.).
3. **Ritarare PID ed e-step** se cambiano hotend, estrusore o termistori: i valori presenti sono specifici per Micro Swiss NG e termistori Creality.
4. **Rivalutare Linear Advance**: il valore K a 0.0 è un segnaposto; dopo la calibrazione aggiornare `ADVANCE_K` e salvare in EEPROM.
5. **Testare il cambio filamento** se si variano lunghezze Bowden/Direct Drive, adattando le distanze di carico/scarico nel blocco `ADVANCED_PAUSE_FEATURE`.

## Risorse ufficiali Marlin

- [Sito principale di Marlin](https://marlinfw.org) per documentazione e novità.
- [Repository MarlinFirmware/Configurations](https://github.com/MarlinFirmware/Configurations) con profili di esempio.
- [Guida Auto Build Marlin](https://marlinfw.org/docs/basics/auto_build_marlin.html) per compilare con Visual Studio Code.
- Canali di supporto: [Discord ufficiale](https://discord.com/servers/marlin-firmware-461605380783472640) e [Issue tracker su GitHub](https://github.com/MarlinFirmware/Marlin/issues/new/choose).

Questa sezione sostituisce il README generico di Marlin mantenendo solo i link più utili all'ecosistema ufficiale.
