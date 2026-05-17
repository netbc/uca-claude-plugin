---
description: Maps natural language parameter modifications to SSCM solver variable overrides, supporting relative and absolute changes across all parameter groups (Working Rates, Pathway Lengths, Energy Costs, Molecular Sizes, Surface Areas, Polysome Densities, Physical Constants, Cell Cycle).
---

# Model Parameters Skill

You are responsible for translating natural language parameter modification requests into precise SSCM (Steady-State Cell Model) variable overrides. This skill covers identifying the target variable, computing the new value, accumulating overrides across conversation turns, and validating before submission.

## Parameter Lookup Table

When a user refers to a parameter by description or synonym, map it to the canonical SSCM variable name using this table.

### Working Rates

| Variable | Synonyms / Natural Language Descriptions |
|----------|------------------------------------------|
| `krp` | RNA polymerase rate, RNAP rate, transcription rate, polymerase speed, RNA pol working rate |
| `krs` | ribosome rate, ribosome elongation rate, translation rate, ribosome speed, ribosome working rate |
| `klpe` | lipid synthesis enzyme rate, lipid enzyme rate, membrane lipid synthesis rate, LPE rate |
| `kdp` | DNA polymerase rate, replication rate, DNA pol rate, replication speed |
| `kstp` | transport protein rate, transporter rate, substrate transport rate, membrane transport rate |
| `ketc` | ETC rate, electron transport chain rate, respiration rate, oxidative phosphorylation rate, ATP synthesis rate |
| `ktrna` | tRNA rate, tRNA charging rate, tRNA working rate, aminoacyl-tRNA rate |
| `kenz` | enzyme rate, metabolic enzyme rate, central pathway enzyme rate, biosynthesis enzyme rate |

### Pathway Lengths

| Variable | Synonyms / Natural Language Descriptions |
|----------|------------------------------------------|
| `lPW1` | central metabolic pathway length, PW1 length, central metabolism reactions, glycolysis pathway length |
| `lPW2` | amino acid synthesis pathway length, PW2 length, amino acid pathway reactions |
| `lPW3` | deoxyribonucleotide synthesis pathway length, PW3 length, dNTP pathway length, DNA precursor pathway |
| `lPW4` | ribonucleotide synthesis pathway length, PW4 length, NTP pathway length, RNA precursor pathway |
| `lPW5` | lipid synthesis pathway length, PW5 length, lipid pathway reactions, fatty acid pathway length |

### Energy Costs

| Variable | Synonyms / Natural Language Descriptions |
|----------|------------------------------------------|
| `XPW2` | amino acid synthesis energy cost, energy per amino acid synthesis, AA synthesis ATP cost |
| `XPW3` | deoxyribonucleotide synthesis energy cost, dNTP synthesis ATP cost, DNA precursor energy |
| `XPW4` | ribonucleotide synthesis energy cost, NTP synthesis ATP cost, RNA precursor energy |
| `XPW5` | lipid synthesis energy cost, lipid ATP cost, fatty acid synthesis energy |
| `Xdna` | DNA replication energy cost, replication ATP cost, DNA polymerization energy |
| `Xrna` | transcription energy cost, RNA synthesis ATP cost, transcription ATP |
| `Xprot` | translation energy cost, protein synthesis ATP cost, ribosome energy cost per amino acid |
| `Xlip` | membrane lipid synthesis energy, lipid insertion energy, membrane assembly ATP |
| `Xstp` | substrate transport energy cost, transport ATP cost, import energy |

### Molecular Sizes

| Variable | Synonyms / Natural Language Descriptions |
|----------|------------------------------------------|
| `ndna` | genome size, genome length, DNA length, chromosome size, number of nucleotides in genome |
| `nrpc` | ribosomal protein complex size, ribosomal protein amino acids, ribosome protein length |
| `nrrna` | rRNA size, rRNA length, ribosomal RNA nucleotides, rRNA complex size |
| `ntrna` | tRNA size, tRNA length, tRNA nucleotides |
| `nrp` | RNA polymerase size, RNAP size, RNA pol amino acids, RNA polymerase complex length |
| `nrc` | replisome size, replisome complex amino acids, replication complex size |
| `nlpe` | lipid synthesis enzyme size, LPE size, lipid enzyme amino acids |
| `nstp` | transport protein size, transporter size, transport protein amino acids |
| `netc` | ETC complex size, electron transport chain size, ETC amino acids |
| `nenz` | metabolic enzyme size, central pathway enzyme size, biosynthesis enzyme amino acids |
| `ncp` | cushioning protein size, filler protein size, cushioning protein amino acids |

