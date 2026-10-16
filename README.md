# hsf-india-bvcoe-2026
# CMS-OD ML Hackathon: Variables and Physics Processes

---

## Weights you can use:

sample Higgs; weight = 0.010778

sample DY; weight = 1.588077

sample WW; weight = 0.067506

sample Top; weight = 0.112716


## 1. Quick primer

- **LHC / CMS:** Protons collide at nearly the speed of light. The CMS detector is a huge "camera" that records what flies out. Each collision is an **event** (one row in your dataset).
- **Goal:** Find rare events where a **Higgs boson** was produced and decayed to two **W bosons**. Most events are something else, so this is a **signal vs background classification** problem.
- **Units:** Energy, momentum and mass are in **GeV** (giga-electron-volts). Treat as "just a number with a scale".
- **Coordinates (see below):** The beam runs along the z-axis. We describe directions with two angles, **φ** and **η**.

| Term           | Meaning                                                                                                                                 |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Beam axis**  | The line the protons travel along                                                                                                       |
| **Transverse** | Perpendicular to the beam. Momentum there is known to be ~zero before the collision, so any imbalance means something invisible escaped |
| **φ (phi)**    | Angle around the beam axis, from −π to +π (like a compass heading)                                                                      |
| **η (eta)**    | Angle from the beam axis. η = 0 is perpendicular to the beam; large \|η\| is close to the beam line                                     |
| **pT**         | Momentum transverse to the beam (GeV). High pT means energetic, usually from a heavy particle decay                                     |
| **Lepton**     | Here: an electron or a muon (clean, easy to detect)                                                                                     |
| **Jet**        | A spray of particles from a quark or gluon, reconstructed as one object                                                                 |
| **Neutrino**   | Invisible to the detector. Shows up only as missing momentum                                                                            |
| **Pileup**     | Extra unrelated collisions happening at the same time (noise)                                                                           |
| **Leading / subleading** | Objects sorted by pT. Leading = highest pT, subleading = second highest                                                       |

---

## 2. What is in the dataset (event selection)

The files you receive are already **pre-filtered**. Only events with an **electron + muon pair** (the "eμ channel") are kept:

