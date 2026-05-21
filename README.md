# neoBIM Technical Assessment — LCA, ÖKOBAUDAT & BIM 


---

# Repository Structure

```text
NeoBIM-Assignment/
│
├── README.md
│
├── Q1_Load-Bearing Wall.html
│
├── Q2_Pipeline_Process.html
├── Q2_Pipeline_Process.ipynb
├── Q2_simpler_Pipeline_Process.ipynb
├── Q2_simpler_Pipeline_Process.html
├── README_Q2.md
│
├── Q3_GWP_Studio_eLCA(5).html
├── eLCA(5).pdf
```

---

# Assignment Overview

The submission contains solutions for the following three tasks:

| Task | Description |
|---|---|
| Q1 | LCA estimate for a load-bearing wall with U-value ≈ 0.20 W/m²K |
| Q2 | AI-assisted ÖKOBAUDAT material matching pipeline |
| Q3 | GWP estimation for a minimal 25 m² studio using eLCA |

---

# Q1 — Load-Bearing Wall LCA

## Objective

Estimate the environmental impacts of a load-bearing external wall using ÖKOBAUDAT datasets.

## Scope

- Functional unit: **1 m² external wall**
- Lifecycle stages: **A1–C3**
- Database: **ÖKOBAUDAT OBD_2024_I_A2**
- Target thermal performance:
  
```math
U \leq 0.20 \text{ W/m²K}
```

## Assumed Wall Build-Up

| Layer | Material | Thickness |
|---|---|---|
| 1 | Gypsum plaster | 15 mm |
| 2 | Brick masonry | 240 mm |
| 3 | Mineral wool insulation | 160 mm |
| 4 | Lime-cement render | 20 mm |

Total wall thickness:

```math
435 \text{ mm}
```

---

## Results

| Indicator | Result |
|---|---:|
| GWP | 39.38 kg CO₂-eq/m² |
| PENRT | 456.25 MJ/m² |
| PERT | 105.05 MJ/m² |
| Verified U-value | 0.196 W/m²K |

---

## Notes

- Results are based on selected ÖKOBAUDAT EPD datasets.
- Missing C3 values were treated as zero where unavailable.
- Module D was excluded.
- Values represent environmental impacts, not monetary costs.

---

# Q2 — AI-Assisted ÖKOBAUDAT Material Matching

## Objective

Develop an automated workflow that maps generic BIM elements to matching Environmental Product Declarations (EPDs) from ÖKOBAUDAT.

---

## Input Example

```json
{
  "name": "Generic Roof",
  "elementType": "Roof",
  "thickness": 200
}
```

---

# Pipeline Overview

The workflow performs the following steps automatically:

1. Decompose building elements into material layers
2. Query ÖKOBAUDAT REST API
3. Rank EPD candidates using LLM reasoning
4. Verify assembly against German construction standards
5. Generate structured outputs and reports

---

# Technologies Used

- Python
- Jupyter Notebook
- Claude API
- ÖKOBAUDAT REST API
- Google Colab
- JSON reporting
- Markdown generation

---

# Standards Referenced

| Standard | Purpose |
|---|---|
| DIN 4108-2 | Thermal insulation |
| DIN 4108-3 | Moisture protection |
| DIN 18531 | Flat roof waterproofing |
| EN ISO 6946 | Thermal resistance calculation |
| GEG 2024 | German Building Energy Act |

---

# Output

The pipeline generates:

- structured JSON outputs
- material layer decomposition
- EPD matches
- compliance verification
- markdown reports

---

# Additional Technical Documentation

Detailed documentation for the pipeline is available in:

- `README_Q2.md`
- `Q2_Pipeline_Process.html`

This includes:

- pipeline architecture
- standards-based decomposition
- API workflow
- EPD ranking logic
- implementation details
- debugging and fixes

---

# Q3 — GWP Estimation for 25 m² Studio

## Objective

Estimate the embodied carbon footprint of a minimal 25 m² studio apartment using eLCA.

---

# Scope

Included:

- A1–A3
- C3
- C4
- Maintenance

Excluded:

- operational energy
- HVAC systems
- furniture
- windows and doors
- MEP systems
- Module D

---

# Model Assumptions

| Element | Assumption |
|---|---|
| Studio area | 25 m² |
| Service life | 50 years |
| Exterior walls | Timber frame |
| Roof | Flat roof |
| Floor | Reinforced concrete slab |

Only the structural shell was modelled.

---

# Results

| Indicator | Result |
|---|---:|
| GWP total | 5.731 kg CO₂-eq/m² NGF |
| Total studio GWP | 143.3 kg CO₂-eq |
| PENRT | 57.65 MJ/m² NGF |
| PERT | 35.19 MJ/m² NGF |

---

# Key Learning

This assessment demonstrates how:

- BIM data can be linked to environmental datasets
- AI can support LCA automation
- standards-based reasoning can improve material matching
- embodied carbon workflows can be structured programmatically

---

# Tools Used

- ÖKOBAUDAT
- eLCA / Bauteileditor
- Python
- Jupyter Notebook
- Claude API
- REST APIs
- HTML/PDF reporting

---

# How to Run the Notebook

## 1. Open Notebook

Open:

```text
okobaudat_pipeline.ipynb
```

in:

- Google Colab
- Jupyter Notebook

---

## 2. Install Dependencies

Install required packages:

```bash
pip install anthropic requests rich
```

---

## 3. Add API Key

Set your Anthropic API key as an environment variable or Colab Secret.

Example:

```python
import os
os.environ["ANTHROPIC_API_KEY"] = "YOUR_KEY"
```

---

## 4. Run the Pipeline

Edit the input element:

```python
element = {
    "name": "External Wall",
    "elementType": "Wall",
    "thickness": 300
}
```

Then run all cells.

---

# Notes for Reviewers

This submission is intended as a technical demonstration and focuses on:

- transparent assumptions
- reproducible workflows
- AI-assisted sustainability automation
- BIM-to-LCA integration logic
- structured environmental reporting

The objective is not only to calculate impacts, but also to demonstrate scalable workflows for future BIM-integrated sustainability systems.

---

# Author

Ann Rose Manavalan Paul

Technical Assessment Submission for neoBIM GmbH