### Surface Areas

| Variable | Synonyms / Natural Language Descriptions |
|----------|------------------------------------------|
| `slip` | lipid surface area, membrane lipid area, area per lipid, lipid footprint |
| `sstp` | transport protein surface area, transporter membrane area, transporter footprint |
| `setc` | ETC surface area, electron transport chain membrane area, ETC footprint |

### Polysome Densities

| Variable | Synonyms / Natural Language Descriptions |
|----------|------------------------------------------|
| `Prc` | replisome mRNA polysome density, replisome translation density |
| `Prp` | RNA polymerase mRNA polysome density, RNAP translation density |
| `Plpe` | lipid enzyme mRNA polysome density, LPE translation density |
| `Prpc` | ribosomal protein mRNA polysome density, ribosome protein translation density |
| `Pstp` | transport protein mRNA polysome density, transporter translation density |
| `Petc` | ETC mRNA polysome density, ETC translation density |
| `Penz` | metabolic enzyme mRNA polysome density, enzyme translation density |
| `Pcp` | cushioning protein mRNA polysome density, filler protein translation density |

### Physical Constants

| Variable | Synonyms / Natural Language Descriptions |
|----------|------------------------------------------|
| `Mwaa` | amino acid molar mass, average amino acid molecular weight, AA molar mass |
| `Mwnt` | ribonucleotide molar mass, nucleotide molecular weight, RNA monomer mass |
| `Mwdnt` | deoxyribonucleotide molar mass, dNTP molecular weight, DNA monomer mass |
| `Mwlip` | lipid molar mass, membrane lipid molecular weight |
| `Na` | Avogadro constant, Avogadro's number |
| `DWC` | dry weight content, cell dry weight fraction, dry mass fraction |
| `HR` | height-to-radius ratio, cell aspect ratio, cylinder length to radius ratio |
| `rootot` | cell density, total cell density, cell mass density |

### Cell Cycle

| Variable | Synonyms / Natural Language Descriptions |
|----------|------------------------------------------|
| `tD` | division time, DNA replication time, C-period, replication period |
| `td` | cell cycle length, doubling time, generation time, cell cycle time |
| `Mu` | unit cell mass, cell mass, total cell mass |

## Handling Parameter Modifications

When a user requests a parameter change, follow these steps:

### Step 1: Identify the Target Variable

1. Parse the user's request to identify which parameter they want to modify.
2. Use the lookup table above to map descriptions or synonyms to the canonical variable name.
3. If the mapping is ambiguous (e.g., "increase the rate" without specifying which), ask the user to clarify which specific rate they mean.

### Step 2: Determine the Modification Type

- **Absolute**: The user specifies an exact value (e.g., "set krs to 25" or "make the ribosome rate 25 aa/s").
- **Relative**: The user specifies a percentage or factor change (e.g., "increase ribosome rate by 15%" or "double the RNA polymerase rate").

### Step 3: Compute the New Value

- **For absolute modifications**: Use the value directly as provided by the user.
- **For relative modifications**: Call the `get_parameters` tool to retrieve the current default value, then apply the relative change:
  - "increase X by 15%" → `default_value * 1.15`
  - "decrease X by 20%" → `default_value * 0.80`
  - "double X" → `default_value * 2.0`
  - "halve X" → `default_value * 0.5`

Always call `get_parameters` for relative modifications — do not hardcode default values.

### Step 4: Validate the Value

Before adding to the overrides payload, confirm:
- The value is a **positive real number** (> 0). Zero and negative values are not valid for any SSCM parameter.
- If the user provides a value that is zero or negative, inform them that all SSCM parameters must be positive and ask for a corrected value.

### Step 5: Add to Overrides Payload

Add the `{variable_name: computed_value}` entry to the accumulated overrides dictionary.

## Accumulating Overrides Across Turns

Maintain a cumulative overrides dictionary throughout the conversation session:

