# The Complete Guide to Vital Signs Detection using IWR1642 / IWR1443

## 1. How It Actually Works: The Science & Theory
FMCW (Frequency Modulated Continuous Wave) radar measures distance by transmitting a signal that linearly increases in frequency (a "chirp") and mixing it with the signal reflected off an object.

To measure vital signs, the radar does not track large movements (like walking). Instead, it tracks the **phase change** of the reflected signal on a completely stationary target. 

1. **Range FFT (Isolating the Chest):** The radar takes a Fast Fourier Transform (FFT) across a single chirp to find where the subject is standing. The chest creates a massive reflection peak in a specific "range bin".
2. **Phase Extraction (Micro-Doppler):** Once the chest's range bin is isolated, the radar analyzes the **phase ($\phi$)** of the signal at that specific distance across hundreds of consecutive frames. 
3. **Displacement Measurement:** As the lungs expand (breathing: 1–12 mm displacement) and the heart pumps (cardiac: ~0.1 mm displacement), the distance to the radar changes minutely. This causes the phase of the reflected radar signal to shift continuously.
4. **Filtering:** The extracted phase signal is a messy combination of breathing and heartbeat. The DSP applies bandpass filters:
   - **0.1 to 0.5 Hz (6 - 30 BPM):** Isolates the Respiration rate.
   - **0.8 to 2.0 Hz (48 - 120 BPM):** Isolates the Heart rate.
5. **Spectral FFT:** A final FFT on these filtered waveforms identifies the dominant frequencies, outputting the final Breaths Per Minute and Beats Per Minute.

---

## 2. Method 1: The Pre-Built TI Vital Signs Lab (Real-Time)
This is the fastest, easiest way to run the project. Texas Instruments provides pre-compiled firmware that processes the FFTs and filters directly on the IWR1642's C674x DSP core, outputting the live data to a computer GUI.

