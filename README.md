# PDEOncology — Tumor Drug Penetration Simulator

[![Live Demo](https://img.shields.io/badge/demo-pdeoncology.com-4d9eff?style=flat-square)](https://pdeoncology.com)
[![Version](https://img.shields.io/badge/version-v0.4-3de383?style=flat-square)](https://pdeoncology.com)
[![License](https://img.shields.io/badge/license-MIT-f0a54a?style=flat-square)](LICENSE)
[![PDEOutreach](https://img.shields.io/badge/outreach-PDEOutreach-a78bfa?style=flat-square)](https://sym19.github.io/pdeoutreach)

A browser-based platform for simulating tumor drug penetration using **reaction-diffusion partial differential equations (PDEs)**. All computation runs client-side in JavaScript — no server, no installation required. Available in English and Chinese (中英双语).

🔗 **Live site:** [pdeoncology.com](https://pdeoncology.com)  
🌐 **Public outreach:** [pdeoutreach.com](https://pdeoutreach.com) — cancer science for everyone

---

## Overview

Drug resistance and treatment failure in solid tumors are often not purely pharmacological — they are **physical**. Even potent drugs can fail to reach the tumor core due to:

- Elevated interstitial fluid pressure (IFP) / 升高的间质液压
- Dense extracellular matrix (ECM) / 致密的细胞外基质
- Poor vascularisation / 血管分布不良

PDEOncology provides an interactive simulation environment to visualise these phenomena. Users can explore how molecular weight, metabolic stability, and receptor expression shape drug distribution — without requiring a computational background.

---

## The PDE Model

Drug concentration C(x,y,t) evolves according to the **reaction-diffusion equation**:

```
∂C/∂t = ∇·(D(x,y)∇C) − λC − k·ρ(x,y)·C
```

| Symbol | Meaning | Range |
|--------|---------|-------|
| `D(x,y)` | Spatially-varying diffusion coefficient | 0.005 – 0.25 |
| `λ` | Drug degradation / metabolic clearance | 0.001 – 0.05 |
| `k` | Cellular uptake rate | 0.01 – 0.15 |
| `ρ(x,y)` | Cell density field (radially graded) | 0.05 – 1.0 |
| `r` | Tumor radius in grid units | 10 – 38 px |

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
| **Results & Export** | Auto-generated report, CSV export, PNG export, print |
| **Bilingual** | Full Chinese/English language toggle (中英双语) |
| **3 Delivery modes** | IV infusion / vascular ring / intratumoral injection |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML / CSS / JavaScript — single `index.html` |
| Fonts | Space Mono + Inter (Google Fonts) |
| PDE solver | Custom FTCS finite difference (pure JS) |
| Visualisation | HTML5 Canvas API |
| AI integration | Anthropic Claude API (`claude-sonnet-4`) |
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

PDEOncology is an educational and exploratory tool. Key simplifications:

- Tumor geometry is circular and homogeneous — real tumors are irregular
- 2D model only — real drug penetration is three-dimensional
- Static cell density and diffusion fields — in reality these evolve with treatment
- No convective transport (IFP-driven bulk flow) — diffusion only
- Normalised dimensionless parameters — not directly comparable to clinical doses
- FTCS is first-order in time — higher-order methods improve accuracy

> Results are model approximations. Not for clinical decision-making.

---

## Version History

| Version | Date | Highlights |
|---------|------|-----------|
| **v0.4** | Mar 2026 | 20-frame animation, Compare tab, bilingual CN/EN, favicon, full About tab |
| v0.3 | Mar 2026 | UI overhaul — Space Mono/Inter, dark academic theme, mobile responsive |
| v0.2 | Mar 2026 | Cloudflare Worker, Claude API, AI fallback, drug DB (21 entries), export |
| v0.1 | Mar 2026 | Initial — PDE solver, heatmap, radial curve, 3 delivery modes |
| v0.5 *(upcoming)* | — | DB expansion (50+ entries), Claude-generated report, parameter sweep |

---

## References

1. Jain RK. Transport of molecules in the tumor interstitium. *Cancer Research*, 47(12):3039–3051, 1987.
2. Nugent LJ, Jain RK. Extravascular diffusion in normal and neoplastic tissues. *Cancer Research*, 44(1):238–244, 1984.
3. Thurber GM, Schmidt MM, Wittrup KD. Antibody tumor penetration. *Advanced Drug Delivery Reviews*, 60(12):1421–1434, 2008.
4. Chauhan VP et al. Delivery of molecular and nanoscale medicine to tumors. *Annual Review of Chemical and Biomolecular Engineering*, 2:281–298, 2011.
5. Tannock IF et al. Limited penetration of anticancer drugs through tumor tissue. *Clinical Cancer Research*, 8(3):878–884, 2002.
6. Minchinton AI, Tannock IF. Drug penetration in solid tumours. *Nature Reviews Cancer*, 6(8):583–592, 2006.

---

## Team

**Yumeng S.** — Technical Development, Website Engineering, UI Design, PDE Implementation  
*NSFZ · IB*

Designed and built the full PDEOncology platform — including the finite difference PDE solver, interactive visualisation engine, Claude API integration, bilingual language system, and the complete frontend interface.

**Tracey Y.** — Literature Research, Drug Database, Science Communication, Outreach & Promotion  
*Wycombe Abbey · UK*

Leads research and content development — curating the drug parameter database from published literature, sourcing biophysical references, and driving outreach across PDEOutreach and social media channels.

> 祝我们都能去自己想上的学校！@MIT @Cambridge

---

## Related

- **[PDEOutreach](https://sym19.github.io/pdeoutreach)** — the public-facing companion platform. Cancer science explained for everyone, with interactive quizzes, risk profiles, and real patient stories.

---

## License

MIT License — free to use, modify, and build upon. Please cite if used in academic work.
