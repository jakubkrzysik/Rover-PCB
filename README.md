# Projekt PCB dla Łazika (Rover-PCB)

Repozytorium zawiera projekt płytki drukowanej w programie **KiCad 9.0** dla autonomicznego lub zdalnie sterowanego łazika (Rover). 

Projekt składa się z **dwóch fizycznych płytek** zaprojektowanych na jednym schemacie (`Rover_PCB.kicad_sch`) i w dwóch osobnych plikach PCB (`Rover_PCB.kicad_pcb` oraz `Rover_PCB _Power.kicad_pcb`), które łączą się ze sobą za pomocą złącza ATX oraz taśm FFC (Flat Flexible Cable).

---

## Główny Mikrokontroler

Sercem układu sterowania jest potężny mikrokontroler:
* **Model:** `STM32H743VITx` (obudowa LQFP-100)
* **Rdzeń:** ARM Cortex-M7 o taktowaniu do 480 MHz
* **Lokalizacja:** Lewa płytka (Main Board)

---

## Podział na Płytki

Projekt składa się z dwóch osobnych płytek drukowanych:

1. **Płytka Główna (Main Control & Driver Board - `Rover_PCB.kicad_pcb`):**
   * Zawiera mikrokontroler STM32H7,
   * Przetworniki analogowo-cyfrowe ADC `HX711` do pomiaru wagi (belek tensometrycznych),
   * Filtry i transceivery magistrali CAN (`SN65HVD230`),
   * Dwa zintegrowane sterowniki mostkowe H dla silników szczotkowych prądu stałego (`VNH5019A-E`),
   * Główne stabilizatory napięć (`AMS1117-3.3` o wydajności do 800 mA dla logiki, `LM2596S-5` o wydajności do 3 A dla linii 5V oraz `LM2596S-12` o wydajności do 3 A dla linii 12V),
   * Porty wejściowe zasilania wysokoprądowego (`XT90` 24V i 48V) oraz bezpośrednie złącza interfejsowe kontrolera (USB-C, RJ45),
   * **Układy MOSFET (low-side):** 10 kluczy MOSFET (2x IRLZ44N w obudowie TO-220 oraz 8x AO3400A w obudowie SOT-23) służących do kluczowania masy dla oświetlenia zewnętrznego, diody statusowej RGY oraz buzzera.

2. **Płytka Zasilania i Interfejsów (Power Distribution & Interface Board - `Rover_PCB _Power.kicad_pcb`):**
   * Działa jako panel dystrybucji zasilania i hub połączeń interfejsów,
   * Posiada złącza silników, serwomechanizmów, interfejsów komunikacyjnych (Magistrala CAN, UART), czujników temperatury (NTC) oraz wejścia belek tensometrycznych (Load Cells),
   * Dystrybuuje zasilanie przez złącza wysokoprądowe `XT60` i `XT30` o napięciach: 12V, 24V oraz 48V,
   * Zawiera diody LED oświetlenia zewnętrznego, wielokolorową diodę statusową RGY, buzzer oraz fizyczne porty do programowania (ST-LINK) i przycisk awaryjny (Safety Button).

### Połączenia Międzypłytkowe (Inter-board Connections)
Tablice łączą się ze sobą bezpośrednio za pomocą czterech mostków kablowych:
* **ATX (J45 na Głównej $\leftrightarrow$ J48 na Zasilania):** Złącze Molex Mini-Fit Jr 24-pin przesyłające główne szyny zasilające (3.3V, 5V, 12V, 24V, 48V, GND), wyjścia prądowe sterowników silników (`OUTA1`/`OUTB1`, `OUTA2`/`OUTB2`) oraz linie sterowania oświetleniem przednim LED (`PC6_OUT` i `PB9_OUT`).
* **FFC 30-pin (J46 na Głównej $\leftrightarrow$ J49 na Zasilania):** Przesyłanie sygnałów logicznych (sygnały PWM serw, interfejsy UART, linie CAN, sygnały pomiarowe z HX711) oraz linii sterowania tylnym oświetleniem LED, diodą statusową i buzzerem (`PB0_OUT`, `PB1_OUT`, `PB5_OUT`, `PB8_OUT`, `PC7_OUT`, `PC8_OUT`, `PC9_OUT`, `PC10_OUT`).
* **FFC 20-pin (J47 na Głównej $\leftrightarrow$ J50 na Zasilania):** Przesyłanie sygnałów z pozostałych belek tensometrycznych oraz linii pomiarowych czujników temperatury NTC.
* **FFC 6-pin (J72 na Głównej $\leftrightarrow$ J71 na Zasilania):** Przedłużenie złącza programatora ST-LINK (SWD) na płytkę zasilania, co umożliwia debugowanie i programowanie urządzenia poprzez wyprowadzone tam złącze `J37` bez konieczności rozpinania konstrukcji.