### 2.1 Software & Resources to Install
1. **TI mmWave SDK:** Download and install version `2.1.0.4` (Standard for IWR1642).
2. **TI UniFlash:** Download and install the latest version from ti.com. This is used to flash `.bin` files to the radar's ROM.
3. **mmWave Industrial Toolbox:** 
   - Go to [TI Resource Explorer](https://dev.ti.com/tirex/explore).
   - Navigate to `Software -> mmWave Sensors -> Industrial Toolbox`.
   - Download the toolbox to your local machine (usually extracted to `C:\ti\mmwave_industrial_toolbox_<version>`).
4. **MATLAB Runtime Engine:** The TI Vital Signs GUI requires a specific runtime to open. Usually, this is **Version 9.2 (R2017a, 32-bit or 64-bit depending on your toolbox version)**. The toolbox documentation will specify the exact version needed.

### 2.2 Boot Modes (SOP Jumper Configuration)
The IWR1642BOOST has three SOP (Sense-on-Power) header pins. By placing jumpers on these pins, you tell the board how to boot.

| Boot Mode | SOP2 | SOP1 | SOP0 | When to use this? |
| :--- | :--- | :--- | :--- | :--- |
| **Flashing Mode (SOP 5)** | ON (1) | OFF (0) | ON (1) | When using UniFlash to burn firmware. |
| **Functional Mode (SOP 4)** | OFF (0) | OFF (0) | ON (1) | When running the radar normally. |
| **Debug/Dev Mode (SOP 2)** | OFF (0) | ON (1) | ON (1) | When using DCA1000 & mmWave Studio. |

*(Note: "ON" means place the black jumper shunt over the two pins to short them. "OFF" means remove the jumper).*

### 2.3 Step 1: Flashing the Vital Signs Firmware
1. **Set to Flashing Mode:** Place jumpers on **SOP0** and **SOP2**. Remove the jumper from SOP1.
2. Connect the Micro-USB cable from the EVM's XDS110 port to your PC.
3. Plug in the 5V/3A Power Supply.
4. **Reset the Board:** Press the `NRST` (Reset) button on the board once.
5. **Open UniFlash:**
   - Select **IWR1642** as your device.
   - Click **Start**.
   - Go to the **Settings & Utilities** tab. Note your COM Port number for the *XDS110 Class Application/User UART* (check Windows Device Manager) and type it in.
   - Go to the **Program** tab. In the "Meta Image 1" section, click Browse.
   - Navigate to the Industrial Toolbox folder: `C:\ti\mmwave_industrial_toolbox_<version>\labs\vital_signs\68xx_vital_signs\prebuilt_binaries` *(Note: Ensure you grab the 16xx equivalent bin file, typically named `xwr16xx_vitalSigns_lab_mss.bin`)*.
   - Click **Load Image**. Wait for the "Programming Successful" message.
6. **Disconnect Power.**

### 2.4 Step 2: Running the Vital Signs GUI
1. **Set to Functional Mode:** Remove the jumper from **SOP2**. Keep the jumper on **SOP0** ONLY.
2. Plug the power back into the board.
3. **Reset the Board:** Press the `NRST` button once. The board is now booting from the firmware you just flashed.
4. **Launch the GUI:**
   - Navigate to `C:\ti\mmwave_industrial_toolbox_<version>\labs\vital_signs\vitalSigns_target\gui\gui_exe`.
   - Run `VitalSignsRadar_Demo.exe`.
5. **Configure the GUI:**
   - **UART Port:** Enter the COM port number for the *User/Application UART* (e.g., `COM4`).
   - **Data Port:** Enter the COM port number for the *Auxiliary Data Port* (e.g., `COM5`).
   - **Load Config:** Click "Browse" and select the `vital_signs_AWR16xx.cfg` configuration file located in the toolbox `chirp_configs` folder.
   - Click **Start**.
6. **Test the Setup:** Sit perfectly still in a chair about 0.8 meters in front of the radar. Breathe normally. Within 15 seconds, the GUI will populate with your live breathing waveform, heart waveform, and BPM numbers.

---

## 3. Method 2: Custom Raw Data Processing (DCA1000EVM)
If you are doing a thesis or building a custom AI model, you cannot use the pre-built GUI. You must capture raw ADC data and write your own MATLAB/Python code.

### 3.1 Hardware Configuration
1. Connect the IWR1642BOOST to the DCA1000EVM via the 60-pin Samtec cable.
2. Set the IWR1642 to **Development Mode (SOP 2):** Jumpers on **SOP0** and **SOP1**, remove SOP2.
3. Follow the network configuration (Static IP: `192.168.33.30`) and firewall rules detailed in the DCA1000 setup guide.

### 3.2 Capturing the Data
1. Launch **mmWave Studio**.
2. Flash the default evaluation firmware (`bss` and `mss` from the `rf_eval_firmware` folder).
3. Set your frame periodicity to **50ms (20 Frames Per Second)**. Capture at least 400 frames (20 seconds of data).
4. Arm the DCA1000 and Trigger the frame. A `.bin` file will be generated.

### 3.3 Custom Python/MATLAB DSP Pipeline
To process the `.bin` file yourself, your script must execute the following math:

1. **Data Parsing:** Read the 16-bit integers from the `.bin` file. Separate the interleaved I (In-phase) and Q (Quadrature) channels to recreate the complex signal $I + jQ$.
2. **Range Profile (1D FFT):** Apply an FFT to the fast-time samples of each chirp. Find the index `max_idx` of the highest peak. This is the human chest.
3. **Phase Extraction:** Extract the angle of the complex numbers at `max_idx` across all frames: 
   `phase = numpy.angle(radar_data[:, max_idx])`
4. **Phase Unwrapping:** Phase is bounded between $[-\pi, \pi]$. Use `numpy.unwrap(phase)` to fix phase jumps caused by deep breaths.
5. **Phase Differencing:** Subtract consecutive phase values to remove static room clutter: 
   `diff_phase = phase[1:] - phase[:-1]`
6. **Digital Filtering:** Design two Butterworth filters (IIR) in Python using `scipy.signal`. 
   - Apply a 0.1-0.5 Hz bandpass filter to `diff_phase` to get the breathing wave.
   - Apply a 0.8-2.0 Hz bandpass filter to `diff_phase` to get the heart wave.
7. **Final FFT:** Apply an FFT to both filtered waves, find the peak frequency in Hz, and multiply by 60 to calculate BPM.
