# Flexlink Lab Tutorial for BCU

> This guide provides a reproducible set of hands-on experiments to help researchers understand the Aubergine switches and the Flexlink protocol.  
> Experiments progress from basic Port-3 management access, to switch-based forwarding, transparent tunneling, and preliminary fibre-port observations.

## 1. Introduction

This lab aims to:
- Familiarize researchers with Aubergine units
- Demonstrate practical behaviours of Flexlink control-plane reachability
- Validate transparent tunnel functionality
- Document real field observations not fully covered by official specs

Experiments are intentionally minimal, using typical PC-to-device setups.

## 2. Preparation

### 2.1 Hardware Availability / Unit Notes
Confirmed lab units:
```
15, 16, 17, 25, 26, 35
```

- `17`: non-functional  
- `35`: not discoverable via Port-3 in maintenance mode  
- Recommended usable set: **15, 16, 25, 26**

### 2.2 Software Requirements
- Controller version: **≥ 3.1.0c**
- Aubergine hardware image: **≥ 3-0-5-0**

### 2.3 Runtime Notes
- Maintenance-mode requires controller PC ↔ **Port 3**
- Disable non-Ethernet adapters (Wi-Fi, VPN, cellular, etc.)
- Static IP configuration is **not required**
- Firewall must permit LAN communication

## 3. Experiments

Assumptions:
- Controller PC connected to an Aubergine **Port 3**
- Controller launched in **maintenance mode**

### 3.1 Experiment A — Direct Management (Port 3)

**Topology**
```
PC → Aubergine (Port 3)
```

**Result**
- Controller successfully discovers the unit
- Full maintenance operations available

### 3.2 Experiment B — Management Through an Ethernet Switch

**Topology**
```
PC → Switch → Aubergine (Port 3)
```

**Result**
- Device discovered normally
- Switch does not disrupt Flexlink management signalling

**Extended Variant**
```
PC → SW → Aub1 (Port 3) → Aub2 (Port 4)
```

### 3.3 Experiment C — Transparent Tunneling (IP-over-Flexlink)

**Topology**
```
PC1 → SW1 → Aub1 (Port X)
        |---- Transparent Tunnel ----|
PC2 ← SW2 ← Aub2 (Port X)
```

**Result**
- Tunnel forms successfully after configuration on both units
- PC1 and PC2 communicate at L2 (e.g. ping)

Key properties:
- Tunnel forwards **Ethernet frames only**
- Does **not** forward Flexlink control-plane
- Controller cannot discover units behind tunnel

### 3.4 Experiment D — Fibre-Port Behaviour (Ports 10/11)

**Topology**
```
Aub1 (Port 10/11) ↔ Fibre ↔ Aub2 (Port 10/11)
```

**Result**
- Ports stayed inactive
- Controller logs: “Fail to set I2C register address for SFP module”
- Physical Flexlink MAC mode not observed

## 4. Interpretations

### 4.1 Physical vs. Virtual Links

Flexlink distinguishes:
- **Physical (Flexlink MAC)**  
  - Allows guaranteed-service flows  
  - Carries control-plane  
  - Controller discovery can extend
- **Virtual (Ethernet / UDP Encapsulation)**  
  - Best-effort traffic only  
  - Does not transport Flexlink discovery signalling  

### 4.2 LED Interpretation (Transparent Mode)

| Left LED | Right LED | Meaning |
|----------|-----------|---------|
| Off      | Green (flash) | Link down |
| Off      | Red           | Link up, tunnel not connected |
| Red      | Red           | Link up + tunnel connected |

Variations can occur due to historical device state.

## 5. Appendix

### Updating Product Code
1. Connect PC to Port 3
2. Run controller in maintenance mode (`-maintenance`)
3. Right-click unit
4. Select **Update product code**

### Configuring Transparent Tunnels
1. Right-click unit (maintenance mode)
2. Enable **Configure a transparent tunnel**
3. Enter:
   - Remote unit number  
   - Remote port number  
   - Local port number  
4. Confirm

Removing:
- Set `Remote unit = 0`

Controller status colours:
- Brown-yellow: link down  
- Red: link up, not connected  
- Green: connected  

---

End of File