---

## Specyfikacja Złączy

Poniższa lista opisuje przeznaczenie wszystkich złączy na obu płytkach oraz sposób sterowania powiązanymi urządzeniami.

### 1. Płytka Główna (Sterowanie i Sterowniki - `Rover_PCB.kicad_pcb`)

| Złącze | Nazwa w schemacie | Typ Złącza | Przeznaczenie |
| :--- | :--- | :--- | :--- |
| **J1** | `XT90_24V_IN` | XT90 (Męskie, kątowe) | Główne wejście zasilania 24V z akumulatora (zabezpieczone bezpiecznikiem F1 - 15A). Zasila regulatory step-down (5V, 12V) oraz mostki silników VNH5019A-E. |
| **J35** | `XT90_48V_IN` | XT90 (Męskie, kątowe) | Wejście wysokiego napięcia zasilania 48V do celów dystrybucyjnych (zabezpieczone bezpiecznikiem F2 - 1A na odczepie pomiarowym). |
| **J43** | `XT90_24V_OUT` | XT90 (Męskie, kątowe) | Wyjście zasilania 24V (przelotka bezpośrednia z J1). |
| **J44** | `XT90_48V_OUT` | XT90 (Męskie, kątowe) | Wyjście zasilania 48V (przelotka bezpośrednia z J35). |
| **J19** | `RJ45` | RJ45 | Złącze interfejsu sieciowego. **Uwaga:** Piny 1 i 2 są połączone z wyjściami mostka H Brushed_Motor2 (`OUTA2`/`OUTB2`), a piny 3 i 4 z zasilaniem 5V/GND, co umożliwia sterowanie zewnętrznym silnikiem lub modułem za pomocą kabla Ethernet. |
| **J38** | `USB_C_Receptacle` | USB-C 16-pin | Port USB-C podłączony bezpośrednio do linii D+/D- mikrokontrolera (komunikacja USB 2.0). |
| **J45** | `ATX` | Molex Mini-Fit Jr 2x12 | Interfejs zasilania, mostków silnikowych oraz przedniego oświetlenia, łączący z płytką zasilania. |
| **J46** | `FFC_30` | FFC Socket 30-pin (1.0mm) | Port taśmy sygnałowej i oświetlenia tylnego/statusowego, łączący z płytką zasilania. |
| **J47** | `FFC_20` | FFC Socket 20-pin (1.0mm) | Port taśmy sygnałowej dla belek tensometrycznych i czujników NTC, łączący z płytką zasilania. |
| **J72** | `FFC_6` | FFC Socket 6-pin (1.0mm) | Port taśmy interfejsu SWD (ST-LINK), łączący z płytką zasilania. |
| **J75** | `TEST_PAD 5V` | Pad testowy (1.5mm) | Punkt testowy napięcia 5V na płytce głównej. |
| **J76** | `TEST_PAD GND` | Pad testowy (1.5mm) | Punkt testowy masy (GND) na płytce głównej. |
| **J77** | `TEST_PAD 24V` | Pad testowy (1.5mm) | Punkt testowy napięcia 24V na płytce głównej. |
| **J78** | `TEST_PAD 3.3V` | Pad testowy (1.5mm) | Punkt testowy napięcia 3.3V na płytce głównej. |
| **J79** | `TEST_PAD 12V` | Pad testowy (1.5mm) | Punkt testowy napięcia 12V na płytce głównej. |

### 2. Płytka Zasilania (Złącza Wykonawcze i Dystrybucja Zasilania - `Rover_PCB _Power.kicad_pcb`)

#### Złącza Połączeniowe z Płytką Główną:
* **J48:** ATX (Molex Mini-Fit 24-pin) - interfejs szyn zasilających i wyjść silników szczotkowych/oświetlenia przedniego
* **J49:** FFC 30-pin - główna taśma sygnałowa (sterowanie serw, UART, CAN, ADCs, oświetlenie tylne/statusowe)
* **J50:** FFC 20-pin - pomocnicza taśma sygnałowa (belki tensometryczne, sensory NTC)
* **J71:** FFC 6-pin - taśma interfejsu programowania SWD (ST-LINK)

