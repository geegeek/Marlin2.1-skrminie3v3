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

Per ricreare la mia configurazione partendo da un Marlin “vanilla”, procedi nell’ordine seguente:

### 1. Identità della stampante e comunicazione (`Configuration.h`)
- Imposta la scheda madre con `#define MOTHERBOARD BOARD_BTT_SKR_MINI_E3_V3_0`.
- Abilita la doppia seriale impostando `#define SERIAL_PORT 2` e `#define SERIAL_PORT_2 -1` con `#define BAUDRATE 115200`.
- Personalizza il nome macchina con `#define CUSTOM_MACHINE_NAME "Ender-3 Pro"`.

### 2. Geometria, motori e cinematiche (`Configuration.h`)
- Imposta il volume di lavoro a 235×235×250 mm (`X_BED_SIZE`, `Y_BED_SIZE`, `Z_MAX_POS`).
- Definisci gli endstop meccanici agli angoli minimi con stato alto (`#define X_MIN_ENDSTOP_HIT_STATE HIGH`, `#define Y_MIN_ENDSTOP_HIT_STATE HIGH`, `#define Z_MIN_PROBE_ENDSTOP_HIT_STATE HIGH`) e abilita entrambi i software endstop (`#define MIN_SOFTWARE_ENDSTOPS`, `#define MAX_SOFTWARE_ENDSTOPS`).
- Inverti il verso dei motori X/Y/E con `#define INVERT_X_DIR true`, `#define INVERT_Y_DIR true`, `#define INVERT_E0_DIR true` mentre Z resta `false`.
- Imposta i passi/mm con `#define DEFAULT_AXIS_STEPS_PER_UNIT   { 80, 80, 400, 400 }` e le velocità massime `{ 500, 500, 5, 25 }` (`DEFAULT_MAX_FEEDRATE`).
- Configura accelerazioni massime `{ 500, 500, 100, 5000 }` (`DEFAULT_MAX_ACCELERATION`) e operative (`DEFAULT_ACCELERATION`, `DEFAULT_RETRACT_ACCELERATION`, `DEFAULT_TRAVEL_ACCELERATION`).
- Abilita `JUNCTION_DEVIATION_MM 0.08` e `#define S_CURVE_ACCELERATION`.

### 3. Sonda BLTouch e livellamento (`Configuration.h`)
- Attiva il BLTouch (`#define BLTOUCH`) e imposta l’offset `#define NOZZLE_TO_PROBE_OFFSET { -44, -9, 3.5 }`.
- Abilita `#define USE_PROBE_FOR_Z_HOMING` e `#define Z_SAFE_HOMING` con centro piatto.
- Definisci le altezze di sicurezza (`Z_CLEARANCE_DEPLOY_PROBE`, `Z_CLEARANCE_BETWEEN_PROBES`, `Z_CLEARANCE_MULTI_PROBE`) a 10/5/5 mm.
- Imposta le velocità del probing (`#define XY_PROBE_FEEDRATE (133*60)`, `#define Z_PROBE_FEEDRATE_FAST (4*60)`, `#define Z_PROBE_FEEDRATE_SLOW (Z_PROBE_FEEDRATE_FAST / 2)`) e il margine con `#define PROBING_MARGIN 10`.
- Abilita il bilinear mesh 5×5 (`#define AUTO_BED_LEVELING_BILINEAR`, `#define GRID_MAX_POINTS_X 5`) con estrapolazione (`#define EXTRAPOLATE_BEYOND_GRID`), fade a 10 mm (`#define DEFAULT_LEVELING_FADE_HEIGHT 10.0`) e segmentazione dei movimenti (`#define SEGMENT_LEVELED_MOVES`, `#define LEVELED_SEGMENT_LENGTH 5.0`).

### 4. Termica e gestione ventole (`Configuration.h`)
- Usa termistori tipo 1 per hotend e piano (`TEMP_SENSOR_0` e `TEMP_SENSOR_BED`).
- Inserisci i PID già calibrati (`DEFAULT_Kp 21.73`, `DEFAULT_Ki 1.54`, `DEFAULT_Kd 76.55` per l’hotend e `DEFAULT_bedKp 50.71`, `DEFAULT_bedKi 9.88`, `DEFAULT_bedKd 173.43`).
- Configura i preset di preriscaldamento (`PREHEAT_1` PLA 185/45/255 e `PREHEAT_2` ABS 240/110/255).
- Imposta la ventola di raffreddamento partendo dal firmware (vedi dettagli in `Configuration_adv.h`).

### 5. EEPROM, sicurezza e qualità di vita (`Configuration.h`)
- Abilita la memoria non volatile (`#define EEPROM_SETTINGS` e `#define EEPROM_AUTO_INIT`) e i messaggi verbosi (`#define EEPROM_CHITCHAT`).
- Imposta il keepalive dell'host con `#define HOST_KEEPALIVE_FEATURE` e intervallo `#define DEFAULT_KEEPALIVE_INTERVAL 2`.
- Seleziona il display grafico Creality con `#define CR10_STOCKDISPLAY` e abilita i menu di livellamento (`#define LCD_BED_LEVELING` e `#define LCD_BED_TRAMMING`).