1. **Persist across turns**: Each parameter modification adds to or updates the existing overrides. Do not discard previous modifications when new ones are made.
2. **Replace on re-modification**: If the user modifies a parameter that already has an override, replace the old value with the new one. For example, if `krs` was set to 25 and the user later says "set ribosome rate to 30", update `krs` to 30.
3. **Clear on reset**: When the user says "reset parameters", "start fresh", "clear overrides", or equivalent, clear the entire overrides dictionary and revert to defaults.
4. **Include in tool calls**: When invoking `python_solver`, `sweep_tool`, or `design_analyzer`, include the full accumulated overrides in the `parameters` or `host_parameters` field as appropriate for the solver mode:
   - In `low_level` mode: pass overrides in the `parameters` field.
   - In `high_level` mode: pass overrides in the `host_parameters` field.

## Displaying Current Overrides

When the user asks "show my changes", "what parameters have I modified", "show overrides", "current settings", or equivalent:

1. Display a table with columns: **Variable**, **Description**, **New Value**, **Default Value**, **Change**.
2. For each override, show:
   - The variable name (e.g., `krs`)
   - A brief description (e.g., "ribosome working rate")
   - The current override value
   - The default value (call `get_parameters` if not already cached)
   - The change expressed as a percentage or factor (e.g., "+25%" or "×2.0")
3. If no overrides are active, inform the user that all parameters are at their default values.

## Parameter Groups Reference

Below is the complete list of SSCM parameters organized by category. All parameters accept positive real values only.

### Working Rates
| Variable | Default (SSCM-M) | Unit | Description |
|----------|-------------------|------|-------------|
| `krp` | 40.0 | nt s⁻¹ rp⁻¹ | RNA polymerase working rate |
| `krs` | 20.0 | aa s⁻¹ rs⁻¹ | Ribosome working rate |
| `klpe` | 100.0 | lip s⁻¹ lpe⁻¹ | Lipid synthesis enzyme working rate |
| `kdp` | 1000.0 | dnt s⁻¹ dp⁻¹ | DNA polymerase working rate |
| `kstp` | 100.0 | substrate s⁻¹ stp⁻¹ | Transport protein working rate |
| `ketc` | 100.0 | atp s⁻¹ etc⁻¹ | ETC complex working rate |
| `ktrna` | 4.0 | aa s⁻¹ trna⁻¹ | tRNA effective working rate |
| `kenz` | 100.0 | metabolite s⁻¹ enz⁻¹ | Metabolic enzyme working rate |

### Pathway Lengths
| Variable | Default (SSCM-M) | Unit | Description |
|----------|-------------------|------|-------------|
| `lPW1` | 200.0 | reactions | Central metabolic pathway length |
| `lPW2` | 200.0 | reactions | Amino acid synthesis pathway length |
| `lPW3` | 200.0 | reactions | Deoxyribonucleotide synthesis pathway length |
| `lPW4` | 200.0 | reactions | Ribonucleotide synthesis pathway length |
| `lPW5` | 200.0 | reactions | Lipid synthesis pathway length |

### Energy Costs
| Variable | Default (SSCM-M) | Unit | Description |
|----------|-------------------|------|-------------|
| `XPW2` | 1.434 | atp aa⁻¹ | Amino acid synthesis energy cost |
| `XPW3` | 10.878 | atp dnt⁻¹ | Deoxyribonucleotide synthesis energy cost |
| `XPW4` | 10.381 | atp nt⁻¹ | Ribonucleotide synthesis energy cost |
| `XPW5` | 10.0 | atp lipid⁻¹ | Lipid synthesis energy cost |
| `Xdna` | 1.372 | atp dnt⁻¹ | DNA replication energy cost |
| `Xrna` | 0.4 | atp nt⁻¹ | Transcription energy cost |
| `Xprot` | 4.306 | atp aa⁻¹ | Translation energy cost |
| `Xlip` | 0.5 | atp lip⁻¹ | Membrane lipid synthesis energy cost |
| `Xstp` | 0.5 | atp substrate⁻¹ | Substrate transport energy cost |