#### Port Programatora i Debuggera:
* **J37 (`STLINK`):** Złącze kołkowe 1x05 na płytce zasilania służące do bezpośredniego podpięcia programatora SWD bez konieczności rozkręcania obudowy łazika. Sygnały: `+3.3V`, `GND`, `SWDIO`, `SWCLK`, `NRST`.

#### Sterowanie Silnikami Szczotkowymi (DC Brushed Motors):
Oba złącza silników są zasilane za pośrednictwem złącza ATX z lewej płytki, gdzie zainstalowane są dedykowane układy H-bridge **VNH5019A-E** (obsługujące prąd ciągły do 12A/szczytowo 30A).
* **J2 (`Brushed_Motor1`):** Podłączenie silnika DC 1 (Linie `/OUTA1` i `/OUTB1` ze sterownika U5).
* **J3 (`Brushed_Motor2`):** Podłączenie silnika DC 2 (Linie `/OUTA2` i `/OUTB2` ze sterownika U6).

#### Serwomechanizmy (Servo Outputs):
Złącza 3-pinowe (GND, zasilanie +5V z LM2596, sygnał PWM) służące do pozycjonowania ramion, chwytaków bądź kamer. Sterowanie odbywa się bezpośrednio za pomocą wbudowanych w STM32 układów zegarowych (Timer PWM):
* **J20 (`Servo1`):** Sterowanie PWM z pinu **`PE9`** (TIM1_CH1)
* **J21 (`Servo2`):** Sterowanie PWM z pinu **`PE11`** (TIM1_CH2)
* **J22 (`Servo3`):** Sterowanie PWM z pinu **`PE13`** (TIM1_CH3)
* **J23 (`Servo4`):** Sterowanie PWM z pinu **`PE14`** (TIM1_CH4)
* **J24 (`Servo5`):** Sterowanie PWM z pinu **`PD14`** (TIM4_CH3)
* **J25 (`Servo6`):** Sterowanie PWM z pinu **`PD15`** (TIM4_CH4)
* **J26 (`Servo7`):** Sterowanie PWM z pinu **`PA6`** (TIM3_CH1)
* **J27 (`Servo8`):** Sterowanie PWM z pinu **`PA7`** (TIM3_CH2)

#### Magistrala CAN (CAN Bus Distribution):
Na płytce zintegrowano magistralę CAN, co pozwala na łączenie urządzeń zewnętrznych (np. sterowników silników BLDC, enkoderów, sensorów). Wszystkie te złącza dzielą wspólne linie `/CANH` i `/CANL` obsługiwane przez transceivery `SN65HVD230` z lewej płytki.
Złącza są typu JST EH 4-pin (GND, CANH, CANL, zasilanie) o różnych napięciach na pinie zasilającym:
* **J11 - J15 (`CAN_S1 - CAN_S5`):** Linia zasilająca podłączona do napięcia **`+12V`**.
* **J16 - J17 (`CAN_S6 - CAN7`):** Linia zasilająca podłączona do napięcia **`+24V`**.
* **J18 (`CAN8`):** Linia zasilająca podłączona do napięcia **`+48V`** (dla najbardziej obciążonych/zewnętrznych modułów).

#### Złącza Komunikacji UART:
* **J28 (`UART1`):** Podłączone do portu UART1 mikrokontrolera (linie `/TX1` i `/RX1`, zasilanie `+5V`, `GND`).
* **J29 (`UART2`):** Podłączone do portu UART2 mikrokontrolera (linie `/TX2` i `/RX2`, zasilanie `+5V`, `GND`).

