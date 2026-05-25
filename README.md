HackMyHome Voice Assistant – Waveshare ESP32-S3 Touch LCD 1.85C
Firmware ESPHome per trasformare la Waveshare ESP32-S3 Touch LCD 1.85C in un piccolo satellite vocale Home Assistant con display rotondo, touch, media player, microWakeWord, gestione timer/sveglia, pagina controlli locale, monitor batteria e sincronizzazione BLE tra due assistenti.
> Progetto nato per la serie **HackMyHome**: un voice assistant da scrivania compatto, visivo e interattivo, basato su ESP32-S3 e integrato con Home Assistant.
Oggetto usato
Prodotto: Waveshare ESP32-S3 Touch LCD 1.85C / 1.85C-BOX
Link Amazon: https://amzn.to/4dHuK3w
Link WaveShare: https://www.waveshare.com/esp32-s3-touch-lcd-1.85c.htm?sku=30684
Documentazione ufficiale Waveshare: https://docs.waveshare.com/ESP32-S3-Touch-LCD-1.85C
> Nota: il codice è pensato per la variante con codec audio **ES8311** e audio encoder **ES7210**, quindi per la revisione hardware recente/V2 della board. Prima di compilare, verificare la serigrafia della scheda e la versione hardware.
---
Caratteristiche principali
Home Assistant Voice Assistant tramite ESPHome.
microWakeWord integrato per wake word locali.
Modalità ascolto continuo attivabile/disattivabile da Home Assistant.
Media player ESPHome con pipeline separata per media e annunci.
Display rotondo 360×360 con UI animata in stile HackMyHome.
Touch CST816T con pagina controlli richiamabile tramite swipe.
Animazioni faccina / stato assistente in base alla fase del Voice Assistant.
Visualizzazione timer e sveglia con stato e nome del prossimo timer.
Antifurto acustico locale basato su soglia rumore.
Monitor batteria LiPo 1S tramite ADC.
Controllo luminosità display dal touch.
BLE Sync per rilevare un secondo Voice Assistant vicino.
Controllo aggiornamento firmware tramite file `version.yaml` remoto.
Azioni API Home Assistant per avvio/stop VA, colore LED/logica UI, sveglia e timezone.
---
Hardware supportato
Il firmware è stato preparato per:
Componente	Dettaglio
MCU	ESP32-S3R8
Flash	16 MB
PSRAM	8 MB Octal PSRAM
Display	1.85" rotondo, 360×360, ST77916 / JC3636W518V2, QSPI
Touch	CST816T su I2C
Audio DAC	ES8311
Audio ADC / microfoni	ES7210
Amplificatore	PA control su GPIO15
Batteria	LiPo 1S con lettura ADC su GPIO8
Connettività	Wi-Fi 2.4 GHz, Bluetooth BLE
---
Pin principali
Funzione	Pin
Display CLK	GPIO40
Display QSPI D0	GPIO46
Display QSPI D1	GPIO45
Display QSPI D2	GPIO42
Display QSPI D3	GPIO41
Display CS	GPIO21
Backlight	GPIO5
Reset display	PCA9554 / pin expander `1`
Touch INT	GPIO4
Reset touch	PCA9554 / pin expander `0`
I2C SCL	GPIO10
I2C SDA	GPIO11
I2S MCLK	GPIO2
I2S BCLK	GPIO48
I2S LRCLK	GPIO38
I2S DOUT speaker	GPIO47
I2S DIN microphone	GPIO39
Amplifier PA_CTRL	GPIO15
Battery ADC	GPIO8
---
Funzioni display
La UI usa il display circolare come superficie principale del Voice Assistant.
Stati gestiti:
Boot / inizializzazione
Idle / standby
Wake word rilevata
Ascolto comando
Elaborazione comando
Risposta TTS
Errore
Riproduzione media / musica
Timer attivo / timer scaduto
Modalità antifurto
Display addormentato / risveglio da touch o rumore
Il codice usa una variabile globale `voice_assistant_phase` per mappare le fasi principali:
Valore	Stato
`1`	Idle / pronto
`2`	In attesa del comando dopo la wake word
`3`	Ascolto attivo
`4`	Elaborazione
`5`	Risposta
`10`	Non pronto
`11`	Errore
---
Controlli touch
La pagina controlli si apre con uno swipe verso l’alto e si chiude con uno swipe verso il basso.
Quando la pagina controlli è aperta, i pulsanti touch sono organizzati così:
```text
          LUX

    VOL-   VA   VOL+

    MUTE        ALARM
```
Area touch	Azione
Alto centro	Cambia luminosità display: 30% → 65% → 100%
Sinistra centro	Volume giù
Centro	Avvia o ferma il Voice Assistant
Destra centro	Volume su
Basso sinistra	Mute / unmute media player
Basso destra	Attiva / disattiva modalità antifurto
La pagina resta visibile per circa 15 secondi dopo l’interazione.
---
Voice Assistant
Il firmware integra il componente `voice_assistant` di ESPHome con:
microfono I2S tramite ES7210;
media player speaker tramite ES8311;
microWakeWord;
modalità singola o continua;
gestione del ducking audio durante ascolto e annunci;
invio eventi Home Assistant per testo STT e URI TTS;
stop automatico della modalità continua dopo inattività prolungata.
Wake word incluse
Nel codice sono presenti i modelli:
`okay_nabu`
`kenobi`
`hey_jarvis`
`stop` interno per fermare timer/annunci
---
Comandi vocali locali
Il firmware intercetta alcuni comandi semplici direttamente sul dispositivo, senza demandare tutto alla logica esterna.
Esempi di parole chiave previste:
Intento locale	Frasi/parole intercettate
Volume su	`alza volume`, `aumenta volume`
Volume giù	`abbassa volume`, `riduci volume`
Modalità silenziosa	`modalita silenziosa`, `modalità silenziosa`
Antifurto ON	`attiva antifurto`, `abilita antifurto`, `inserisci antifurto`
Antifurto OFF	`disattiva antifurto`, `disabilita antifurto`, `disinserisci antifurto`
---
Media player e annunci
La parte audio usa una pipeline con:
speaker fisico I2S;
mixer speaker;
ingresso media;
ingresso annunci/TTS;
resampler separati;
gestione annunci con pipeline dedicata.
I file audio usati per feedback e timer vengono scaricati dal repository pubblico Home Assistant Voice PE:
`mute_switch_on.flac`
`mute_switch_off.flac`
`timer_finished.flac`
`wake_word_triggered.flac`
---
Modalità antifurto
Il firmware espone una modalità antifurto locale basata sul livello rumore.
Entità principali:
`Modalità antifurto`
`Soglia Rumore`
`Rilevamento rumore`
`Test antifurto`
Quando il rumore supera la soglia configurata, il dispositivo può:
risvegliare il display;
mostrare una reazione grafica;
impostare lo stato di allarme interno;
generare log di evento.
---
BLE Sync tra due Voice Assistant
Il codice include una logica di sincronizzazione BLE per due assistenti HackMyHome.
Componenti usati:
`esp32_ble_beacon`
`esp32_ble_tracker`
`ble_presence`
RSSI stabile del companion
Parametri principali:
```yaml
ble_sync_uuid: "7f4c8f1e-4f22-4d8f-9d8b-6c7a9f1a2026"
ble_sync_major: "2026"
my_ble_minor: "2"
ble_peer_minor: "1"
```
Entità esposte:
`Companion BLE Presente`
`RSSI altro Voice Assistant`
Questa base permette di creare interazioni tra due dispositivi quando sono vicini, ad esempio animazioni sincronizzate, cambio stato coordinato o logiche di presenza locale.
---
Batteria
La batteria viene letta tramite ADC su `GPIO8`.
Il firmware espone:
`${friendly} Battery ADC Raw`
`${friendly} Battery Voltage`
`${friendly} Battery`
La tensione batteria viene stimata moltiplicando la lettura ADC per `3.0`:
```yaml
return id(battery_adc_raw).state * 3.0f;
```
La percentuale è calcolata come LiPo 1S semplice:
```text
3.20 V = 0%
4.20 V = 100%
```
> La curva è volutamente semplice. Per una stima più realistica si può sostituire con una curva LiPo non lineare.
---
Entità principali in Home Assistant
Select
`Display - Emozione`
`WakeWord - Sensibilita`
`Livello log`
`Sveglia - Azione`
Switch
`Modalità antifurto`
`Assistente - Ascolto continuo`
`Amplificatore`
`Microfono disattivato`
`Suono Mute-Unmute`
`Suono Wake Word`
`Timer ringing`
`Sveglia attiva`
`Controllo nuovo firmware`
Number
`Soglia Rumore`
`Mic-Guadagno`
`VA-Guadagno`
`Sopp. Rumore`
Sensor / binary sensor
`Prossimo timer`
`Prossimo timer - Nome`
`Battery Voltage`
`Battery`
`RSSI altro Voice Assistant`
`Companion BLE Presente`
`Rilevamento rumore`
Button
`Test antifurto`
`Ripristino di fabbrica`
`Riavvia`
`Controlla nuovo firmware`
---
Azioni API esposte a Home Assistant
Il firmware espone alcune azioni richiamabili da Home Assistant:
Azione	Descrizione
`set_led_color`	Aggiorna i valori colore usati dalla UI/logica interna
`start_va`	Avvia il Voice Assistant
`stop_va`	Ferma il Voice Assistant
`set_alarm_time`	Imposta la sveglia nel formato `HH:MM`
`set_time_zone`	Imposta la timezone POSIX
---
Prerequisiti
Home Assistant funzionante.
ESPHome compatibile con `min_version: 2025.8.0` o superiore.
Board Waveshare ESP32-S3 Touch LCD 1.85C, preferibilmente revisione V2.
Wi-Fi 2.4 GHz.
Pipeline Voice Assistant configurata in Home Assistant.
Accesso a ESPHome Dashboard o compilazione locale.
---
Installazione rapida
Copiare il file YAML in ESPHome.
Aggiornare le sostituzioni iniziali:
```yaml
device_name: home-assistant-voice-speaker-02
friendly: "Speaker-04"
fw_version: "1.0.0"
ssid: "WiFi SSID"
password: "WiFi Password"
ip: "192.168.xxx.xxx"
gateway: "192.168.xxx.xxx"
subnet: "255.255.255.0"
dns: "192.168.xxx.xxx"
```
Impostare correttamente:
```yaml
api_key: "..."
ota_key: "..."
```
Compilare da ESPHome.
Flashare la board via USB-C.
Aprire Home Assistant e verificare la comparsa delle entità.
Testare prima:
display backlight;
touch;
media player;
microfono;
wake word;
batteria;
BLE sync se si usa un secondo assistente.
---
Consiglio sicurezza
Prima di pubblicare il codice su GitHub:
non caricare password Wi-Fi reali;
non caricare chiavi API reali;
non caricare chiavi OTA reali;
spostare i dati sensibili in `secrets.yaml`;
verificare che IP, SSID e token non siano presenti nello storico Git.
Esempio consigliato:
```yaml
ssid: !secret wifi_ssid
password: !secret wifi_password
api_key: !secret api_key
ota_key: !secret ota_key
```
---
Troubleshooting
Il display resta nero
Verificare:
backlight su GPIO5;
reset display tramite PCA9554 pin `1`;
bus QSPI e modello `JC3636W518V2`;
`data_rate: 80MHz`, da ridurre a 40MHz in caso di instabilità.
Il touch non risponde
Verificare:
indirizzo `0x15`;
interrupt su GPIO4;
reset touch tramite PCA9554 pin `0`;
I2C su GPIO10/GPIO11.
L’audio non funziona
Verificare:
presenza ES8311 a indirizzo `0x18`;
presenza ES7210 a indirizzo `0x40`;
`PA_CTRL` su GPIO15;
clock I2S condivisi;
modalità `primary/secondary` coerente con la revisione hardware.
La batteria legge valori strani
Verificare:
collegamento batteria;
ADC su GPIO8;
fattore di moltiplicazione `3.0f`;
attenuazione `12db`;
curva percentuale 3.20–4.20 V.
microWakeWord non riparte
Il codice include uno script `ensure_wake_word_running` che riavvia microWakeWord solo se:
API connessa;
Voice Assistant connesso;
microfono non mutato;
timer non in ringing;
media player non in riproduzione o annuncio;
Voice Assistant non già in esecuzione.
---
Roadmap possibili
Sincronizzazione grafica tra due assistenti via BLE.
Animazioni coordinate quando i due dispositivi sono vicini.
Pagina meteo/Home Assistant sul display.
Dashboard locale per timer, sveglia e stato casa.
Miglioramento curva batteria LiPo.
Modalità notte con luminosità adattiva.
UI dedicata per volume media e stato TTS.
---
Crediti
Progetto e personalizzazione: HackMyHome
Hardware: Waveshare ESP32-S3 Touch LCD 1.85C
Firmware base: ESPHome
Integrazione domotica: Home Assistant
---
Disclaimer
Questo progetto è sperimentale e pensato per uso maker/domotico. Verificare sempre pinout, revisione hardware, alimentazione, batteria e configurazioni audio prima dell’uso continuativo.
