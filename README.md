# # The Fabric Wars
### NVSwitch · NVLink · UALink · Ultra Ethernet

Companion repository to the [Fabric Wars article](YOUR_SUBSTACK_URL) published on Substack.

Interactive visual explainer + hands-on PyCUDA code demonstrating how GPU interconnect actually works — from the crossbar ASIC to Unified Virtual Addressing — with measured bandwidth comparisons you can run yourself.

---

## Live Demos

| Resource | What it covers |
|---|---|
| [Interactive Visual Explainer](https://fahad-najam.github.io/fabric-wars/nvswitch-visuals.html) | Animated crossbar grid · NVSwitch generation evolution · GB200 NVL72 rack anatomy · Scale-up vs scale-out fabric |
| [Code Companion — Show Me the Silicon](https://fahad-najam.github.io/fabric-wars/nvswitch-code-companion.html) | PyCUDA examples with live bandwidth simulators · PCIe vs NVLink vs UVA |

---

## What's In This Repo

```
fabric-wars/
├── nvswitch-visuals.html          # Interactive visual explainer (4 animated diagrams)
├── nvswitch-code-companion.html   # Code companion with syntax-highlighted PyCUDA
├── nvswitch-interconnect-post.html # Full article HTML version
└── README.md
```

---

## The Three Code Examples

All runnable on any multi-GPU system with NVLink (DGX H100, HGX H100, H200, or equivalent).

### Example 01 — PCIe Baseline
CPU pinned memory → GPU via PCIe. Timed with CUDA events.
Establishes the before — the ceiling you're breaking through.
Expected: **~55 GB/s** on PCIe Gen5.

### Example 02 — NVLink Peer-to-Peer
GPU 0 HBM → NVSwitch crossbar → GPU 1 HBM. CPU not involved. PCIe bypassed.
Expected: **~700 GB/s** on H100 (NVLink 4.0), **~1,800 GB/s** on B200 (NVLink 5.0).

### Example 03 — Unified Virtual Address
One pointer. `cudaMemcpyDefault`. CUDA inspects the address, detects GPU-to-GPU,
routes via NVLink automatically. No manual routing. Same bandwidth as Example 02.

```bash
# Verify NVLink topology first
nvidia-smi topo -m               # NV4 = NVLink 4.0 connected
nvidia-smi nvlink --status -i 0  # check active lanes on GPU 0

# Install
pip install pycuda numpy

# Run in order — compare the numbers
python 01_pcie_baseline.py
python 02_nvlink_peer_to_peer.py
python 03_unified_virtual_address.py
```

---

## The Core Concepts

What a crossbar is:
An N×N grid of switch points in silicon. Every sender has a direct path to every receiver simultaneously. Non-blocking means no combination of active transfers creates a bottleneck. NVSwitch implements this as a physical ASIC between GPUs.

What NVLink is:
The high-speed protocol and SerDes running over that crossbar. NVLink is the language. NVSwitch is the postal system. Without NVSwitch, NVLink only connects two GPUs. With NVSwitch, it becomes a full fabric.

What UVA is:
Unified Virtual Addressing creates one shared address space across all GPU and CPU memories. CUDA reads the pointer, knows which device owns it, picks the fastest available path — NVLink if peer access is enabled. The address is the routing instruction. The crossbar executes it.

Scale-up vs scale-out:
NVLink/NVSwitch and UALink are scale-up fabrics — inside a pod, load/store memory semantics, sub-microsecond latency. Ultra Ethernet and InfiniBand are scale-out fabrics — rack to rack, message-passing, datacenter-wide. They are collaborators, not competitors.

---

## Bandwidth Reference

| Path | Technology | Bandwidth | Latency |
|---|---|---|---|
| CPU → GPU | PCIe Gen5 x16 | ~64 GB/s | ~10 µs |
| GPU ↔ GPU | NVLink 4.0 (H100) | 900 GB/s | sub-µs |
| GPU ↔ GPU | NVLink 5.0 (B200) | 1,800 GB/s | sub-µs |
| Rack-wide | NVL72 all-to-all | 130 TB/s | sub-µs |
| Pod-scale | UALink 1.0 (spec) | 800 Gb/s/port | 100–150 ns |
| Scale-out | Ultra Ethernet | 400–800 Gb/s | ~1–2 µs |

---

## About

Fahad Najam is a Principal AI Infrastructure Architect and founder of Fahad Najam Consulting. His work spans 15+ years across firmware, SmartNIC/RDMA, edge AI inference, and large-scale GPU cluster deployment — including the NVIDIA H200 Superpod at Texas A&M, where he led full-stack infrastructure build-out across DDN storage, Spectrum-X networking, BlueField-3 DPUs, and liquid-cooled compute.

His frameworks — Energy-Aware Computing (EAC) and Power-Thermal Aware Scheduling (PTAS) — treat energy-per-token as a first-class infrastructure metric.

→ [Substack — Silicon & Soul](YOUR_SUBSTACK_URL)
→ [LinkedIn](YOUR_LINKEDIN_URL)

---

*© 2026 Fahad Najam Consulting · All rights reserved*
