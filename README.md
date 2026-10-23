# Mixed-Signal SPICE Simulator (Python + VHDL) for Google Colab

A custom circuit simulator based on Modified Nodal Analysis (MNA), designed to run entirely in Google Colab. It features co-simulation with Python and VHDL code blocks, transient analysis, and transfer function support, all inside a single, interactive Jupyter Notebook.

## 🚀 Features

- **Custom MNA Engine:** Solves R, L, C, independent/dependent sources, and ideal opamps using a custom Python/NumPy implementation.
- **Mixed-Signal Co-Simulation:** Seamlessly integrates analog circuitry with:
    - **Python blocks:** For ADCs, digital logic, or complex math.
    - **VHDL blocks:** For synchronous logic, state machines, and registers (supports `process`, `rising_edge`, and vectors).
- **Transfer Function (FDT) Blocks:** Define dynamic systems using numerator/denominator coefficients (e.g., filters) directly in the netlist. Internally solved using state-space and Backward Euler.
- **Transient Analysis:** Fixed-step Backward Euler integration for time-domain simulation.
- **Interactive GUI:** A built-in editor with syntax highlighting, error panels, and the ability to save directly to Google Drive.
- **Advanced Plotting:** Supports `SUBPLOT=ON` for stacked vertical plots, ideal for comparing digital and analog signals.

## 🛠️ How to Run

You don't need to install anything. The entire simulator runs in the cloud.

1.  Click the button below to open the notebook in Google Colab.
2.  Run the single cell to initialize the GUI and the simulation engine.
3.  Write or paste your SPICE netlist in the editor.
4.  Click **Simulate**.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rick-starnez/simulador-spice-modo_mixto/blob/main/SimuladorSPICE.ipynb)

(Remember to replace `YOUR_USERNAME`, `YOUR_REPO_NAME`, and `YOUR_NOTEBOOK_NAME.ipynb` with your actual GitHub details).*

## 📖 Netlist Syntax

The simulator supports standard SPICE elements (`R`, `C`, `L`, `V`, `I`, `E`, `G`) and custom directives for mixed-signal blocks.

### Example: Complete System (ADC -> VHDL -> DAC -> FDT)

```spice
* Rampa -> ADC (Python) -> Registro (VHDL) -> DACs -> Filtro FDT

V_in 1 0 PWL(0 0 10m 5 20m 0)
V_clk 2 0 PULSE(0 5 1m 1u 1u 1m 2m)

* Python ADC Block
.SUBCKT adc_python
.PORTS IN_ANALOG: v_analog
.PORTS OUT: adc_bits[0..2]
.MAP v_analog -> 1
.MAP adc_bits -> 3..5
.PARAMS V_HIGH=5.0 V_TH=2.5
.LANG python
.CODE
valor = int(round((v_analog / v_high) * 7.0))
if valor > 7: valor = 7
if valor < 0: valor = 0
adc_bits[0] = float((valor >> 0) & 1)
adc_bits[1] = float((valor >> 1) & 1)
adc_bits[2] = float((valor >> 2) & 1)
.ENDCODE
.ENDS

* VHDL Register Block
.SUBCKT registro_vhdl
.PORTS IN_DIG: clk
.PORTS IN_DIG: data_in[0..2]
.PORTS OUT: Q_act[0..2]
.MAP clk -> 2
.MAP data_in -> 3..5
.MAP Q_act -> 7..9
.LANG vhdl
.CODE
Q_act[0] <= data_in[0];
Q_act[1] <= data_in[1];
Q_act[2] <= data_in[2];
.ENDCODE
.ENDS

* FDT Block (Low-pass filter, tau=2ms)
.SUBCKT filtro_fdt
.PORTS IN_ANALOG: vin
.PORTS OUT: vout
.MAP vin -> 9
.MAP vout -> 10
.FDT NUM: [1]
.FDT DEN: [0.002, 1]
.ENDS