### 6. Cambio filamento e runout (`Configuration_adv.h` e `Configuration.h`)
- In `Configuration_adv.h` abilita `#define ADVANCED_PAUSE_FEATURE` e imposta i parametri su misura del Micro Swiss NG:
  - `#define PAUSE_PARK_RETRACT_LENGTH 2`
  - `#define FILAMENT_CHANGE_UNLOAD_LENGTH 40`
  - `#define FILAMENT_CHANGE_FAST_LOAD_FEEDRATE 6`
  - `#define FILAMENT_CHANGE_FAST_LOAD_LENGTH 60`
  - `#define ADVANCED_PAUSE_PURGE_LENGTH 50`
  - `#define FILAMENT_UNLOAD_PURGE_RETRACT 13`, `#define FILAMENT_UNLOAD_PURGE_DELAY 5000`, `#define FILAMENT_UNLOAD_PURGE_LENGTH 8`
  - Mantieni `#define PARK_HEAD_ON_PAUSE`, `#define HOME_BEFORE_FILAMENT_CHANGE`, `#define FILAMENT_LOAD_UNLOAD_GCODES`, `#define CONFIGURE_FILAMENT_CHANGE` e `#define PAUSE_PARK_NO_STEPPER_TIMEOUT`.
- In `Configuration.h` lascia pronto lo script `#define FILAMENT_RUNOUT_SCRIPT "M600"` (il sensore è facoltativo ma la routine è già predisposta).

### 7. Driver TMC2209 (`Configuration.h` e `Configuration_adv.h`)
- Seleziona i driver TMC2209 (`#define X_DRIVER_TYPE  TMC2209` ecc.) e imposta le correnti RMS a 580 mA (X/Y/Z) e 650 mA (E0) tramite `#define X_CURRENT`, `#define Y_CURRENT`, `#define Z_CURRENT`, `#define E0_CURRENT`.
- Mantieni `#define HOLD_MULTIPLIER 0.5` e `#define INTERPOLATE true` con microstepping a 1/16 e `RSENSE 0.11` su tutti gli assi.
- Abilita StealthChop (`#define STEALTHCHOP_XY`, `#define STEALTHCHOP_Z`, `#define STEALTHCHOP_E`) e lascia il chopper default 24 V (`#define CHOPPER_TIMING CHOPPER_DEFAULT_24V`).
- Riduci la corrente in homing con `#define X_CURRENT_HOME (X_CURRENT/2)`, `#define Y_CURRENT_HOME (Y_CURRENT/2)` e `#define Z_CURRENT_HOME (Z_CURRENT/2)`.

### 8. Funzioni avanzate e interfaccia (`Configuration_adv.h`)
- Abilita il wizard LCD per la sonda con `#define PROBE_OFFSET_WIZARD`.
- Configura il babystepping evoluto con `#define DOUBLECLICK_FOR_Z_BABYSTEPPING`, `#define DOUBLECLICK_MAX_INTERVAL 1250` e `#define BABYSTEP_ZPROBE_OFFSET`, lasciando commentato `#define BABYSTEP_DISPLAY_TOTAL`.
- Attiva il watchdog hardware con `#define USE_WATCHDOG` e il parser di emergenza con `#define EMERGENCY_PARSER`.
- Imposta la ventola estrusore automatica (`#define E0_AUTO_FAN_PIN FAN1_PIN`, `#define EXTRUDER_AUTO_FAN_TEMPERATURE 50`, `#define EXTRUDER_AUTO_FAN_SPEED 255`).
- Abilita il Linear Advance (`#define LIN_ADVANCE` e `#define ADVANCE_K 0.0`) e il supporto agli archi G2/G3 (`#define ARC_SUPPORT`).
- Abilita la reportistica automatica verso l'host (`#define AUTO_REPORT_SD_STATUS`, `#define AUTO_REPORT_TEMPERATURES`, `#define AUTO_REPORT_POSITION`) e i prompt (`#define HOST_ACTION_COMMANDS`, `#define HOST_PROMPT_SUPPORT`).

## Risorse ufficiali Marlin

- [Sito principale di Marlin](https://marlinfw.org) per documentazione e novità.
- [Repository MarlinFirmware/Configurations](https://github.com/MarlinFirmware/Configurations) con profili di esempio.
- [Guida Auto Build Marlin](https://marlinfw.org/docs/basics/auto_build_marlin.html) per compilare con Visual Studio Code.
- Canali di supporto: [Discord ufficiale](https://discord.com/servers/marlin-firmware-461605380783472640) e [Issue tracker su GitHub](https://github.com/MarlinFirmware/Marlin/issues/new/choose).

Questa sezione sostituisce il README generico di Marlin mantenendo solo i link più utili all'ecosistema ufficiale.
