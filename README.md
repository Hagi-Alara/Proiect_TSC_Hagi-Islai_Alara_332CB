# InkTime - Open Source Smartwatch

Prezentul repository include documentația tehnică și fișierele de proiectare hardware (stadiul EVT - Engineering Verification Test) pentru proiectul InkTime. Acesta reprezintă un smartwatch open-source, având la bază microcontrolerul nRF52840. Proiectul acoperă întregul ciclu de dezvoltare hardware: specificații, schemă electrică, proiectare PCB și integrare mecanică.

## Descrierea Funcționalității Hardware

Sistemul este arhitecturat în jurul SoC-ului nRF52840, selectat pentru suportul Bluetooth Low Energy (BLE) și eficiența energetică ridicată.

**Module și Interfețe Integrate:**
* **Display E-Paper:** Interfațat prin protocolul SPI, garantând un consum energetic minim (curent absorbit exclusiv în timpul actualizării ecranului).
* **Unitate de Măsură Inerțială (IMU):** Utilizată pentru detectarea gesturilor. Comunicarea se realizează exclusiv prin magistrala I2C.
* **Feedback Haptic:** Motor de vibrații controlat printr-un circuit integrat (driver) dedicat, implementat pentru notificări.
* **Interfață Utilizator:** Trei butoane fizice cu acționare tactilă, configurate cu rezistențe de pull-up interne.
* **Managementul Energiei:** Sistemul este alimentat printr-un acumulator Li-Po conectat direct la pad-urile circuitului imprimat, eliminând conectorul JST pentru a respecta limitările de gabarit pe axa Z. Încărcarea se realizează prin intermediul interfeței USB-C, gestionată de un circuit integrat de management al puterii (PMIC), în timp ce nivelul de încărcare este monitorizat de un Fuel Gauge.

## Diagrama Bloc

*(Inserați aici diagrama bloc a arhitecturii hardware. Exemplu: `![Diagrama Bloc](link_catre_imagine.png)`)*

## Bill of Materials (BOM) Principal

*(Notă: Tabelul reflectă componentele majore integrate în schemă)*

| Componentă | Pachet / Capsulă | Funcție |
| :--- | :--- | :--- |
| **nRF52840-QIAA** | aQFN-73 | Microcontroler principal (SoC) + BLE |
| **BMA421** | LGA-12 | IMU (Accelerometru pentru gesturi) |
| **BQ25180** | DSBGA | PMIC (Management Încărcare Baterie) |
| **MAX17048** | TDFN-8 | Fuel Gauge (Monitorizare nivel baterie) |
| **DRV2605** | BGA-9 | Driver Haptic pentru motorul de vibrații |
| **RT6160A** | WLCSP | Regulator DC/DC (Buck-Boost pentru 3.3V) |
| **Conector Molex** | FPC 24-pin 0.5mm | Conector pentru Display E-Paper |

## Maparea Pinilor (nRF52840)

Alocarea pinilor a fost optimizată pentru a minimiza încrucișările de trasee pe cablajul imprimat.

| Pin nRF52840 | Nume Rețea | Interfață / Funcție |
| :--- | :--- | :--- |
| **P$AD10, P$K2, P$AD12** | `EPD_DC`, `EPD_CS`, `EPD_RST` | Control E-Paper (Data/Command, Chip Select, Reset) |
| **P$P13, P$A12** | `MOSI`, `SCK` | SPI Date & Clock pentru afișaj |
| **P$AD12** | `EPD_BUSY` | Status E-Paper (Indicator de ocupare) |
| **P$L1, P$M2** | `SDA`, `SCL` | I2C Date & Clock (PMIC, Fuel Gauge, IMU) |
| **P$AD8, P$AC9, P$W24** | `P0.13`, `P0.14`, `P1.02` | Butoane interfață utilizator (Up, Enter, Down) |
| **P$AD6, P$AD4, P$AD2**| `D+`, `D-`, `VBUS` | USB-C ESD Protection & VBUS Detect |
| **P$U1** | `HAPTIC_EN` | Semnal de activare Driver Haptic |
| **P$T2** | `PMIC_INT` | Întrerupere Baterie Li-Po |
| **P$N1, P$P2** | `IMU_INT1`, `IMU_INT2` | Întreruperi IMU |
| **P$J24** | `ALERT` | Alertă Fuel Gauge |
| **P$AD22, P$AA24, P$AC24** | `SWO`, `SWDCLK`, `SWDIO`| Interfață de depanare (SWD) |
| **P$AC13** | `RESET` | Semnal Hardware Reset |
| **P$Y23** | `P1.01` | Control E-Paper Drive Circuit |

