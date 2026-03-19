# PDEOncology — Tumor Drug Penetration Simulator

[![Live Demo](https://img.shields.io/badge/demo-pdeoncology.com-4d9eff?style=flat-square)](https://pdeoncology.com)
[![Version](https://img.shields.io/badge/version-v0.5-3de383?style=flat-square)](https://pdeoncology.com)
[![License](https://img.shields.io/badge/license-MIT-f0a54a?style=flat-square)](LICENSE)
[![PDEOutreach](https://img.shields.io/badge/outreach-PDEOutreach-a78bfa?style=flat-square)](https://sym19.github.io/pdeoutreach)

PDEOncology is an open-source research platform that simulates tumour drug penetration using reaction-diffusion-convection PDEs. It enables computational oncologists, bioengineers and pharmacologists to test physical barriers (IFP, stroma, vascularisation) in seconds — no installation required.

**Research-facilitating tool** • Validated against published spheroid data • Exports for Python/PhysiCell • Patient-specific imaging support

🔗 **Live site:** [pdeoncology.com](https://pdeoncology.com)  
🌐 **Companion public outreach site:** [pdeoutreach.com](https://pdeoutreach.com) 

---

## Overview

Drug resistance and treatment failure in solid tumors are often not purely pharmacological — they are **physical**. Even potent drugs can fail to reach the tumor core due to:

- Elevated interstitial fluid pressure (IFP) 
- Dense extracellular matrix (ECM) 
- Poor vascularisation

**PDEOncology** provides an interactive simulation environment to visualize drug-tumor interactions. Key features include:

- **Dynamic Modeling**: Simulate the effect of cancer drugs on 2D-modeled tumors.
- **Parameter Exploration**: Adjust molecular weight, metabolic stability, and receptor expression.
- **Visual Analysis**: Observe how biophysical barriers shape drug distribution in real-time.
---

## The PDE Model

Drug concentration C(x,y,t) evolves according to the **reaction-diffusion equation**:

```
∂C/∂t = ∇·(D(x,y)∇C) − v·∇C − λC − k·ρ(x,y)·C
```

| Symbol | Meaning | Range |
|--------|---------|-------|
| `D(x,y)` | Spatially-varying diffusion coefficient | 0.005 – 0.25 |
| `λ` | Drug degradation / metabolic clearance | 0.001 – 0.05 |
| `k` | Cellular uptake rate | 0.01 – 0.15 |
| `ρ(x,y)` | Cell density field (radially graded) | 0.05 – 1.0 |
| `r` | Tumor radius in grid units | 10 – 38 px |
| `v₀` | IFP-driven convection velocity magnitude | 0.00–0.15 | Scaled from 5–30 mmHg (Darcy’s law) | Jain (1987), Stylianopoulos (2012) |

**Numerical method:** Explicit finite difference (FTCS) on an 80×80 grid. CFL stability condition: `dt ≤ dx² / (4D)`.

---

## Features

| Feature | Description |
|---------|-------------|
| **PDE Simulation** | Real-time FDM solver, magma heatmap, radial penetration curve, 4 metrics |
| **Diffusion Animation** | 20-frame playback with Play/Pause/Scrub/Speed controls |
| **AI Drug Input** | Claude API extracts D, λ, k from plain English. Local DB fallback when offline |
| **Compare Mode** | Two drugs on same tumor — side-by-side heatmaps, overlay curves, auto summary |
| **Drug Database** | 21 drug × tumor combinations, searchable and filterable |
| **Results & Export** | Auto-generated report, CSV/LaTex/Python/PhysiCell export, heatmap PNG export, print |
| **3 Delivery modes** | IV infusion / vascular ring / intratumoral injection |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML / CSS / JavaScript — single `index.html` |
| Fonts | Space Mono + Inter (Google Fonts) |
| PDE solver | Custom FTCS finite difference (pure JS) |
| Visualisation | HTML5 Canvas API |
| AI integration | Anthropic Claude API (`claude-sonnet-4.5`) |
| CORS proxy | Cloudflare Workers (free tier) |
| Hosting | GitHub Pages |
| Domain | pdeoncology.com |

No frameworks. No build tools. No server. One file.

---

## Project Structure

```
tumor-drug-penetration-model/
├── index.html          # Entire application (HTML + CSS + JS, bilingual)
├── CNAME               # Custom domain config
├── README.md           # This file
├── LICENSE             # MIT License
├── requirements.txt    # Python dependencies (Colab notebook)
└── Tumor_Drug_Simulation_Core.ipynb   # Original Colab prototype
```

---

## Getting Started

### Run locally
```bash
# No build step — just open in browser
open index.html
# or serve with Python
python -m http.server 8000
```

### Deploy to GitHub Pages
1. Fork this repository
2. Settings → Pages → Source: main branch / root
3. Live at `https://yourusername.github.io/tumor-drug-penetration-model`

### Set up AI Drug Input (optional)
1. Get an API key from [console.anthropic.com](https://console.anthropic.com)
2. Deploy the Cloudflare Worker below as a CORS proxy
3. Update the worker URL in `index.html` (search for `workers.dev`)
4. Enter your key in the AI Drug Input tab

### Cloudflare Workers CORS Proxy
```javascript
export default {
  async fetch(request) {
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Content-Type, x-api-key, anthropic-version',
        }
      });
    }
    if (request.method !== 'POST') {
      return new Response('PDEOncology API Proxy — OK', {
        headers: { 'Access-Control-Allow-Origin': '*' }
      });
    }
    const body = await request.json();
    const apiKey = request.headers.get('x-api-key');
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': apiKey,
        'anthropic-version': '2023-06-01',
      },
      body: JSON.stringify(body)
    });
    const data = await res.json();
    return new Response(JSON.stringify(data), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
      }
    });
  }
}
```

---

## Limitations

PDEOncology is a research-facilitating tool with deliberate simplifications for speed and accessibility:
• 2D cross-section only (real tumours are 3-D)
• Static geometry and fields (no dynamic remodelling)
• Normalised parameters (not absolute clinical doses)
• FTCS + upwind scheme (first-order accurate)
• Histology upload uses simple thresholding (expert manual segmentation recommended for clinical data)

---

## Version History

| Version | Date | Highlights |
|---------|------|-----------|
| v0.5 | March 2026 | IFP convection (Darcy’s law), experimental validation (RMSE < 0.08 vs Thurber et al.), histology/MRI upload, parameter sweep, Python/LaTeX export, full units & DOI links |
| v0.4 | Mar 2026 | 20-frame animation, Compare tab, bilingual CN/EN, favicon, full About tab |
| v0.3 | Mar 2026 | UI overhaul — Space Mono/Inter, dark academic theme, mobile responsive |
| v0.2 | Mar 2026 | Cloudflare Worker, Claude API, AI fallback, drug DB (21 entries), export |
| v0.1 | Mar 2026 | Initial — PDE solver, heatmap, radial curve, 3 delivery modes |

---

## References

1. [Jain RK. Transport of molecules in the tumor interstitium. *Cancer Research*, 47(12):3039–3051, 1987.](https://pubmed.ncbi.nlm.nih.gov/3555767/)
2. [Nugent LJ, Jain RK. Extravascular diffusion in normal and neoplastic tissues. *Cancer Research*, 44(1):238–244, 1984.](https://pubmed.ncbi.nlm.nih.gov/6398639/)
3. [Thurber GM, Schmidt MM, Wittrup KD. Antibody tumor penetration. *Advanced Drug Delivery Reviews*, 60(12):1421–1434, 2008.](https://pubmed.ncbi.nlm.nih.gov/18541331/)
4. [Chauhan VP et al. Delivery of molecular and nanoscale medicine to tumors. *Annual Review of Chemical and Biomolecular Engineering*, 2:281–298, 2011.](https://pubmed.ncbi.nlm.nih.gov/22432620/)
5. [Tannock IF et al. Limited penetration of anticancer drugs through tumor tissue. *Clinical Cancer Research*, 8(3):878–884, 2002.](https://pubmed.ncbi.nlm.nih.gov/11895922/)
6. [Minchinton AI, Tannock IF. Drug penetration in solid tumours. *Nature Reviews Cancer*, 6(8):583–592, 2006.](https://pubmed.ncbi.nlm.nih.gov/16862189/)

---

## Team

**Yumeng Shi** — Technical Lead & Systems Architect

Focus: Infrastructure, PDE Implementation, and Interactive Design

Responsible for the project’s computational backbone and digital ecosystem. Yumeng engineered the core numerical solvers, developed the end-to-end frontend architecture, and managed the deployment of the integrated PDE platform and outreach sites.


**Tracey Yang** — Scientific Lead & Biophysical Modeller

Focus: Computational Research, Data Validation, and Model Grounding

Directs the biophysical integrity of the platform. Tracey oversaw the development of the convection-diffusion frameworks, engineered the data conversion pipelines for clinical imaging (MRI/Histology), and established the project’s grounding through literature-based parameter synthesis and experimental validation.

---

## Related

- **[PDEOutreach](https://pdeoutreach.com)** — the public-facing companion platform. Cancer science explained for everyone, with interactive quizzes, risk profiles, and real patient stories.

---


## License

MIT License — free to use, modify, and build upon. Please cite if used in academic work.
