# PDEOncology — Tumor Drug Penetration Simulator

[![Live Demo](https://img.shields.io/badge/demo-pdeoncology.com-4d9eff?style=flat-square)](https://pdeoncology.com)
[![Version](https://img.shields.io/badge/version-v0.5-3de383?style=flat-square)](https://pdeoncology.com)
[![License](https://img.shields.io/badge/license-AGPL--3.0-f0a54a?style=flat-square)](https://github.com/SYM19/PDEOncology/blob/main/LICENSE)
[![PDEOutreach](https://img.shields.io/badge/outreach-PDEOutreach-a78bfa?style=flat-square)](https://pdeoutreach.com)

A browser-based platform for simulating tumor drug penetration using **reaction-diffusion-convection partial differential equations (PDEs)**. All computation runs client-side in JavaScript — no server, no installation required.

🔗 **Live site:** [pdeoncology.com](https://pdeoncology.com)  
🌐 **Public outreach:** [PDEOutreach](https://pdeoutreach.com) — cancer science for everyone

---

## Overview

Drug resistance and treatment failure in solid tumors are often not purely pharmacological — they are **physical**. Even potent drugs can fail to reach the tumor core due to:

* Elevated interstitial fluid pressure (IFP) / 升高的间质液压
* Dense extracellular matrix (ECM) / 致密的细胞外基质
* Poor vascularisation / 血管分布不良

PDEOncology provides an interactive simulation environment to visualise these phenomena. Users can explore how molecular weight, metabolic stability, receptor expression, and IFP-driven convection shape drug distribution — without requiring a computational background.

---

## The PDE Model

Drug concentration C(x,y,t) evolves according to the **reaction-diffusion-convection equation**:

```
∂C/∂t = ∇·(D(x,y)∇C) − v·∇C − λC − k·ρ(x,y)·C
```

| Symbol | Meaning | Range |
| --- | --- | --- |
| `D(x,y)` | Spatially-varying diffusion coefficient | 0.005 – 0.25 |
| `v` | IFP-driven interstitial flow velocity (Darcy's law) | 0.00 – 0.15 |
| `λ` | Drug degradation / metabolic clearance | 0.001 – 0.05 |
| `k` | Cellular uptake rate | 0.01 – 0.15 |
| `ρ(x,y)` | Cell density field (radially graded or image-derived) | 0.05 – 1.0 |
| `r` | Tumor radius in grid units | 10 – 38 px |

**Numerical method:** Explicit finite difference (FTCS) on an 80×80 grid. Convection discretised with an upwind scheme. Extended CFL stability condition: `dt ≤ min(dx²/4D, dx/max|v|)`.

### IFP convection

In solid tumors, elevated interstitial fluid pressure (IFP) creates an outward radial flow that opposes inward drug delivery. This is modelled using Darcy's law for a homogeneous spherical tumor (Jain 1987):

```
v(r) ≈ ifpMag · (r/R) · r̂    [r ≤ R, inside tumor]
```

The linear radial profile (v ∝ r) is the standard result for uniform interstitial conductivity (Jain 1987, Stylianopoulos et al. 2012).

---

## Features

| Feature | Description |
| --- | --- |
| **PDE Simulation** | Real-time FDM solver, magma heatmap, radial penetration curve, 4 metrics |
| **IFP Convection** | Darcy's law outward flow (Jain 1987), upwind scheme, IFP magnitude slider |
| **Receptor Expression (φ)** | Multiplicative uptake scaling — models HER2 overexpression, ADC targeting |
| **Particle Size → D_eff** | Stokes-Einstein slider: radius (nm) auto-syncs D slider; tissue D note included |
| **Two-Compartment NP** | Encapsulated depot (C_nano) releasing bioavailable drug (C_free) at rate κ |
| **Diffusion Animation** | 20-frame playback with Play/Pause/Scrub/Speed controls |
| **AI Drug Input** | Claude API extracts D, λ, k from plain English; local DB fallback when offline |
| **Compare Mode** | Two drugs on same tumor — side-by-side heatmaps, overlay curves, auto summary |
| **Validation Tab** | Model vs. digitised spheroid data (Thurber 2008, Minchinton 2006); RMSE report |
| **Custom CSV Validation** | Upload your own radial penetration data; auto-normalised; RMSE computed |
| **Parameter Sweep** | 5×5 D × k grid; coverage heatmap; tornado sensitivity chart; CSV export |
| **Image → ρ(x,y)** | Histology PNG/JPG or MRI T1-post + T2/FLAIR → spatially-varying cell density |
| **Drug Database** | 24+ drug × tumor combinations, searchable and filterable by class and difficulty |
| **Results & Export** | Report summary, LaTeX methods snippet, Python notebook, CSV, PNG, PhysiCell JSON |
| **3 Delivery modes** | IV infusion / vascular ring / intratumoral injection |
| **3 Boundary PK profiles** | Constant IV / bolus IV (exponential decay) / timed infusion (ramp → plateau → decay) |

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | Vanilla HTML / CSS / JavaScript — single `index.html` |
| Fonts | Space Mono + Inter (Google Fonts) |
| PDE solver | Custom FTCS + upwind finite difference (pure JS) |
| Visualisation | HTML5 Canvas API |
| AI integration | Anthropic Claude API (`claude-sonnet-4`) |
| CORS proxy | Cloudflare Workers (free tier) |
| Hosting | GitHub Pages |
| Domain | pdeoncology.com |

No frameworks. No build tools. No server. One file.

---

## Project Structure

```
PDEOncology/
├── index.html          # Entire application (HTML + CSS + JS, bilingual)
├── CNAME               # Custom domain config
├── README.md           # This file
├── CITATION.cff        # Citation metadata
├── CONTRIBUTING.md     # Contribution guidelines
├── LICENSE             # GNU AGPL v3
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
3. Live at `https://yourusername.github.io/PDEOncology`

### Set up AI Drug Input (optional)

1. Get an API key from [console.anthropic.com](https://console.anthropic.com)
2. Deploy the Cloudflare Worker below as a CORS proxy
3. Update the worker URL in `index.html` (search for `workers.dev`)
4. Enter your key in the AI Drug Input tab

### Cloudflare Workers CORS Proxy

```js
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

## Model Validation

PDEOncology v0.5 includes an experimental validation tab comparing radial penetration curves against digitised spheroid data from peer-reviewed publications. Experimental points were extracted using WebPlotDigitizer methodology (±5–10% digitisation error). RMSE is computed in normalised concentration units.

| Agreement | RMSE threshold |
| --- | --- |
| Good | < 0.08 |
| Acceptable | < 0.15 |
| Poor | ≥ 0.15 |

Datasets: Thurber et al. 2008 (IgG antibody, small molecule), Minchinton & Tannock 2006 (doxorubicin spheroid). Poor agreement for Trastuzumab (large antibody) is a known model limitation — the binding-site barrier effect is not captured by the current linear uptake term.

> Experimental data points are approximate digitisations. Full quantitative validation would require raw tabular data directly from authors.

---

## Limitations

PDEOncology is an educational and exploratory tool. Key simplifications:

* Tumor geometry is circular and homogeneous — real tumors are irregular
* 2D model only — real drug penetration is three-dimensional
* Static cell density and diffusion fields — in reality these evolve with treatment
* IFP profile assumes uniform interstitial conductivity (linear radial flow) — real IFP gradients are more complex
* Normalised dimensionless parameters — not directly comparable to clinical doses
* Receptor binding kinetics approximated as linear uptake (`k·ρ·C`) — does not capture binding-site barrier for high-affinity antibodies
* FTCS is first-order in time — higher-order methods improve accuracy

> Results are model approximations. Not for clinical decision-making.

---

## Version History

| Version | Date | Highlights |
| --- | --- | --- |
| **v0.5** | Mar 2026 | IFP convection (Darcy's law), experimental validation, custom CSV upload, parameter sweep, two-compartment NP model, receptor expression slider, Stokes-Einstein particle size, MRI image upload, Python/LaTeX export |
| v0.4 | Mar 2026 | 20-frame animation, Compare tab, bilingual CN/EN, favicon, full About tab |
| v0.3 | Mar 2026 | UI overhaul — Space Mono/Inter, dark academic theme, mobile responsive |
| v0.2 | Mar 2026 | Cloudflare Worker, Claude API, AI fallback, drug DB, export |
| v0.1 | Mar 2026 | Initial — PDE solver, heatmap, radial curve, 3 delivery modes |

---

## References

1. Jain RK. Transport of molecules in the tumor interstitium. *Cancer Research*, 47(12):3039–3051, 1987.
2. Nugent LJ, Jain RK. Extravascular diffusion in normal and neoplastic tissues. *Cancer Research*, 44(1):238–244, 1984.
3. Thurber GM, Schmidt MM, Wittrup KD. Antibody tumor penetration. *Advanced Drug Delivery Reviews*, 60(12):1421–1434, 2008.
4. Chauhan VP et al. Delivery of molecular and nanoscale medicine to tumors. *Annual Review of Chemical and Biomolecular Engineering*, 2:281–298, 2011.
5. Tannock IF et al. Limited penetration of anticancer drugs through tumor tissue. *Clinical Cancer Research*, 8(3):878–884, 2002.
6. Minchinton AI, Tannock IF. Drug penetration in solid tumours. *Nature Reviews Cancer*, 6(8):583–592, 2006.
7. Kyle AH et al. Direct evidence of heterogeneous drug delivery and pharmacodynamics in solid tumors. *Journal of the National Cancer Institute*, 99(14):1118–1128, 2007.
8. Stylianopoulos T et al. Causes, consequences, and remedies for growth-induced solid stress in murine and human tumors. *PNAS*, 109(38):15101–15108, 2012.

---

## Team

**Y. Shi** — Technical Development, Website Engineering, UI Design, PDE Implementation

Designed and built the PDEOncology platform — including the finite difference PDE solver, interactive visualisation engine, Claude API integration, and the complete frontend interface.

**T. Yang** — Literature Research, Drug Database, Biophysical Modelling, System Co-development

Leads research and content development — curating the drug parameter database from published literature, sourcing biophysical references, optimisation of reaction-diffusion-convection algorithms, and the synthesis of multi-source experimental data.

---

## Related

* **[PDEOutreach](https://pdeoutreach.com)** — the public-facing companion platform. Cancer science explained for everyone, with interactive quizzes, risk profiles, and real patient stories.

---

## Citing PDEOncology

```
Yang ZT, Shi Y. PDEOncology: Tumor Drug Penetration Simulator using
Reaction-Diffusion-Convection PDEs with IFP Convection, 2026.
https://pdeoncology.com
```

A `CITATION.cff` file is included for automated citation in GitHub and Zenodo.

---

## License

This project is licensed under the **GNU Affero General Public License v3.0 (AGPL-3.0)**.

You are free to use, modify, and distribute this software under the following conditions:

* **Copyleft:** Any modified version must also be released under AGPL-3.0.
* **Network use:** If you run a modified version of this software as a web service (e.g. on a public server), you must make the complete corresponding source code available to users of that service.
* **Attribution:** Original authorship must be preserved in all copies and derivatives.

See the [LICENSE](https://github.com/SYM19/PDEOncology/blob/main/LICENSE) file for the full license text, or visit [gnu.org/licenses/agpl-3.0](https://www.gnu.org/licenses/agpl-3.0.html).

> AGPL-3.0 was chosen to ensure that this tool and its derivatives remain freely available as open-source software, including when deployed as a hosted web application.
