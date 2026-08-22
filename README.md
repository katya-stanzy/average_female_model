# Average Female OpenSim Model

A population-average female musculoskeletal model for OpenSim, derived from
MRI-based personalised models of 25 adult women and symmetrised left/right.
The model is of the Rajagopal lineage (Rajagopal et al. 2016; Lai et al. 2017;
Uhlrich et al. 2022) and is intended as a generic scaling template for gait
analysis in female cohorts, in place of the standard male-derived generic model.

**Full methods, validation and all reported tests are in [PREPRINT, in
preparation].** This file describes only the released files and how they differ
from one another.

Tested with **OpenSim 4.5.2**.

---

## The four files

The release contains four variants of the *same* model. All four share
identical bone geometry, joint definitions, muscle paths and wrap objects.
They differ in exactly two independent choices — whether arms are present, and
which body segment parameter (BSP) set the segments carry:

|               | Male-derived BSP                          | Female-adjusted BSP                         |
|---------------|-------------------------------------------|---------------------------------------------|
| **Armless**   | `average_female_armless_male-bsp.osim`     | `average_female_armless_female-bsp.osim`     |
| **With arms** | `average_female_with-arms_male-bsp.osim`   | `average_female_with-arms_female-bsp.osim`   |

As shipped (unscaled):

| variant               | bodies | coordinates | muscles | markers | total mass |
|-----------------------|-------:|------------:|--------:|--------:|-----------:|
| armless, male-bsp     |     14 |          25 |      80 |      40 | 59.6020 kg |
| armless, female-bsp   |     14 |          25 |      80 |      40 | 59.6020 kg |
| with-arms, male-bsp   |     22 |          35 |      80 |      66 | 59.6020 kg |
| with-arms, female-bsp |     22 |          35 |      80 |      66 | 59.6020 kg |

Total body mass is identical across all four variants; it is conserved by
construction in both the arm-grafting and the BSP-adjustment steps. All four
carry 44 wrap objects and load with zero missing-geometry warnings from this
folder.

The model's skeleton corresponds to a woman of about **166 cm** and **59.6 kg**
(the source cohort mean). Do not take stature from the display meshes: the
inherited head mesh has no cervical spine and stops roughly 7 cm below a real
head vertex, so a mesh-derived head-to-sole height reads about 1528 mm and is
not the model's anthropometry.

`SHA256SUMS.txt` lists checksums for the four `.osim` files.

---

## What "male-bsp" and "female-bsp" mean

**Geometry is average-female in all four files. There is no male skeletal
geometry in this release.**

"BSP" means the inertial description of each segment: its mass, the position of
its centre of mass, and its inertia tensor. These do not affect inverse
kinematics, but they enter inverse dynamics, static optimisation and joint
reaction analysis directly.

- **`male-bsp`** — the segments carry the body segment parameters inherited
  unchanged from the generic Rajagopal template, which were themselves derived
  from male data. The geometry is still average-female; only the inertial
  properties are of male origin.
- **`female-bsp`** — mass, centre-of-mass height and inertia diagonal have been
  adjusted using sex-specific ratios from de Leva (1996), applied
  *multiplicatively* to this model's own values. de Leva's absolute values are
  **not** substituted in; only the female/male ratio is transferred, so the
  adjustment isolates the sex effect and leaves the model's own BSP lineage in
  place.

The adjustment factors, relative to the male-bsp file (mass, centre-of-mass
y-component, and the shape-only inertia ratios about each axis):

| body   | mass   | CoM y  | Ixx    | Iyy    | Izz    |
|--------|-------:|-------:|-------:|-------:|-------:|
| pelvis | 1.1164 | 1.0057 | 0.7746 | 0.8940 | 0.8317 |
| femur  | 1.0438 | 0.8821 | 1.2579 | 1.1821 | 1.2241 |
| tibia  | 1.1109 | 0.9902 | 1.1316 | 0.8135 | 1.1430 |
| foot   | 0.9416 | 0.9092 | 1.3536 | 1.2566 | 1.2968 |
| torso  | 0.9318 | 0.9738 | 0.9170 | 0.8839 | 0.9324 |

Patella takes the tibia factors; talus, calcaneus and toes take the foot
factors. Centre of mass is adjusted on the long axis (y) only, because that is
the only axis de Leva constrains. Segment masses are renormalised so that total
body mass is preserved, with the residual absorbed by the torso, so the torso's
realised mass factor differs slightly from the nominal value above. In the
with-arms variants the arm segments carry their own de Leva-derived factors and
the torso is built without arms.

Geometry, markers, muscle paths and wrap objects are **identical** between the
two BSP variants.

---

## Which variant should I use?

### male-bsp vs female-bsp

**`female-bsp` is the recommended model.** It is the variant whose inertial
properties match the model's name, and there is no accuracy cost at the outputs
the model is usually built for: peak stance hip and knee joint reaction force
change by +0.05 % and −0.06 % between the two variants, neither significant.

**`male-bsp` is shipped as the as-validated archive variant.** The
cross-validation reported in the preprint was run entirely on male-derived
inertia, so this is the inertia lineage that the accuracy claims refer to. Use
it if you want results directly comparable with those reported. One precision
when citing: the cross-validation was run on per-fold models, each built from a
subset of the cohort and each carrying male BSP. The file shipped here is the
full-cohort sibling of those, built by the same pipeline with the same BSP
lineage; it is not itself one of the validated files.

