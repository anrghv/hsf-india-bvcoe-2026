# CMS-OD ML Hackathon: Variables and Physics Processes

---

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

---

## 2. The four physics processes

You have one **signal** and three **backgrounds**. The classifier must tell them apart.

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
- **Key traits:** **b-jets** (high `Jet_btagDeepFlavB`) and **many jets** (high `nJet`). This is why jet variables matter.
- **Analogy:** a very common, look-alike false alarm that has extra tell-tale features.

### Background 2: Non-resonant WW production

- Two W bosons are produced directly, with no Higgs involved. They decay the same way (lepton + neutrino each).
- Final state is **identical** to the signal. Hardest background to separate.
- **Key traits:** Differences are subtle: the leptons are **more spread out** in angle, and `mll`/`mth` distributions are broader.
- **Analogy:** an identical-looking input that differs only in statistical shape.

### Background 3: Drell-Yan (DY, Z/γ\* → ℓℓ)

- A Z boson (or virtual photon) decays into two leptons. Very high rate.
- No real neutrinos, so **little true missing momentum**. Apparent missing momentum comes from measurement error.
- **Key traits:** `mll` peaks near the Z mass (~91 GeV), leptons are back-to-back (large `dphill`), low `PuppiMET_pt`.
- **Analogy:** high-volume background noise with a clear spectral line.

### Summary table

| Process           | Real neutrinos? | b-jets / many jets? | Leptons close in angle? | Distinguishing feature         |
| ----------------- | --------------- | ------------------- | ----------------------- | ------------------------------ |
| **Signal (H→WW)** | Yes             | No                  | **Yes**                 | Small `dphill`, low `mll`      |
| **tt̄**            | Yes             | **Yes**             | No                      | High `nJet`, b-tag score       |
| **WW**            | Yes             | No                  | Less so                 | Subtle shape differences       |
| **Drell-Yan**     | No              | Rarely              | No (back-to-back)       | `mll` peak at ~91 GeV, low MET |

---

## 3. Variable definitions

### 3.1 Electron variables

| Variable                       | Plain-language meaning                                                                                                                                                                                               | Type       |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| `Electron_charge`              | Electric charge of the electron: +1 or −1                                                                                                                                                                            | Discrete   |
| `Electron_eta`                 | Direction angle relative to the beam axis (see primer)                                                                                                                                                               | Continuous |
| `Electron_phi`                 | Direction angle around the beam axis                                                                                                                                                                                 | Continuous |
| `Electron_pt`                  | Transverse momentum (energy-like "strength"), in GeV                                                                                                                                                                 | Continuous |
| `Electron_mvaFall17V2Iso_WP90` | Yes/no flag from a pre-trained ML electron identifier ("Fall17V2" version, "Iso" = also requires isolation, "WP90" = working point keeping 90% of real electrons). True means "looks like a real, isolated electron" | Boolean    |

### 3.2 Muon variables

| Variable       | Plain-language meaning                                              | Type       |
| -------------- | ------------------------------------------------------------------- | ---------- |
| `Muon_charge`  | Electric charge: +1 or −1                                           | Discrete   |
| `Muon_eta`     | Direction angle relative to the beam axis                           | Continuous |
| `Muon_phi`     | Direction angle around the beam axis                                | Continuous |
| `Muon_pt`      | Transverse momentum, in GeV                                         | Continuous |
| `Muon_tightId` | Yes/no quality flag: passes the strict muon identification criteria | Boolean    |

### 3.3 Jet variables

| Variable            | Plain-language meaning                                                                                                                                           | Type             |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `Jet_pt`            | Transverse momentum of the jet, in GeV                                                                                                                           | Continuous       |
| `Jet_eta`           | Direction angle relative to the beam axis                                                                                                                        | Continuous       |
| `Jet_phi`           | Direction angle around the beam axis                                                                                                                             | Continuous       |
|`Jet_btagDeepFlavB` | Score from 0 to 1 from a neural network (DeepJet/DeepFlavour) estimating if the jet came from a **b-quark**. Close to 1 = very likely b-jet. Key for spotting tt̄ | Continuous (0–1) |
| `Jet_jetId`         | Quality flag (integer code) indicating the jet passes basic checks, rejecting detector noise                                                                     | Discrete         |
| `Jet_puId`          | Flag (integer code) indicating whether the jet is likely from the main collision rather than pileup                                                              | Discrete         |

### 3.4 Missing momentum (neutrino proxy)

| Variable       | Plain-language meaning                                                                                                                                                                                                                                                                   | Type       |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| `PuppiMET_pt`  | **Missing transverse energy (MET).** Before the collision, transverse momentum sums to ~zero. If the visible particles don't balance out, something invisible (neutrinos) carried it away. This is the size of that imbalance, in GeV. "PUPPI" is an algorithm that removes pileup noise | Continuous |
| `PuppiMET_phi` | The direction (angle) of the missing momentum                                                                                                                                                                                                                                            | Continuous |

### 3.5 Engineered (high-level) variables

Built by combining the basic ones above. These encode physics knowledge and are usually the most powerful inputs.

| Variable | Plain-language meaning                                                                                                                                                                          | Why it helps                                                           |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `mll`    | **Invariant mass of the two leptons.** A combined "mass" computed from their energies and angles. If two leptons come from one parent particle, this reflects the parent's mass                 | Z→ℓℓ (Drell-Yan) peaks at ~91 GeV; signal sits lower                   |
| `ptll`   | Transverse momentum of the **two-lepton system** combined (vector sum), in GeV                                                                                                                  | Differs by production mechanism and recoil against jets                |
| `dphill` | **Angle difference in φ between the two leptons** (0 to π). 0 = same direction, π = back-to-back                                                                                                | Signal leptons are close together (small), a strong discriminator      |
| `mth`    | **Transverse mass of the Higgs candidate**: uses the two leptons and the missing momentum. Like an invariant mass, but only with transverse components, since neutrinos can't be fully measured | Signal clusters below the Higgs mass (125 GeV), backgrounds spread out |
| `mtw2`   | **Transverse mass of the second (lower-pT) lepton with the missing momentum.** Approximates the mass of one W boson                                                                             | Checks consistency with a W decay                                      |
| `mjj`    | **Invariant mass of the two leading jets.** Zero or a default value if fewer than two jets                                                                                                      | Separates event topologies, relevant for jet-heavy backgrounds like tt̄ |
| `nJet`   | **Number of jets** in the event                                                                                                                                                                 | tt̄ usually has several jets, signal has few                            |

---

## 4. Practical tips

- **Booleans/flags** (`..._WP90`, `tightId`, `jetId`, `puId`) are mostly used to **select good objects** in preprocessing, not necessarily as classifier inputs.
- **Raw angles** (`phi`) are less useful on their own. What matters is the **difference** between objects (`dphill`), which is why engineered variables help.
- **Mass of leptons** (`Electron_mass`, `Muon_mass`) is nearly constant, so expect almost no information. Good example for a feature importance discussion.
- **Variable-length data:** Events have different numbers of leptons and jets. Decide how to handle it (e.g. take the leading two leptons, the leading two jets, pad with a default value).
- **Check feature correlations:** `mll`, `mth`, `mtw2` and `ptll` are strongly related. This links directly to your correlation matrix task.