#### Czujniki Belki Tensometrycznej (Load Cells):
Układ obsługuje do 4 czujników nacisku/wagi. Każde złącze to standardowe wyprowadzenie 4-pinowe (zasilanie analogowe AVDD, różnicowe wejścia sygnałowe INA+/INA- oraz GND). Pomiary są filtrowane i cyfryzowane przez przetworniki HX711 (U9-U12) znajdujące się na lewej płytce.
* **J39 (`Load_Cell1`):** Podłączone do przetwornika `U9` (linie `/AVDD`, `/INA+`, `/INA-` przesyłane przez FFC 30-pin).
* **J40 (`Load_Cell2`):** Podłączone do przetwornika `U10` (linie `/AVDD1`, `/INA1+`, `/INA1-` przesyłane przez FFC 30-pin).
* **J41 (`Load_Cell3`):** Podłączone do przetwornika `U11` (linie `/AVDD2`, `/INA2+`, `/INA2-` przesyłane przez FFC 20-pin).
* **J42 (`Load_Cell4`):** Podłączone do przetwornika `U12` (linie `/AVDD3`, `/INA3+`, `/INA3-` przesyłane przez FFC 20-pin).

#### Czujniki Temperatury NTC:
Służą do monitorowania temperatury krytycznych elementów konstrukcji (akumulatory, silniki). Termistory NTC tworzą dzielnik napięcia (zasilanie +3.3V, wyjście na ADC i GND). Pomiar odbywa się bezpośrednio za pomocą kanałów przetwornika ADC mikrokontrolera STM32:
* **J51 (`NTC1`):** Podłączony do pinu analogowego **`PC0`** (przez FFC)
* **J52 (`NTC2`):** Podłączony do pinu analogowego **`PC1`** (przez FFC)
* **J53 (`NTC3`):** Podłączony do pinu analogowego **`PC2`** (przez FFC)
* **J54 (`NTC4`):** Podłączony do pinu analogowego **`PC3`** (przez FFC)

#### Diody Sygnalizacyjne LED (Oświetlenie / Status):
Wszystkie diody LED znajdują się na płytce zasilania. Są zasilane z napięcia +12V (oświetlenie zewnętrzne) lub +24V (dioda statusowa J10) i sterowane od strony masy (low-side) za pomocą kluczy MOSFET typu N zlokalizowanych na płytce głównej:

* **Oświetlenie Przednie (Klucze MOSFET THT - IRLZ44N w obudowie TO-220):**
  * **J8 (`LED_F_R` - Przód Prawy):** Sterowane z pinu **`PB9`** (klucz Q5, sygnał `PB9_OUT`)
  * **J9 (`LED_F_L` - Przód Lewy):** Sterowane z pinu **`PC6`** (klucz Q6, sygnał `PC6_OUT`)

* **Oświetlenie Tylne (Klucze MOSFET SMD - AO3400A w obudowie SOT-23):**
  * **J4 (`LED_R_LL` - Tylny Lewy Zewnętrzny):** Sterowane z pinu **`PB0`** (klucz Q15, sygnał `PB0_OUT`)
  * **J5 (`LED_R_LR` - Tylny Lewy Wewnętrzny):** Sterowane z pinu **`PB1`** (klucz Q14, sygnał `PB1_OUT`)
  * **J6 (`LED_R_RR` - Tylny Prawy Zewnętrzny):** Sterowane z pinu **`PB5`** (klucz Q13, sygnał `PB5_OUT`)
  * **J7 (`LED_R_RL` - Tylny Prawy Wewnętrzny):** Sterowane z pinu **`PB8`** (klucz Q12, sygnał `PB8_OUT`)

* **Sygnalizacja Systemowa (Klucze MOSFET SMD - AO3400A w obudowie SOT-23):**
  * **J10 (`LED_RGY_B` - Trójkolorowa dioda statusu RGY + Buzzer):** Złącze zasilane z napięcia +24V, sterowane z czterech pinów:
    * Pin **`PC7`** (kolor czerwony LED - klucz Q11, sygnał `PC7_OUT`)
    * Pin **`PC8`** (kolor zielony LED - klucz Q16, sygnał `PC8_OUT`)
    * Pin **`PC9`** (kolor żółty LED - klucz Q17, sygnał `PC9_OUT`)
    * Pin **`PC10`** (Buzzer - klucz Q18, sygnał `PC10_OUT`)

#### Dystrybucja Zasilania Zewnętrznego (Power Distribution Ports):
Wyjścia zasilania służą do zasilania urządzeń o dużym poborze prądu.
* **Szyna 24V:**
  * `J55`, `J56`, `J57`, `J58` (złącza XT60)
  * `J59`, `J60`, `J61`, `J62` (złącza XT30)
* **Szyna 48V:**
  * `J63`, `J64` (złącza XT60)
  * `J67`, `J68` (złącza XT30)
