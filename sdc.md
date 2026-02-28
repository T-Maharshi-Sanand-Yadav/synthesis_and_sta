Good — I’ll  **clean, organize, fix syntax errors, and structure this like an industry SDC** , block-wise.
Your original content had issues:

* Repeated `set_units`
* Mixed designs together
* Wrong commands (`set_unit`, `all_output`)
* Duplicate `sdc_version`
* Typo: `iset`
* Clock uncertainty applied to wrong clock
* No clear separation per block

Below is a  **properly arranged SDC template** .
Use **one section per design** (don’t keep all active together in one file unless switching with `$DESIGN`).

---

# Clean Structured Constraints (Industry Format)

---

## Global Template (Common Header)

```tcl
#############################################
# Common Settings
#############################################
set sdc_version 2.0
set_units -time 1000ps
set_units -capacitance 1000fF
```

---

# 1. AMBER (a23_core)

```tcl
#############################################
# Design : AMBER
#############################################
current_design a23_core

# Clock
create_clock -name DES_CK -period 1.42 -waveform {0 0.71} [get_ports i_clk]
create_clock -name VCLK   -period 1.42 -waveform {0 0.71}

# Clock uncertainty
set_clock_uncertainty -setup 0.14 [get_clocks DES_CK]
set_clock_uncertainty -hold  0.071 [get_clocks DES_CK]

# IO delays
set_input_delay  -clock VCLK -max 0.852 [all_inputs]
set_input_delay  -clock VCLK -min 0.284 [all_inputs]
set_output_delay -clock VCLK -max 0.568 [all_outputs]
set_output_delay -clock VCLK -min 0.142 [all_outputs]

# Optimization
set_attribute lbr_seq_in_out_phase_opto true
```

---

# 2. PACKET PROCESSOR / MEMORY

```tcl
#############################################
# Design : Packet Processor
#############################################
current_design pck_proc_int_mem_TOP

create_clock -name pck_proc_int_mem_fsm_clk -period 2 -waveform {0 1} \
[get_ports pck_proc_int_mem_fsm_clk]
create_clock -name VCLK -period 2 -waveform {0 1}

# Clock quality
set_clock_transition 0.25 [get_clocks pck_proc_int_mem_fsm_clk]
set_clock_uncertainty 0.01 [get_clocks pck_proc_int_mem_fsm_clk]

# IO constraints
set_input_delay  -clock VCLK -max 0.8 [all_inputs]
set_input_delay  -clock VCLK -min 0.4 [all_inputs]
set_output_delay -clock VCLK -max 0.5 [all_outputs]
set_output_delay -clock VCLK -min 0.2 [all_outputs]

# Design environment
set_input_transition 0.12 [all_inputs]
set_load 0.15 [all_outputs]
set_max_fanout 30 [current_design]
```

---

# 3. PSVPU (Vector Processor)

```tcl
#############################################
# Design : Vector Processor
#############################################
current_design vector_processor_top

create_clock -name vector_processor_top_clk -period 2 -waveform {0 1} \
[get_ports vector_processor_top_clk]
create_clock -name VCLK -period 2 -waveform {0 1}

set_clock_transition 0.25 [get_clocks vector_processor_top_clk]
set_clock_uncertainty 0.01 [get_clocks vector_processor_top_clk]

set_input_delay  -clock VCLK -max 0.8 [all_inputs]
set_input_delay  -clock VCLK -min 0.4 [all_inputs]
set_output_delay -clock VCLK -max 0.5 [all_outputs]
set_output_delay -clock VCLK -min 0.2 [all_outputs]

set_input_transition 0.12 [all_inputs]
set_load 0.15 [all_outputs]
```

---

# 4. SHA256 (1.4 GHz)

```tcl
#############################################
# Design : SHA256
#############################################
current_design sha256

create_clock -name CLK  -period 0.71 -waveform {0 0.35} [get_ports clk]
create_clock -name VCLK -period 0.71 -waveform {0 0.35}

set_clock_uncertainty -setup 0.14 [get_clocks CLK]
set_clock_uncertainty -hold  0.07 [get_clocks CLK]

# Inputs
set_input_delay -clock VCLK -max 0.42 [get_ports {reset_n cs we address[*] write_data[*]}]
set_input_delay -clock VCLK -min 0.18 [get_ports {reset_n cs we address[*] write_data[*]}]

# Outputs
set_output_delay -clock VCLK -max 0.14 [get_ports {read_data[*] error}]
set_output_delay -clock VCLK -min 0.07 [get_ports {read_data[*] error}]

# Cell restrictions
set_dont_use {DFFNSRX1 DFFNSRX2 DFFNSRX4 DFFNSRXL DFFRHQX1 DFFRHQX2 DFFRHQX4 DFFRHQX8}
```

---

# 5. SPU 64

```tcl
#############################################
# Design : SPU 64
#############################################
current_design spu_core_top_64

create_clock -name spu_core_top_clk -period 2 -waveform {0 1} \
[get_ports spu_core_top_clk]
create_clock -name VCLK -period 2 -waveform {0 1}

set_clock_transition 0.25 [get_clocks spu_core_top_clk]
set_clock_uncertainty 0.01 [get_clocks spu_core_top_clk]

set_input_delay  -clock VCLK -max 0.8 [all_inputs]
set_input_delay  -clock VCLK -min 0.4 [all_inputs]
set_output_delay -clock VCLK -max 0.5 [all_outputs]
set_output_delay -clock VCLK -min 0.2 [all_outputs]

set_input_transition 0.12 [all_inputs]
set_load 0.15 [all_outputs]
set_max_fanout 30 [current_design]
```

---

# What I Fixed

* Removed duplicates
* Corrected:
  * `set_unit` → `set_units`
  * `all_output` → `all_outputs`
  * `iset` typo
* Grouped per design
* Clean industry structure
* Removed conflicting constraints
* Correct clock names
* Clean formatting for Genus/PT

---

# Important Usage Note

**Do NOT use all sections together.**

Instead:

```
amber.sdc
sha256.sdc
spu.sdc
```

Each design → separate SDC.

---

If you want to become strong in AMD STA/synthesis, next I can give:

 **Industry “Perfect SDC Template” (what real projects use + common mistakes interviewers ask)** .
