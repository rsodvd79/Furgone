# FURGONE — ESP8266 OLED Controller

Firmware PlatformIO per ESP8266 (ESP‑12E) con display OLED 128×32 (SSD1306, I²C), interfaccia web su LittleFS e modulo Meteo basato su OpenWeatherMap. Consente di:

- Visualizzare varie schermate su OLED: Wi‑Fi, Ora/Data, Eyes, Meteo, Game of Life, ciclo schermi
- Configurare rete Wi‑Fi e nome mDNS da web UI
- Recuperare condizioni meteo correnti via API OpenWeatherMap


## Requisiti

- PlatformIO (VS Code estensione o CLI)
- Scheda ESP8266 ESP‑12E (o compatibile, es. NodeMCU)
- Librerie Arduino (installate nella tua cartella Arduino o come `lib_deps`):
  - ESP8266WiFi (core ESP8266)
  - ESP8266HTTPClient (core ESP8266)
  - ESP8266mDNS (core ESP8266)
  - LittleFS (core ESP8266)
  - ArduinoJson
  - Adafruit GFX Library
  - Adafruit SSD1306

Il progetto è configurato in `platformio.ini` per l’ambiente `esp12e` e usa `lib_extra_dirs = ~/Documents/Arduino/libraries`. In alternativa puoi usare `lib_deps` di PlatformIO se preferisci che PlatformIO gestisca automaticamente le dipendenze.


## Hardware

- MCU: ESP8266 ESP‑12E
- Display: OLED 128×32 SSD1306 via I²C (indirizzo 0x3C)
  - Pin I²C ESP8266 di default: SDA = D2 (GPIO4), SCL = D1 (GPIO5)


## Struttura progetto

- `src/Furgone.ino`: firmware principale (OLED, web server, Wi‑Fi, mDNS, gestione schermate)
- `src/meteo.cpp`, `src/meteo.h`: modulo meteo (OpenWeatherMap, parsing JSON)
- `src/screen.*`, `src/eyes.*`, `src/GOL.*`, `src/Button.*`: componenti di visualizzazione/controllo
- `src/data/`: file per LittleFS (interfaccia web e configurazioni)
  - `INDEX.HTM`: controller web (selezione schermate, stato dispositivo)
  - `SETUP.HTM`: pagina di setup della rete Wi‑Fi
  - `WIFI_SID.TXT`, `WIFI_PWD.TXT`: SSID e password Wi‑Fi
  - `MDNS_NM.TXT`: nome mDNS (es. `FURGONE`)
  - `SCREEN.TXT`: indice schermata da avviare
  - `METEO_API.TXT`: query string per OpenWeatherMap (vedi sotto)


## Configurazione (LittleFS)

Il firmware legge le impostazioni da LittleFS all’avvio. Prepara i file nella cartella `src/data/` prima di caricare l’immagine del filesystem:

- `WIFI_SID.TXT`: SSID della rete
- `WIFI_PWD.TXT`: password della rete
- `MDNS_NM.TXT` (opzionale): nome mDNS (es. `FURGONE`), usato anche come SSID in modalità AP
- `SCREEN.TXT` (opzionale): numero schermata iniziale (es. `1`)
- `METEO_API.TXT`: parte di query per OpenWeatherMap, per es.:
  
  ```
  id=<CITY_ID>&cnt=1&units=metric&lang=it&appid=<API_KEY>
  ```

Sicurezza: evita di committare chiavi/API e credenziali. Mantieni `METEO_API.TXT`, `WIFI_*` fuori dal VCS o usa segreti separati.


## Build e Upload

Da VS Code (PlatformIO):

1. Upload LittleFS: target “Upload Filesystem Image” (carica il contenuto di `src/data/`)
2. Compila e flash: target “Upload” sull’ambiente `esp12e`

Da CLI:

- Build: `pio run -e esp12e`
- Upload LittleFS: `pio run -e esp12e -t uploadfs`
- Upload firmware: `pio run -e esp12e -t upload`


## Utilizzo

- All’avvio il dispositivo legge le credenziali da LittleFS e prova a connettersi in modalità STA.
- Se fallisce, avvia un Access Point aperto con SSID uguale al nome mDNS (es. `FURGONE`) e mostra l’IP su OLED.
- Interfaccia web:
  - In STA: `http://<mdns>.local/` oppure `http://<ip>`
  - In AP: `http://<ip_ap>` (mostrato su OLED)
- Dalla Home puoi cambiare schermata (Wi‑Fi, Ora, Eyes, Meteo, GOL, ciclo) e accedere a “Setup Wi‑Fi” per salvare SSID/password/mDNS.

Il modulo Meteo usa OpenWeatherMap: il metodo `classMeteo::Update()` effettua una GET su `http://api.openweathermap.org/data/2.5/weather?` concatenando il contenuto di `METEO_API.TXT` (temperatura, umidità, vento, descrizione, icona, eventuale neve 1h).


## Note di sviluppo

- Board e framework: vedi `platformio.ini`
- Display: `Adafruit_SSD1306` a 0x3C
- File principali: `src/Furgone.ino`, `src/meteo.cpp`, `src/meteo.h`


## Licenza

Specifica qui la licenza del progetto (es. MIT). Attualmente nessuna licenza è dichiarata.