| Requirement | Detail |
| ----------- | ------ |
| Lepton count | Exactly **2 tight leptons**: one electron and one muon |
| Charge | **Opposite** electric charge |
| Lepton quality | Electron passes the tight ID flag. Muon passes tight ID and is isolated (`pfRelIso04_all < 0.15`) |
| Lepton pT | Leading lepton > 25 GeV. Subleading: electron > 13 GeV, muon > 10 GeV |
| Lepton η | Electron \|η\| < 2.5, muon \|η\| < 2.4 (inside the detector's precise region) |
| Good jets | Pass quality and pileup-rejection flags, \|η\| < 4.7, and are **not overlapping a lepton** (ΔR > 0.4). Only jets with **pT ≥ 30 GeV** are counted |

**Why eμ?** Requiring one electron and one muon removes most of the huge same-flavour Drell-Yan background (Z → ee or μμ). This is a standard, clean choice in H→WW analyses.

**Lepton1 / Lepton2** are ordered by pT, not by flavour. Lepton1 can be an electron or a muon.

---

## 3. The four physics processes

You have one **signal** and three **backgrounds**. The classifier must tell them apart. All four end up as **one electron + one muon + invisible neutrinos**, which is why the problem is hard.

### Signal: Higgs → WW → ℓνℓν

- A Higgs boson (made mostly by two gluons fusing) decays into two W bosons.
- Each W decays into a **lepton + neutrino**. Final state: **two leptons + invisible neutrinos (missing momentum)**.
- **Key traits:**
  - The two leptons fly off **close together in angle** (small `dphill`), because the Higgs has no spin and the W spins are correlated.
  - **Low dilepton mass** (`mll` small), well below the Higgs mass, since neutrinos carry energy away.
  - Few or no jets.
- **Analogy:** a rare, specific fault signature in a huge log file.

### Background 1: Top quark pair (tt̄)

- A top and an anti-top quark are produced; each decays to a W and a **b-quark jet**.
- Final state also has **two leptons + neutrinos**, so it mimics the signal.
- **Key traits:** **many jets** (high `njet`, `jet_cat` = 1 or 2), jets with high pT, and large missing momentum.
- **Analogy:** a very common, look-alike false alarm that has extra tell-tale features.

### Background 2: Non-resonant WW production

- Two W bosons are produced directly, with no Higgs involved. They decay the same way (lepton + neutrino each).
- Final state is **identical** to the signal. Hardest background to separate.
- **Key traits:** Differences are subtle: the leptons are **more spread out** in angle, and `mll`/`mth` distributions are broader.
- **Analogy:** an identical-looking input that differs only in statistical shape.

### Background 3: Drell-Yan to tau pairs (DY, Z/γ\* → ττ)

- A Z boson (or virtual photon) decays into two **tau leptons**. Each tau then decays further, here into an electron or a muon plus neutrinos. This is how Drell-Yan survives the eμ selection.
- Very high production rate.
- **Key traits:** The neutrinos come from the tau decays, so the missing momentum tends to **point along the leptons** rather than away from them. That tends to give **small `mth` and `mtw2`**. Few jets.
- **Analogy:** high-volume background noise that survives the filter through a side door.

### Summary table

| Process           | Real neutrinos? | Many jets?  | Leptons close in angle? | Distinguishing feature                         |
| ----------------- | --------------- | ----------- | ----------------------- | ---------------------------------------------- |
| **Signal (H→WW)** | Yes             | No          | **Yes**                 | Small `dphill`, low `mll`                      |
| **tt̄**            | Yes             | **Yes**     | No                      | High `njet`, high jet pT                       |
| **WW**            | Yes             | No          | Less so                 | Subtle shape differences                       |
| **Drell-Yan (ττ)** | Yes (from tau decays) | Rarely | Varies              | Small `mth`/`mtw2`, MET aligned with leptons   |

---

## 4. Variable definitions (the 22 features)

### 4.1 Lepton variables

Ordered by pT. Lepton1 = highest pT, Lepton2 = second highest.

| Variable       | Plain-language meaning                                                      | Type       |
| -------------- | --------------------------------------------------------------------------- | ---------- |
| `Lepton1_pt`   | Transverse momentum of the leading lepton, in GeV                           | Continuous |
| `Lepton1_eta`  | Direction of the leading lepton relative to the beam axis                   | Continuous |
| `Lepton1_phi`  | Direction of the leading lepton around the beam axis                        | Continuous |
| `Lepton2_pt`   | Transverse momentum of the subleading lepton, in GeV                        | Continuous |
| `Lepton2_eta`  | Direction of the subleading lepton relative to the beam axis                | Continuous |
| `Lepton2_phi`  | Direction of the subleading lepton around the beam axis                     | Continuous |

### 4.2 Jet variables

Only good jets with pT ≥ 30 GeV are used. Jet1 = highest pT, Jet2 = second highest.

| Variable    | Plain-language meaning                                                    | Type       |
| ----------- | ------------------------------------------------------------------------- | ---------- |
| `Jet1_pt`   | Transverse momentum of the leading jet, in GeV                            | Continuous |
| `Jet1_eta`  | Direction of the leading jet relative to the beam axis                    | Continuous |
| `Jet1_phi`  | Direction of the leading jet around the beam axis                         | Continuous |
| `Jet2_pt`   | Transverse momentum of the subleading jet, in GeV                         | Continuous |
| `Jet2_eta`  | Direction of the subleading jet relative to the beam axis                 | Continuous |
| `Jet2_phi`  | Direction of the subleading jet around the beam axis                      | Continuous |

> **Missing jets:** If the event has fewer jets than the slot needs, the value is the placeholder **`-999`**. For example, an event with one jet has `Jet2_pt = Jet2_eta = Jet2_phi = -999`. This is *not* a physical value. See the tips section on how to handle it.

### 4.3 Missing momentum (neutrino proxy)

| Variable       | Plain-language meaning                                                                                                                                                                                                                                                                   | Type       |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| `PuppiMET_pt`  | **Missing transverse energy (MET).** Before the collision, transverse momentum sums to ~zero. If the visible particles don't balance out, something invisible (neutrinos) carried it away. This is the size of that imbalance, in GeV. "PUPPI" is an algorithm that removes pileup noise | Continuous |
| `PuppiMET_phi` | The direction (angle) of the missing momentum                                                                                                                                                                                                                                            | Continuous |

### 4.4 Engineered (high-level) variables

Built by combining the basic ones above. These encode physics knowledge and are usually the most powerful inputs.

| Variable  | Plain-language meaning                                                                                                                                                                          | Why it helps                                                           |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `mll`     | **Invariant mass of the two leptons.** A combined "mass" computed from their energies and angles. If two leptons come from one parent particle, this reflects the parent's mass                 | Signal sits at low values, backgrounds have different shapes           |
| `ptll`    | Transverse momentum of the **two-lepton system** combined (vector sum), in GeV                                                                                                                  | Differs by production mechanism and recoil against jets                |
| `dphill`  | **Difference in φ between Lepton1 and Lepton2**, wrapped to the range −π to +π. 0 = same direction, ±π = back-to-back. It is *signed*, so the sign depends on lepton ordering                   | Signal leptons are close together (value near 0), a strong discriminator |
| `mth`     | **Transverse mass of the Higgs candidate**: uses the two-lepton system and the missing momentum. Like an invariant mass, but only with transverse components, since neutrinos can't be fully measured | Signal clusters below the Higgs mass (125 GeV), backgrounds spread out |
| `mtw2`    | **Transverse mass of Lepton2 (the lower-pT lepton) with the missing momentum.** Approximates the mass of one W boson                                                                            | Checks consistency with a W decay                                      |
| `mjj`     | **Invariant mass of the two highest-pT good jets.** `0` if the event has fewer than two good jets                                                                                               | Separates event topologies, relevant for jet-heavy backgrounds like tt̄ |
| `njet`    | **Number of good jets** with pT ≥ 30 GeV                                                                                                                                                        | tt̄ usually has several jets, signal has few                            |
| `jet_cat` | **Jet category** (integer): `0` = no jet above 30 GeV, `1` = exactly one, `2` = two or more                                                                                                     | Analysts split the signal region by jet count, since each behaves differently |

---

## 5. Practical tips

- **Placeholder values (`-999`):** Jet slots without a jet are `-999`. Neural networks cannot treat that as a real number (it would dominate the scaling). Options: replace with 0 and add a "has jet" flag, or rely on `njet`/`jet_cat`. BDTs (XGBoost) handle it fine because they only split on thresholds.
- **`mjj` uses `0`, not `-999`:** For events with fewer than two jets `mjj = 0`. Treat it consistently with the jet slots.
- **Raw angles (`phi`) are less useful on their own.** What matters is the **difference** between objects (`dphill`), which is why engineered variables help. Absolute φ should look flat for every process (the detector has no preferred direction).
- **Feature scaling:** Run a standard scaler for the DNN/GNN. Not needed for BDTs.
- **Check feature correlations:** `mll`, `mth`, `mtw2` and `ptll` are strongly related. This links directly to your correlation matrix task.
- **Compare raw vs engineered:** Train once on only the raw variables (pT, η, φ of leptons, jets, MET) and once with the engineered ones. Does the model rediscover `mll` and `dphill` on its own?
