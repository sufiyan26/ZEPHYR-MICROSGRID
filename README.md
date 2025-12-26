# Zephyr RTOS Microgrid Simulator

A 6‑node **battery-powered IoT microgrid** simulated on Zephyr RTOS (QEMU), focused on cooperative energy sharing, undervoltage protection, and real‑time telemetry. Each node represents a Li‑ion–powered device with heterogeneous harvesting and load patterns, coordinated by a greedy richest‑to‑weakest controller.

## Core Concept & Objective

- **Goal:** Prevent brown‑outs in a cluster of battery nodes by **redistributing energy** in real time whenever any node approaches a low‑voltage protection threshold.
- **Key Idea:** Run all nodes under Zephyr RTOS, periodically scan their voltages, and trigger controlled “energy transfers” from the richest node to the weakest node while ensuring donor nodes do not drop below a safe minimum.

## Technical Design

- **Platform:** Zephyr RTOS on `qemu_x86_64` (simulation target).
- **Nodes:**
  - 6 virtual nodes with individual voltage state (≈3.2–4.2 V band).
  - Simple stochastic models for load and energy harvesting per node.
- **Energy Sharing Controller:**
  - Periodic task finds:
    - `richest` node (max voltage)
    - `weakest` node (min voltage)
  - If `weakest < 3.60 V` (Li‑ion protection threshold) and `richest > donor_min`:
    - Transfer a fixed quantum of “energy” (e.g., 50 mV equivalent) from richest to weakest.
  - Protects donors from going below a configured minimum (e.g., 3.2 V).
- **Telemetry:**
  - Uses Zephyr `printk` to log timestamp, node ID, voltage, and share events.
  - Logs exported to a text file and analyzed offline in Python/Colab to compute uptime and generate voltage‑vs‑time plots.

## Project
├── src/
│ └── main.c # Zephyr application: nodes + controller loop
├── prj.conf # Zephyr configuration (logging, timing)
├── logs/
│ └── microgrid.log # Sample telemetry from a long run
└── analysis/
└── analyze.ipynb # Python notebook for plots and metrics

## Example Algorithm (Pseudo‑Code)
for each control_period {
update_node_voltages(); // load + harvest effects
richest = argmax(nodes[i].voltage);
weakest = argmin(nodes[i].voltage);

if (nodes[weakest].voltage < 3600 && nodes[richest].voltage > 3200) {
    nodes[richest].voltage -= 50;
    nodes[weakest].voltage += 50;
    printk("SHARE r=%d w=%d Vr=%d Vw=%d\n", richest, weakest,
           nodes[richest].voltage, nodes[weakest].voltage);
}
}


## Observed Results (Example Run)

- **All‑nodes‑safe uptime (≥3.60 V):** ≈ **93%** over tens of thousands of timesteps.
- **Total sharing events:** **10k+** transfers in a long simulation.
- **Minimum recorded voltage:** ≈ **3.21 V** (still above deep‑discharge region).
- Clear visual proof of:
  - Nodes dipping toward threshold.
  - Controller redistributing “energy”.
  - System returning to safe operating region.



## Skills Demonstrated

Embedded C with Zephyr RTOS • QEMU simulation • Real‑time task design • Cooperative control algorithms • Telemetry logging and analysis • Python data processing • Energy management concepts for industrial IoT microgrids




