# FMCW Radar Vital Signs Detection Project Guide

## 1. Overview
This guide details the workflow for detecting human vital signs (breathing and heart rate) using the IWR1642/1443 EVM and DCA1000EVM. Unlike macro-movement tracking, vital signs detection relies on measuring the minute phase changes of the radar signal reflecting off the human chest.

---

## 2. Physical Setup & Environment
Because vital signs monitoring relies on measuring displacements as small as fractions of a millimeter, the physical setup must be strictly controlled.

*   **Positioning:** Mount the radar securely on a tripod or fixed stand so it is perfectly stationary. Point it directly at the subject’s chest.
*   **Distance:** Keep the subject at a close, fixed distance (typically between 0.5 meters and 1.5 meters from the sensor).
*   **Subject State:** The subject must remain completely still and breathe normally. Any macro-movement (such as shifting, talking, or moving arms) will completely wash out the micro-Doppler signals of the heart and lungs.

---

## 3. Sensor Configuration (Chirp Design)
To capture heart and breath rates, your chirp configuration in mmWave Studio must satisfy the Nyquist sampling theorem for the specific biological frequencies you want to measure. 

*   **Breathing Rate Frequency Range:** 0.1 Hz to 0.5 Hz (6 to 30 breaths per minute).
*   **Heart Rate Frequency Range:** 0.8 Hz to 2.0 Hz (48 to 120 beats per minute).

To detect a 2.0 Hz signal, your frame rate must be at least 4 Hz. For high-resolution phase extraction, a frame rate of **20 fps** (50 ms periodicity) is recommended.

**Recommended mmWave Studio Settings:**
*   **Antennas:** 1 Tx and 4 Rx.
*   **Frame Periodicity:** 50 ms.
*   **Number of Frames:** 200 to 400 frames (yielding 10 to 20 seconds of data to generate a clean FFT output for slow breathing frequencies).

---

## 4. Signal Processing Pipeline (Offline DSP)
Once you capture the raw `.bin` data using the DCA1000EVM, process it using a custom script (MATLAB or Python) following this standard Digital Signal Processing (DSP) pipeline:

### Step 1: Range FFT (Fast-Time Processing)
Perform a 1D FFT on the raw ADC data across the fast-time samples for every chirp. This converts the data from the time domain to the range domain. Isolate the specific range bin where the subject is located by identifying the bin with the maximum magnitude (highest reflected energy).

### Step 2: Phase Extraction (Slow-Time Processing)
Look at the complex values of your isolated range bin across consecutive frames (slow-time). Extract the phase $\phi$ of the signal. This phase is highly sensitive to sub-millimeter chest vibrations.

### Step 3: Phase Unwrapping
The extracted phase is mathematically bounded between $-\pi$ and $\pi$. If the chest displacement causes the phase to exceed these limits, the signal will artificially wrap. Apply a phase unwrapping algorithm to create a continuous signal waveform.

### Step 4: Phase Difference & Clutter Removal
Calculate the phase difference between consecutive frames. This isolates the moving components (the chest) and effectively filters out static clutter (furniture, walls) which maintain a constant phase.

### Step 5: Bandpass Filtering
Separate the composite phase signal into two distinct waveforms using digital IIR/FIR filters:
*   **Respiration Waveform:** Apply a bandpass filter from 0.1 Hz to 0.5 Hz.
*   **Cardiac Waveform:** Apply a bandpass filter from 0.8 Hz to 2.5 Hz.

### Step 6: Spectral Analysis (FFT)
Perform an FFT on the filtered respiration and cardiac waveforms. 
*   The frequency corresponding to the maximum peak in the respiration FFT is the **Breathing Rate** in Hz.
*   The frequency corresponding to the maximum peak in the cardiac FFT is the **Heart Rate** in Hz. 
*   Multiply these Hz values by 60 to convert them to Breaths/Beats Per Minute (BPM).

---

## 5. Alternative: Real-Time On-Chip Processing
If you prefer to run this project in real-time without processing raw `.bin` files offline, you can use Texas Instruments' pre-compiled software. This method utilizes the onboard DSP (C674x) and ARM cores to run the FFTs, filtering, and phase unwrapping directly on the radar chip.

1.  Download the **TI mmWave Industrial Toolbox** via TI Resource Explorer.
2.  Navigate to the `Labs > Vital Signs` directory.
3.  Use **UniFlash** to flash the specific Vital Signs `.bin` firmware directly onto your IWR1642/1443 EVM (do not use the standard MSS/BSS evaluation firmware).
4.  Launch the provided TI Vital Signs GUI executable on your host PC.
5.  Connect via the UART COM ports to view the heart rate and breathing rate plotted in real-time.
