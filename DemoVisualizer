# The Complete Guide to TI mmWave Demo Visualizer

## 1. Overview
The **TI mmWave Demo Visualizer** is a web-based (or offline) graphical interface provided by Texas Instruments. It is the fundamental tool for quickly testing and visualizing radar data (Point Clouds, Range Profiles, and Doppler signatures) without needing a DCA1000EVM data capture card. 

The visualizer works by communicating with the Out-of-Box (OOB) Demo firmware flashed onto the IWR1642 / IWR1443. It sends configuration commands via UART and receives processed 3D object data back via a separate UART data port.

---

## 2. Software & Hardware Prerequisites

### Software
1. **TI mmWave SDK:** Install version `2.1.0.4` (Recommended for IWR1642/1443).
2. **TI UniFlash:** Download and install from ti.com to flash the OOB firmware.
3. **Google Chrome browser:** Required if using the web version of the visualizer.
4. **TI Cloud Agent Bridge:** A browser extension that allows Chrome to communicate with your local COM ports. Chrome will prompt you to install this automatically upon opening the Visualizer.

### Hardware
*   **Radar Sensor:** IWR1642BOOST or IWR1443BOOST.
*   **Power Supply:** 5V, 3A with a 2.1mm barrel jack.
*   **Micro-USB Cable:** Connects the XDS110 port to your PC.
*   *(Note: The DCA1000EVM is NOT used for this demo).*

---

## 3. Step 1: Flashing the Out-of-Box (OOB) Firmware
Before the Visualizer can read data, the radar must be running the correct firmware.

### Jumper Configuration (Flashing Mode)
Set the board to **Flashing Mode (SOP 5)**:
*   **SOP0:** Jumper ON (1)
*   **SOP1:** Jumper OFF (0)
*   **SOP2:** Jumper ON (1)

### Flashing Process
1. Connect the power supply and USB cable to the EVM.
2. Press the `NRST` (Reset) button on the board once.
3. Open **TI UniFlash**.
4. Select your specific board (e.g., **IWR1642**) and click **Start**.
5. Go to the **Settings & Utilities** tab. Enter the COM port for the *XDS110 Class Application/User UART* (verify in Windows Device Manager).
6. Go to the **Program** tab. 
7. In the "Meta Image 1" section, click Browse and navigate to the SDK folder:
   *   *Path:* `C:\ti\mmwave_sdk_02_01_00_04\packages\ti\demo\xwr16xx\mmw\xwr16xx_mmw_demo.bin`
8. Click **Load Image**. Wait for the console to output "Programming Successful".
9. **Disconnect the power supply.**

---

## 4. Step 2: Running the mmWave Demo Visualizer

### Jumper Configuration (Functional Mode)
Set the board to **Functional Mode (SOP 4)**:
*   **SOP0:** Jumper ON (1)
*   **SOP1:** Jumper OFF (0)
*   **SOP2:** Jumper OFF (0) *(Remove this jumper)*

1. Reconnect the power supply.
2. Press the `NRST` (Reset) button once to boot the newly flashed OOB firmware.

### Connecting to the Visualizer
1. Open Google Chrome and navigate to the [TI mmWave Demo Visualizer](https://dev.ti.com/gallery/view/mmwave/mmWave_Demo_Visualizer/ver/2.1.0/).
   *   *(Note: Ensure you select version 2.1.0 to match your SDK version. Newer visualizer versions may not support older 1642 boards).*
2. Go to the **Options > Serial Port** menu (usually found in the top-left or right corner, depending on UI updates).
3. Configure the COM Ports:
   *   **CFG_port:** Enter your *XDS110 Class Application/User UART* COM port. Set baud rate to `115200`.
   *   **DATA_port:** Enter your *XDS110 Class Auxiliary Data Port* COM port. Set baud rate to `921600`.
4. Click **Connect**. The status bar at the bottom should change to "Hardware Connected".

---

## 5. Step 3: Configuring the Radar & Visualizing Data

Once connected, you must send a `.cfg` file (Chirp Configuration) to the radar to tell it how to operate.

1. Go to the **Configure** tab in the Visualizer.
2. You can either:
   *   **Use the Sliders:** Adjust parameters like Frame Rate, Range Resolution, and Maximum Range. The UI will automatically calculate the FMCW chirp parameters.
   *   **Load a Config:** Click **Load Config From PC and Send** and select a pre-made `.cfg` file from the SDK (`C:\ti\mmwave_sdk_02_01_00_04\packages\ti\demo\xwr16xx\mmw\profiles`).
3. Click **Send Config To mmWave Device**.
4. The console will rapidly scroll as it sends the commands.
5. Switch to the **Plots** tab. 
6. You will now see real-time data:
   *   **Scatter Plot:** Shows the 2D/3D point cloud of detected objects.
   *   **Range Profile:** Shows the raw magnitude of reflections at different distances.
   *   **Doppler-Range Heatmap:** Shows the velocity of objects relative to their distance.

---

## 6. Troubleshooting Common Issues

*   **"Hardware Not Connected" / "Waiting for Data...":** 
    *   Ensure the board is in Functional Mode (SOP2 removed).
    *   Ensure you pressed the `NRST` button after changing SOP jumpers.
    *   Verify your COM ports and Baud Rates are correct.
*   **Visualizer loads but no point cloud appears:** 
    *   Ensure you actually clicked "Send Config To mmWave Device".
    *   Make sure there are moving objects in front of the radar. Static objects are often filtered out by default in the OOB demo.
*   **Firmware version mismatch error:** 
    *   The Visualizer version must match the SDK version flashed to the board. If you flashed SDK 2.1, you must use Visualizer v2.1.0, not v3.5 or v4.x.
