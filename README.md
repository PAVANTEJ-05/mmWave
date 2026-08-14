# IWR1642 & IWR1443 with DCA1000EVM Data Capture Setup Guide

## Table of Contents
1. [Overview](#overview)
2. [Required Hardware & Software](#required-hardware--software)
3. [Hardware Preparation & Connections](#hardware-preparation--connections)
4. [Host PC Network Configuration](#host-pc-network-configuration)
5. [Software Setup & Data Capture Workflow](#software-setup--data-capture-workflow)
6. [Official Manuals & Reference Guides](#official-manuals--reference-guides)

---

## 1. Overview
This guide provides a comprehensive, step-by-step workflow for setting up the **Texas Instruments IWR1642 or IWR1443 EVMs** (Evaluation Modules) with the **DCA1000EVM Data Capture Card**. Following these instructions will enable you to successfully capture raw ADC radar data over Ethernet for further signal processing and analysis.

---

## 2. Required Hardware & Software

### Hardware Checklist
- **Radar Sensor EVM:** IWR1642BOOST or IWR1443BOOST.
- **Data Capture Card:** DCA1000EVM.
- **Power Supplies (x2):** 5V, >2.5A (or 3A) power supplies with 2.1mm barrel jacks (one for the IWR EVM, one for the DCA1000EVM).
- **Cables:**
  - 1x 60-pin Samtec high-density ribbon cable (included with DCA1000EVM).
  - 2x Micro-USB cables.
  - 1x RJ45 Gigabit Ethernet cable.

### Software Prerequisites
- **TI mmWave Studio:** Version 2.1.1.0 (or the version matching your device's generation).
- **MATLAB Runtime Engine:** Version 8.5.1 (32-bit). 
  > **⚠️ CRITICAL:** mmWave Studio specifically requires this exact version to render the Post-Processing GUI. Newer versions will not work.
- **FTDI & XDS110 Drivers:** Generally included during the mmWave Studio and Code Composer Studio (CCS) installation process.

---

## 3. Hardware Preparation & Connections

Before powering anything on, you must correctly configure the physical jumpers and mate the boards to ensure proper communication.

### 3.1. IWR1642 / IWR1443 Jumper Configuration (SOP Mode 2)
To allow mmWave Studio to communicate with the radar chip and load firmware over SPI, the EVM must be placed in **Development Mode (SOP Mode 2)**.

Locate the SOP (Sense-on-Power) header pins on your EVM and place the jumper shunts as follows:

| Pin | State | Instruction |
| :--- | :--- | :--- |
| **SOP0** | Closed (1) | Place jumper shunt ON |
| **SOP1** | Closed (1) | Place jumper shunt ON |
| **SOP2** | Open (0) | Remove jumper shunt |

### 3.2. DCA1000EVM Switch Configuration
Locate **SW2** on the DCA1000EVM. To allow mmWave Studio to fully configure the board over Ethernet, force it into Software Configuration Mode.

| Switch | State | Instruction |
| :--- | :--- | :--- |
| **SW2.5** | `SW_CONFIG` | Push toward the side labeled "SW_CONFIG" (Pin 12 side) |

*(Leave the rest of the SW2 switches in their default positions, as `SW_CONFIG` overrides them).*

### 3.3. Cable Connections
1. **Mate the Boards:** Connect the IWR EVM to the DCA1000EVM using the 60-pin Samtec ribbon cable.
2. **USB to Radar:** Connect a Micro-USB cable from the Host PC to the **XDS110 micro-USB port** on the IWR EVM.
3. **USB to DCA1000:** Connect a Micro-USB cable from the Host PC to the **Radar FTDI micro-USB port (J1)** on the DCA1000EVM.
4. **Ethernet:** Connect the Ethernet cable from the **J6 Ethernet jack** on the DCA1000EVM directly to your Host PC.
5. **Power:** Plug the 5V/3A power supplies into the barrel jacks of **both** boards. *(Best practice: Power the DCA1000EVM first, then the IWR EVM).*

---

## 4. Host PC Network Configuration

The DCA1000EVM streams massive amounts of raw UDP data over Ethernet to a static IP address. Misconfiguration here is the most common cause of silent failures.

### Step 1: Open Network Adapters
Press `Win + R`, type `ncpa.cpl`, and hit Enter. Right-click the Ethernet adapter connected to the DCA1000EVM and select **Properties**.

### Step 2: Set Static IPv4 Address
Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**. Configure the following exact values:
- **IP address:** `192.168.33.30`
- **Subnet mask:** `255.255.255.0`
- **Default gateway:** *(Leave blank)*

### Step 3: Disable Firewall (Critical)
> **⚠️ WARNING:** You must go to Windows Defender Firewall settings and **turn off the firewall** for this specific Ethernet network. If the firewall is active, it will silently drop the incoming UDP radar data, resulting in a 0-byte capture file.

---

## 5. Software Setup & Data Capture Workflow

Launch **mmWave Studio**. Check the bottom output console to ensure there are no FTDI errors upon boot.

### 5.1. Phase 1: Connection & Firmware Loading
1. **Select Subsystem:** On the left pane, select **AR1642** or **AR1443** depending on your hardware.
2. **RS232 Connect:** 
   - Go to the *Connection* tab.
   - Select the COM port corresponding to the **XDS110 Class Application/User UART** (verify via Windows Device Manager).
   - Set Baud Rate to `115200`.
   - Click **RS232 Connect**. The console should indicate success.
3. **Load Firmware:** 
   - Browse for the **BSS FW** (Baseband) and **MSS FW** (Master). 
   - *Typical Path:* `C:\ti\mmwave_studio_<version>\rf_eval_firmware`
   - Click **Load** for BSS, wait for the success message, then click **Load** for MSS.
4. **SPI Connect & RF Power:** 
   - Click **SPI Connect**. 
   - Once the button text changes to "SPI Disconnect", click **RF Power UP**.

### 5.2. Phase 2: Sensor Configuration
Navigate through the tabs sequentially to define your radar parameters. Ensure you click **Set** after configuring each section.

1. **Static Config:** 
   - Select your Tx/Rx antennas.
   - **Lane Configuration:** Select **2 Lanes** for IWR1642, or **4 Lanes** for IWR1443. 
   - ADC config: Complex 1x or 2x, 16-bit. Click **Set**.
2. **Data Config:** Set data path to **LVDS**. Click **Set**.
3. **Sensor Config:** Define your Profile, Chirp, and Frame configurations according to your radar application needs.

### 5.3. Phase 3: DCA1000 Configuration & Arming
1. Still in the *Sensor Config* tab, locate the **DCA1000** section at the very bottom.
2. Click **Setup DCA1000**. A new popup window will open.
3. Click **Connect, Reset and Configure**. 
   - *Verification:* The output console must confirm that the FPGA version was read successfully over Ethernet.
4. Close the setup popup window and return to the main GUI.
5. In the *Sensor Config* tab, designate a file path for your capture file (e.g., `C:\ti\RawData.bin`).
6. Click **DCA1000 Arm**.
7. Click **Trigger Frame**. 

The DCA1000 will now capture the LVDS data over Ethernet and save it to your specified `.bin` file. Once the capture frames complete, click the **PostProc** button. This will launch the MATLAB runtime GUI, process the raw `.bin` file, and display your 1D/2D FFT plots.

---

## 6. Official Manuals & Reference Guides

For architecture specifics, deep-dive troubleshooting, or Lua API references, consult the official TI documents:

- **DCA1000EVM Data Capture Card User's Guide:** [Download PDF](https://www.ti.com/lit/ug/spruij4a/spruij4a.pdf)
- **IWR1642BOOST User's Guide:** [Download PDF](https://www.ti.com/lit/ug/swru521c/swru521c.pdf)
- **DCA1000 Debugging Handbook:** [Download PDF](https://dr-download-cdn.ti.com/software-development/ide-configuration-compiler-or-debugger/MD-NeRAa3SNis/03.01.01.00/DCA1000_Debugging_Handbook.pdf)
- **mmWave Studio User's Guide:** Located locally on your machine at `C:\ti\mmwave_studio_<version>\docs\mmwave_studio_user_guide.pdf`