## Randări 3D
<img width="1191" height="815" alt="image" src="https://github.com/user-attachments/assets/d70806fd-48fd-442e-a2dd-e098458f0f61" />

<img width="1155" height="991" alt="image" src="https://github.com/user-attachments/assets/fa92a6f4-6b4c-4cfd-a3cd-26c49ee54e4b" />
<img width="1262" height="850" alt="image" src="https://github.com/user-attachments/assets/2b0a4fa6-e957-4989-86a7-556bf81fb66e" />


## Etape de Implementare

1. **Definirea Specificațiilor și a Componentelor:** Selectarea componentelor necesare respectând cerința unui consum redus de energie (ultra-low-power).
2. **Proiectarea Schemei Electrice (Validare ERC):** Interconectarea subsistemelor de alimentare (PMIC, LDO), logică digitală (I2C, SPI) și radiofrecvență (RF).
3. **Proiectarea Cablajului Imprimat (Validare DRC):** * Amplasarea componentelor respectând constrângerile mecanice ale carcasei.
   * Rutarea traseelor de impedanță controlată pentru antena BLE, cu implementarea zonelor de restricție (Keepout) pe toate straturile.
   * Generarea planurilor de masă (Polygon Pours) pe straturile Top și Bottom.
4. **Validarea Mecanică:** Exportul ansamblului în format 3D STEP pentru verificarea toleranțelor de integrare în carcasa fizică.

## Erori de Proiectare (DRC/ERC) și Decizii de Inginerie Asumate

În urma procesului de verificare automată a regulilor de proiectare și a schemelor electrice (DRC/ERC), au fost raportate o serie de avertismente. Acestea au fost analizate și acceptate intenționat, având la bază următoarele justificări tehnice:

* **Erori ERC: "POWER pin connected to..." (Exemplu: U1 VDD connected to 3V3)**
  * *Justificare:* Aceste avertismente au fost ignorate. Cauza constă în definirea generică a pinilor de alimentare în biblioteca software (ex. "VDD"), în timp ce în schema proiectului rețeaua poartă o denumire specifică ("3V3" sau "VBAT"). Conexiunile sunt corecte din punct de vedere electric.
* **Erori ERC: "Only one pin on net" (Exemplu: SWO, SDA/1.4C, P1.15)**
  * *Justificare:* Aceste erori au fost ignorate. Avertismentele sunt generate deoarece pinii respectivi au fost rutați exclusiv către puncte de test (Test Points) pentru depanare sau au fost etichetați pentru funcționalități viitoare, nefiind conectați la un alt circuit integrat. Prezența lor este necesară pentru testarea hardware.
* **Erori DRC: "Overlap", "Solder mask", "Pad ct Top"**
  * *Justificare:* Aceste erori au fost acceptate. Având în vedere densitatea ridicată a plăcii, toleranțele măștii de lipire (Solder Mask) pentru componentele cu pas foarte mic (cum este conectorul FPC) se suprapun la nivelul software-ului CAD. Aceste deviații se încadrează în capabilitățile și toleranțele standard ale producătorului de cablaje imprimate.
* **Erori DRC: "Board Outline / Copper Clearance"**
  * *Justificare:* Aceste erori au fost acceptate. Erorile sunt provocate de componentele amplasate la limita cablajului (conectorul USB-C, contactele bateriei). Din rațiuni de asamblare mecanică, pinii de susținere și portul trebuie să se intersecteze cu conturul fizic al plăcii (Board Outline). Deși programul raportează o încălcare a regulilor de distanță, implementarea corespunde cerințelor fizice de funcționare.
