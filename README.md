The Fabric Wars

NVSwitch · NVLink · UALink · Ultra Ethernet

Companion repository to The Fabric Wars article published on Substack.

Interactive visual explainer + hands-on PyCUDA code demonstrating how GPU interconnect actually works — from the crossbar ASIC to Unified Virtual Addressing — with measured bandwidth comparisons you can run yourself.

---

Live Project Page

The project page is hosted here:

https://fahad18719.github.io/fabric-wara

Use this page for the interactive visual explainer, code companion, and supporting material.

---

Live Demos

Resource| Link
Interactive Visual Explainer| https://fahad18719.github.io/fabric-wara
Code Companion — Show Me the Silicon| https://fahad18719.github.io/fabric-wara

Note: The direct demo pages will be linked separately after GitHub Pages confirms the individual HTML files are deployed correctly.

---

What's In This Repo

fabric-wara/
├── nvswitch-visuals.html           # Interactive visual explainer
├── nvswitch-code-companion.html    # Code companion with PyCUDA examples
├── nvswitch-interconnect-post.html # Full article HTML version
└── README.md

---

The Three Code Examples

All runnable on any multi-GPU system with NVLink, including DGX H100, HGX H100, H200, or equivalent systems.

Example 01 — PCIe Baseline

CPU pinned memory → GPU via PCIe. Timed with CUDA events.

Establishes the baseline — the ceiling you are breaking through.

Expected: ~55 GB/s on PCIe Gen5.

Example 02 — NVLink Peer-to-Peer

GPU 0 HBM → NVSwitch crossbar → GPU 1 HBM. CPU not involved. PCIe bypassed.

Expected: ~700 GB/s on H100 with NVLink 4.0, and ~1,800 GB/s on B200 with NVLink 5.0.

Example 03 — Unified Virtual Addressing

One pointer. "cudaMemcpyDefault".

CUDA inspects the address, detects GPU-to-GPU movement, and routes through NVLink automatically. No manual routing. Same bandwidth path as Example 02.

# Verify NVLink topology first
nvidia-smi topo -m
nvidia-smi nvlink --status -i 0

# Install dependencies
pip install pycuda numpy

# Run in order and compare the numbers
python 01_pcie_baseline.py
python 02_nvlink_peer_to_peer.py
python 03_unified_virtual_address.py

---

The Core Concepts

What a crossbar is

An N×N grid of switch points in silicon. Every sender has a direct path to every receiver simultaneously. Non-blocking means no combination of active transfers creates a bottleneck. NVSwitch implements this as a physical ASIC between GPUs.

What NVLink is

The high-speed protocol and SerDes running over that crossbar. NVLink is the language. NVSwitch is the postal system. Without NVSwitch, NVLink only connects two GPUs. With NVSwitch, it becomes a full fabric.

What UVA is

Unified Virtual Addressing creates one shared address space across GPU and CPU memory. CUDA reads the pointer, knows which device owns it, and picks the fastest available path — NVLink if peer access is enabled.

The address is the routing instruction. The crossbar executes it.

Scale-up vs scale-out

NVLink/NVSwitch and UALink are scale-up fabrics — inside a pod, using load/store memory semantics and sub-microsecond latency.

Ultra Ethernet and InfiniBand are scale-out fabrics — rack to rack, message-passing, datacenter-wide.

They are collaborators, not competitors.

---

Bandwidth Reference

Path| Technology| Bandwidth| Latency
CPU → GPU| PCIe Gen5 x16| ~64 GB/s| ~10 µs
GPU ↔ GPU| NVLink 4.0 / H100| 900 GB/s| sub-µs
GPU ↔ GPU| NVLink 5.0 / B200| 1,800 GB/s| sub-µs
Rack-wide| NVL72 all-to-all| 130 TB/s| sub-µs
Pod-scale| UALink 1.0 spec| 800 Gb/s/port| 100–150 ns
Scale-out| Ultra Ethernet| 400–800 Gb/s| ~1–2 µs

---

About

Fahad Najam is a Principal AI Infrastructure Architect and founder of Fahad Najam Consulting. His work spans 15+ years across firmware, SmartNIC/RDMA, edge AI inference, and large-scale GPU cluster deployment — including NVIDIA H200 Superpod infrastructure at Texas A&M, with full-stack infrastructure work across DDN storage, Spectrum-X networking, BlueField-3 DPUs, and liquid-cooled compute.

His frameworks — Energy-Aware Computing and Power-Thermal Aware Scheduling — treat energy-per-token as a first-class infrastructure metric.

---

Links

Substack — Silicon & Soul: PASTE_SUBSTACK_ARTICLE_URL_HERE

LinkedIn: https://www.linkedin.com/in/fahadnajam

Project page: https://fahad18719.github.io/fabric-wara

---

© 2026 Fahad Najam Consulting · All rights reserved