* **Szyna 12V** (zasilana ze stabilizatora LM2596S-12):
  * `J65`, `J66` (złącza XT60)
  * `J69`, `J70` (złącza XT30)
* **Złącza Śrubowe (Terminal Blocks):**
  Umożliwiają bezpośrednie wyprowadzenie stałego napięcia (VOUT i GND) dla mniejszych odbiorników:
  * `J30` -> 3.3V
  * `J31` -> 5.0V (ze stabilizatora LM2596S-5)
  * `J32` -> 12.0V (ze stabilizatora LM2596S-12)
  * `J33` -> 24.0V (bezpośrednio z XT90_24V_IN)
  * `J34` -> 48.0V (bezpośrednio z XT90_48V_IN)

#### Monitoring Akumulatora (Battery Measurement):
* **J73 (`BAT1`):** Podłączenie pojedynczej celi / baterii do monitorowania napięcia.
* **J74 (`BAT_TOTAL`):** Podłączenie pełnego pakietu do monitorowania sumarycznego napięcia.
* **J36 (`Safety_Button`):** Złącze pod przycisk awaryjny (np. grzybek E-STOP), podłączone do pinu **`PE15`** mikrokontrolera STM32.

---

## Parametry Fizyczne PCB i Klasy Połączeń

Płytki w projekcie zostały zaprojektowane z określonym stackupem i regułami szerokości połączeń w programie KiCad.

### 1. Ułożenie Warstw (Stackup) dla Płytki Głównej
Płytka Główna (`Rover_PCB.kicad_pcb`) jest płytką **4-warstwową** o grubości laminatu **1.6 mm** ze standardową grubością miedzi **35 µm (1 oz)** na wszystkich warstwach:
* **F.Cu (Top)** — Zewnętrzna górna warstwa sygnałowa/mieszana.
* **In1.Cu (Internal 1)** — Wewnętrzna warstwa sygnałowo-zasilająca.
* **In2.Cu (Internal 2)** — Wewnętrzna dedykowana warstwa zasilania (szyna power).
* **B.Cu (Bottom)** — Zewnętrzna dolna warstwa sygnałowa/mieszana.

Dielektryk między warstwami zewnętrznymi a wewnętrznymi stanowi prepreg FR4 o grubości **0.1 mm**, a rdzeń wewnętrzny (core) ma grubość **1.24 mm**.

### 2. Klasy Netów (Net Classes) i Trasowanie
W projekcie skonfigurowano dedykowane klasy połączeń, dopasowane do obciążalności prądowej oraz typu sygnałów:

| Klasa Połączeń | Szerokość ścieżki | Odstęp (Clearance) | Średnica przelotki | Wiercenie przelotki | Zastosowanie / Przypisane Połączenia |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **`Default`** | **0.20 mm** | 0.20 mm | 0.60 mm | 0.30 mm | Sygnały logiczne mikrokontrolera, GPIO, szyna `/+3.3V` |
| **`CAN`** | **0.25 mm** | 0.20 mm | 0.60 mm | 0.30 mm | Różnicowe linie magistrali CAN (`/CANH`, `/CANL`, `/CANH_ETH`, `/CANL_ETH`) |
| **`GND`** | **0.50 mm** | 0.20 mm | 0.80 mm | 0.40 mm | Masa główna układu (`/GND`) |
| **`Power10A`** | **4.00 mm** | 0.30 mm | 1.50 mm | 0.80 mm | Główna szyna wysokoprądowa zasilania `/+48V` |
| **`Power5A`** | **2.00 mm** | 0.30 mm | 1.20 mm | 0.60 mm | Szyny zasilania `/+12V` oraz `/+24V` |
| **`Power1A`** | **0.50 mm** | 0.20 mm | 0.80 mm | 0.40 mm | Szyna zasilania logiki `/+5V` |
| **`USB_FS`** | **0.20 mm** | 0.15 mm | 0.60 mm | 0.30 mm | Sygnały różnicowe portu USB-C (`/D-`, `/D+`) |

### 3. Niestandardowe Szerokości Ścieżek (Ręczne/Custom)
W celu optymalizacji trasowania w gęstych obszarach lub specjalnych wyprowadzeń zdefiniowano i użyto:
* **0.10 mm**: użyte do wyprowadzenia linii portu szeregowego **`/RX2`** na warstwie górnej (do MCU).
* **1.00 mm**: użyte w krytycznych punktach zasilania oraz masy.
* **1.1018 mm** oraz **1.5998 mm**: lokalne przewężenia (neckdown) na linii zasilania **`/+24V`**.