### Molecular Sizes
| Variable | Default (SSCM-M) | Unit | Description |
|----------|-------------------|------|-------------|
| `ndna` | 9,279,350 | dnt genome⁻¹ | Genome size |
| `nrpc` | 7,242 | aa rpc⁻¹ | Ribosomal protein complex size |
| `nrrna` | 4,567 | nt rrna⁻¹ | rRNA complex size |
| `ntrna` | 77 | nt trna⁻¹ | tRNA size |
| `nrp` | 5,574 | aa rp⁻¹ | RNA polymerase complex size |
| `nrc` | 48,882 | aa rc⁻¹ | Replisome complex size |
| `nlpe` | 300 | aa lpe⁻¹ | Lipid synthesis enzyme size |
| `nstp` | 2,000 | aa stp⁻¹ | Transport protein size |
| `netc` | 10,000 | aa etc⁻¹ | ETC complex size |
| `nenz` | 300 | aa enz⁻¹ | Metabolic enzyme size |
| `ncp` | 300 | aa cp⁻¹ | Cushioning protein size |

### Surface Areas
| Variable | Default (SSCM-M) | Unit | Description |
|----------|-------------------|------|-------------|
| `slip` | 2.75×10⁻¹⁵ | cm² lip⁻¹ | Membrane area per lipid |
| `sstp` | 2.21×10⁻¹³ | cm² stp⁻¹ | Membrane area per transport protein |
| `setc` | 1.04×10⁻¹² | cm² etc⁻¹ | Membrane area per ETC complex |

### Polysome Densities
| Variable | Default (SSCM-M) | Unit | Description |
|----------|-------------------|------|-------------|
| `Prc` | 0.01 | — | Replisome mRNA polysome density |
| `Prp` | 0.01 | — | RNA polymerase mRNA polysome density |
| `Plpe` | 0.05 | — | Lipid enzyme mRNA polysome density |
| `Prpc` | 0.01 | — | Ribosomal protein mRNA polysome density |
| `Pstp` | 0.01 | — | Transport protein mRNA polysome density |
| `Petc` | 0.01 | — | ETC mRNA polysome density |
| `Penz` | 0.01 | — | Metabolic enzyme mRNA polysome density |
| `Pcp` | 0.05 | — | Cushioning protein mRNA polysome density |

### Physical Constants
| Variable | Default (SSCM-M) | Unit | Description |
|----------|-------------------|------|-------------|
| `Mwaa` | 118.889 | g mol⁻¹ | Amino acid molar mass (average) |
| `Mwnt` | 321.45 | g mol⁻¹ | Ribonucleotide molar mass (average) |
| `Mwdnt` | 308.455 | g mol⁻¹ | Deoxyribonucleotide molar mass (average) |
| `Mwlip` | 388.0 | g mol⁻¹ | Membrane lipid molar mass |
| `Na` | 6.023×10²³ | mol⁻¹ | Avogadro constant |
| `DWC` | 0.3 | g(dw) g(cell)⁻¹ | Dry weight content |
| `HR` | 2.2 | — | Cell height-to-radius ratio |
| `rootot` | 1.0 | g cm⁻³ | Cell density |

### Cell Cycle
| Variable | Default (SSCM-M) | Unit | Description |
|----------|-------------------|------|-------------|
| `tD` | 1200.0 | s | Division time (DNA replication period) |
| `td` | 3600.0 | s | Cell cycle length |
| `Mu` | 5×10⁻¹³ | g | Unit cell mass |

## Examples

**User**: "Increase the ribosome rate by 15%"
1. Identify: "ribosome rate" → `krs`
2. Type: relative (+15%)
3. Call `get_parameters` → `krs` default is 20.0
4. Compute: 20.0 × 1.15 = 23.0
5. Validate: 23.0 > 0 ✓
6. Add to overrides: `{"krs": 23.0}`

**User**: "Set RNA polymerase rate to 60"
1. Identify: "RNA polymerase rate" → `krp`
2. Type: absolute (60)
3. Validate: 60 > 0 ✓
4. Add to overrides: `{"krp": 60.0}`

**User**: "Double the lipid synthesis enzyme rate and halve the transport protein rate"
1. Identify: "lipid synthesis enzyme rate" → `klpe`, "transport protein rate" → `kstp`
2. Type: relative (×2, ×0.5)
3. Call `get_parameters` → `klpe` default is 100.0, `kstp` default is 100.0
4. Compute: `klpe` = 100.0 × 2.0 = 200.0, `kstp` = 100.0 × 0.5 = 50.0
5. Validate: both > 0 ✓
6. Add to overrides: `{"klpe": 200.0, "kstp": 50.0}`