Neither variant is deprecated.

> **Trunk and lumbar loading is the one place this choice is not small.**
> Back joint reaction force differs by 8.3 % between the two BSP variants, and
> lumbar extension moment is the single most BSP-sensitive output in the model —
> its uncertainty exceeds the uncertainty of the BSP input driving it. There is
> no measured female trunk variability behind either variant's trunk
> construction. **Neither variant is validated for spine or trunk work.** If you
> use either model for lumbar loading, report which variant you used and treat
> this as the weak axis of both.

### armless vs with-arms

**Use the with-arms variant only if you place markers on participants' arms.**

OpenSim's ScaleTool derives each body's scale factor from marker measurements.
With no arm markers, the arm bodies receive no measurement and are left at a
scale factor of 1.0 — they keep this model's absolute arm dimensions while the
rest of the body scales to your participant, and the model's own arm markers are
dropped from the scaled model at the same time. This happens silently: no error,
no warning.

If your marker set stops at the trunk, use the **armless** variant. That is the
variant the validation study used.

**Arm construction.** The arm segments are grafted from the generic Rajagopal
male model and scaled per segment, using stature-normalised de Leva (1996)
segment length ratios applied to this model's own size — the same ratio-transfer
approach used for the BSP adjustment:

| segment   | factor   |
|-----------|---------:|
| upper arm | 0.953753 |
| forearm   | 0.959928 |
| hand      | 0.883730 |

The three factors differ because the female hand is about 9 % shorter relative
to stature than the male hand, against 1–2 % for the upper arm and forearm; a
single scale factor cannot express that. The factors are applied to segment
lengths, mesh scale factors, arm markers and mass/CoM/inertia (mass scaling with
the cube of the length factor). Torso mass is reduced by exactly the grafted arm
mass, so total body mass is unchanged.

**Arm segment shape is still male.** No female arm imaging exists for this
cohort, so only the proportions were changed; the arm bone meshes are the stock
Rajagopal geometry. State this if you report arm kinematics.

---

## Files and geometry

```
average_female_armless_male-bsp.osim
average_female_armless_female-bsp.osim
average_female_with-arms_male-bsp.osim
average_female_with-arms_female-bsp.osim
FemaleGeometry/      81 mesh files, used by all four models
SHA256SUMS.txt
```

`FemaleGeometry/` must stay alongside the `.osim` files, under exactly that
name. Every mesh in all four models is referenced as `FemaleGeometry\<name>`.

The folder holds two kinds of mesh:

| files               | what they are | how they are scaled |
|---------------------|---------------|---------------------|
| 21 named `female_*` | Female-specific: 11 cohort-averaged, symmetrised lower-limb bones (`.stl` — femur, tibia, fibula, patella, pelvis L/R, sacrum) and 10 torso, head and foot meshes (`.vtp`) | baked into the vertex coordinates; the models carry `scale_factors = 1 1 1` |
| 60 with stock names | Stock Rajagopal arm and hand geometry, byte-identical to OpenSim's own files (humerus, radius, ulna, carpals, metacarpals, phalanges). Used by the with-arms models only | applied at load time via XML `scale_factors` |

The `female_` prefix is deliberate. Ten of the female meshes would otherwise
carry the same filenames as their male counterparts in a standard OpenSim
`Geometry/` folder (`hat_skull.vtp`, `r_foot.vtp`, `r_talus.vtp`, and so on).
Mixing such a folder with a stock OpenSim installation would silently shadow one
set with the other — no error, and no visible symptom beyond subtly wrong bones.
The 60 stock meshes keep their original names, because they *are* the stock
meshes.

---

## Known limitations

- **Trunk and lumbar loading** — see the note above. Neither BSP variant is
  validated for spine work.
- **Arm segment shape is male** in the with-arms variants; only the proportions
  are female-adjusted.
- **Left-femur length bias** — in the underlying cohort the left femur is
  systematically slightly shorter than the right. The released models are
  left/right symmetrised, but this bias is present in the averaged geometry they
  were built from. Discussed in the preprint.
- **Products of inertia** are zero in all bodies, and were carried through
  unchanged by the female-BSP adjustment.

---

## Licence

[TO BE CONFIRMED]

## Citation

If you use these models, please cite:

[PREPRINT, in preparation]

The models are derived from the Rajagopal-lineage generic model; please also
cite Rajagopal et al. (2016), Lai et al. (2017) and Uhlrich et al. (2022) as
appropriate, and de Leva (1996) if you use a female-BSP or a with-arms variant.

## References

- de Leva P (1996). Adjustments to Zatsiorsky-Seluyanov's segment inertia
  parameters. *Journal of Biomechanics* 29(9):1223–1230.
- Rajagopal A, Dembia CL, DeMers MS, Delp DD, Hicks JL, Delp SL (2016).
  Full-body musculoskeletal model for muscle-driven simulation of human gait.
  *IEEE Transactions on Biomedical Engineering* 63(10):2068–2079.
- Lai AKM, Arnold AS, Wakeling JM (2017). Why are antagonist muscles
  co-activated in my simulation? *Annals of Biomedical Engineering*
  45(12):2762–2774.
- Uhlrich SD, Jackson RW, Seth A, Kolesar JA, Delp SL (2022). Muscle
  coordination retraining inspired by musculoskeletal simulations reduces knee
  contact force. *Scientific Reports* 12:9842.