*Uwaga: Płytka Zasilania (`Rover_PCB _Power.kicad_pcb`) wykorzystuje standardową szerokość ścieżek **0.20 mm** dla wszystkich sygnałów, a główne zasilania i masa są tam dystrybuowane za pomocą rozległych wylewek miedzianych (zones).*

---

## Wykaz Części (BOM) do Złożenia Płytki

Poniższa tabela zawiera kompletny wykaz wszystkich elementów niezbędnych do zmontowania płytek (Płytki Głównej oraz Płytki Zasilania).
Każdy element został powiązany z konkretnym, pasującym footprintem w programie KiCad, co gwarantuje poprawność montażu po zakupie.

| Lp. | Oznaczenia na schemacie (Ref) | Wartość/Nazwa | Obudowa / Nazwa Footprintu (KiCad) | Sugerowany Numer Części (MPN) | Opis / Uwagi | Ilość |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | D4, D5 | D_Zener | `DO-201 (THT, raster 12.7 mm)` | **1N5349BG** | Dioda Zenera 12V, 5W | 2 |
| 2 | D1, D2 | 1N5822 | `DO-201AD (THT, raster 15.24 mm)` | **1N5822-E3/54** | Dioda Schottky 40V, 3A (THT) | 2 |
| 3 | U2 | AMS1117-3.3 | `SOT-223-3` | **AMS1117-3.3** | Stabilizator liniowy LDO 3.3V, 1A | 1 |
| 4 | Q11, Q12, Q13, Q14, Q15, Q16, Q17, Q18 | AO3400A | `SOT-23 (SMD)` | **AO3400A** | Tranzystor N-MOSFET (30V, 5.7A, Rds=19mOhm) | 8 |
| 5 | J59, J60, J61, J62, J67, J68, J69, J70 | XT30_12V_OUT | `Amass XT30U-F (pionowe żeńskie PCB)` | **Amass XT30U-F** | Złącze wysokoprądowe XT30 żeńskie pionowe na płytkę | 8 |
| 6 | J55, J56, J57, J58, J63, J64, J65, J66 | XT60_24V_OUT | `Amass XT60-M (pionowe męskie PCB)` | **Amass XT60-M** | Złącze wysokoprądowe XT60 męskie pionowe na płytkę | 8 |
| 7 | J1, J35, J43, J44 | XT90_48V_OUT | `Amass XT90PW-M (kątowe męskie PCB)` | **Amass XT90PW-M** | Wysokoprądowe złącze XT90 męskie kątowe na płytkę | 4 |
| 8 | J38 | USB_C_Receptacle_USB2.0_16P | `USB-C 16-pin GCT USB4105 (poziome)` | **GCT USB4105-GF-A** | Gniazdo USB 2.0 typu C, 16-pin, montaż mieszany SMT+THT | 1 |
| 9 | U9, U10, U11, U12 | HX711 | `SOIC-16 (szerokość 3.9 mm, raster 1.27 mm)` | **HX711** | 24-bitowy przetwornik ADC dla belek tensometrycznych | 4 |
| 10 | Q5, Q6 | IRLZ44N | `TO-220-3 (pionowy)` | **IRLZ44NPBF** | Tranzystor N-MOSFET logic-level (55V, 47A, Rds=22mOhm) | 2 |
| 11 | J2, J3, J4, J5, J6, J7, J8, J9, J36, J73, J74 | LED_R_RL | `JST EH 2-pin vertical (raster 2.50 mm)` | **JST B2B-EH-A(LF)(SN)** | Złącze JST serii EH, 2-pinowe pionowe męskie | 11 |
| 12 | J51, J52, J53, J54 | NTC1 | `JST EH 3-pin vertical (raster 2.50 mm)` | **JST B3B-EH-A(LF)(SN)** | Złącze JST serii EH, 3-pinowe pionowe męskie | 4 |
| 13 | J11, J12, J13, J14, J15, J16, J17, J18, J28, J29, J39, J40, J41, J42 | CAN7 | `JST EH 4-pin vertical (raster 2.50 mm)` | **JST B4B-EH-A(LF)(SN)** | Złącze JST serii EH, 4-pinowe pionowe męskie | 14 |
| 14 | J10 | LED_RGY_B | `JST EH 5-pin vertical (raster 2.50 mm)` | **JST B5B-EH-A(LF)(SN)** | Złącze JST serii EH, 5-pinowe pionowe męskie | 1 |
| 15 | U4 | LM2596S-12 | `TO-263-5 (D2PAK)` | **LM2596S-12/NOPB** | Przetwornica obniżająca step-down impulsowa 12V, 3A | 1 |
| 16 | U3 | LM2596S-5 | `TO-263-5 (D2PAK)` | **LM2596S-5.0/NOPB** | Przetwornica obniżająca step-down impulsowa 5V, 3A | 1 |
| 17 | F1 | 15A | `Bezpiecznik samochodowy Mini Blade (lutowany w płytkę)` | **Littelfuse 0297015.WXNV + uchwyt Littelfuse 01530007Z** | Bezpiecznik Mini Blade 15A z uchwytem lutowanym | 1 |
| 18 | F2 | 1A | `Littelfuse 395 Series (raster 5.08 mm)` | **Littelfuse 39511000000** | Bezpiecznik miniaturowy TR5 1A, zwłoczny | 1 |
| 19 | Taśma-FFC-6 | Taśma FFC 6-pin | `Taśma FFC 6-pin (raster 1.0 mm, dł. 152 mm, typ A)` | **Molex 15167-0211 (lub zamiennik 6-pin, raster 1.0mm)** | Elastyczny przewód płaski FFC łączący gniazda J71 i J72 (linie SWD ST-Link) | 1 |
| 20 | Taśma-FFC-20 | Taśma FFC 20-pin | `Taśma FFC 20-pin (raster 1.0 mm, dł. 152 mm, typ A)` | **Molex 15167-0355 (lub zamiennik 20-pin, raster 1.0mm)** | Elastyczny przewód płaski FFC łączący gniazda J47 i J50 | 1 |
| 21 | Taśma-FFC-30 | Taśma FFC 30-pin | `Taśma FFC 30-pin (raster 1.0 mm, dł. 152 mm, typ A)` | **Molex 15167-0487 (lub zamiennik 30-pin, raster 1.0mm)** | Elastyczny przewód płaski FFC łączący gniazda J46 i J49 | 1 |
| 22 | J71, J72 | FFC | `FFC/FPC 6-pin, raster 1.00 mm (poziome)` | **Molex 200528-0060** | Gniazdo taśmy elastycznej FFC 6-pin, poziome SMT | 2 |
| 23 | J47, J50 | FFC | `FFC/FPC 20-pin, raster 1.00 mm (poziome)` | **Molex 200528-0200** | Gniazdo taśmy elastycznej FFC 20-pin, poziome SMT | 2 |
| 24 | J46, J49 | Conn_01x30_Socket | `FFC/FPC 30-pin, raster 1.00 mm (poziome)` | **Molex 200528-0300** | Gniazdo taśmy elastycznej FFC 30-pin, poziome SMT | 2 |
| 25 | J45, J48 | ATX | `Molex Mini-Fit Jr 5566-24A (2x12, raster 4.2 mm)` | **Molex 39-28-1243 (lub 39-28-8240)** | Złącze ATX 24-pin pionowe męskie | 2 |
| 26 | L1, L2 | 30u | `MS42 (SMD, Neosid Ms42)` | **Neosid MS42 30uH** | Dławik/cewka indukcyjna SMD 30µH | 2 |
| 27 | J19 | RJ45 | `RJ45 Jack Ninigi GE` | **Ninigi RJ45-GE (lub Molex 95501-2881)** | Gniazdo Ethernet RJ45 8p8c THT | 1 |
| 28 | J37 | STLINK | `PinSocket 1x05 vertical, raster 2.54 mm` | **PPTC051LFBN-RC (Sullins)** | Gniazdo kołkowe (żeńskie) 1x5, raster 2.54mm | 1 |
| 29 | SW1 | SW_Push | `Tactile Switch 6x6 mm, wys. 4.3 mm (THT)` | **PTS645TL43-2 LFS** | Przycisk monostabilny (tact switch) THT | 1 |
| 30 | C5, C6 | 2.2u | `Radialny THT (Ø4.0 mm, raster 1.50 mm)` | **Panasonic ECE-A1HN2R2UB** | Kondensator elektrolityczny bipolarny 2.2µF, 50V | 2 |
| 31 | C1, C2, C3, C4 | 220u | `Radialny THT (Ø5.0 mm, raster 2.50 mm)` | **Panasonic EEU-FR1E221** | Kondensator elektrolityczny niskoimpedancyjny 220µF, 25V | 4 |
| 32 | J30, J31, J32, J33, J34 | 48V | `Phoenix MKDS-1.5-2 (raster 5.08 mm)` | **Phoenix Contact 1715721 (MKDS 1,5/2-5,08)** | Złącze śrubowe (terminal block) 2-pin do druku | 5 |
| 33 | J20, J21, J22, J23, J24, J25, J26, J27 | Servo8 | `PinHeader 1x03 vertical, raster 2.54 mm` | **Pin Header 1x03 vertical (2.54mm pitch)** | Listwa kołkowa (goldpin) 1x3, raster 2.54mm | 8 |
| 34 | R1, R4, R5, R6, R7, R8, R9, R10, R11, R12, R13, R14, R15, R16, R22 | 10k | `SMD 0603 (1608 Metric)` | **RC0603FR-0710KL** | Rezystor SMD 0603, tolerancja 1%, wartość 10k | 15 |
| 35 | R17-Terminator1 | 120 | `SMD 0603 (1608 Metric)` | **RC0603FR-07120RL** | Rezystor SMD 0603, tolerancja 1%, wartość 120 | 1 |
| 36 | R17 | 330 | `SMD 0603 (1608 Metric)` | **RC0603FR-07330RL** | Rezystor SMD 0603, tolerancja 1%, wartość 330 | 1 |
| 37 | R19, R21 | 3.9k | `SMD 0603 (1608 Metric)` | **RC0603FR-073K9L** | Rezystor SMD 0603, tolerancja 1%, wartość 3.9k | 2 |
| 38 | R18, R20 | 47k | `SMD 0603 (1608 Metric)` | **RC0603FR-0747KL** | Rezystor SMD 0603, tolerancja 1%, wartość 47k | 2 |
| 39 | R2, R3 | 5.1k | `SMD 0603 (1608 Metric)` | **RC0603FR-075K1L** | Rezystor SMD 0603, tolerancja 1%, wartość 5.1k | 2 |
| 40 | D6 | SD36_SOD323 | `SOD-323 (SMD)` | **SD36-01FTG** | Dioda TVS jedno-kierunkowa 36V, obudowa SOD-323 | 1 |
| 41 | U7, U8 | SN65HVD230 | `SOIC-8 (szerokość 3.9 mm, raster 1.27 mm)` | **SN65HVD230D** | Transceiver CAN, zasilany napięciem 3.3V | 2 |
| 42 | U1 | STM32H743VITx | `LQFP-100 (14x14 mm, raster 0.5 mm)` | **STM32H743VIT6** | Mikrokontroler ARM Cortex-M7, 2MB Flash, 1MB RAM | 1 |
| 43 | U5, U6 | VNH5019A-E | `MultiPowerSO-30 (STMicroelectronics)` | **VNH5019ATR-E** | Mostek H - sterownik silników DC szczotkowych (30A/41V) | 2 |
| 44 | D3 | LED | `LED 5.0 mm (THT)` | **WP7113ID** | Dioda LED czerwona 5mm (THT) | 1 |
| 45 | C7, C8 | 10n | `SMD 0603 (1608 Metric)` | **Yageo CC0603KRX7R9BB103** | Kondensator ceramiczny MLCC SMD 0603 X7R 10nF 50V | 2 |
| 46 | C9, C10 | 18pF | `SMD 0603 (1608 Metric)` | **CC0603JRNPO9BN180** | Kondensator ceramiczny MLCC SMD 0603 NP0 18pF 50V (dla rezonatora Y2) | 2 |
| 47 | Y2 | 8MHz | `Crystal:Crystal_SMD_3225-4Pin_3.2x2.5mm_HandSoldering` | **NX3225GD-8.000M-STD-CRA-3** | Rezonator kwarcowy SMD 3.2x2.5mm (dla STM32H7) | 1 |
| 48 | J75, J76, J77, J78, J79 | TEST_PAD | `TestPoint:TestPoint_Pad_D1.5mm` | **Pady na PCB** | Punkty testowe zasilania i masy (5V, GND, 24V, 3.3V, 12V) - nie wymagają dodatkowego montażu | 5 |
