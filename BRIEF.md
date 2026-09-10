# Permeability to occupancy explorer — technical brief

## What this is

A single-page interactive tool for drug discovery scientists. It answers one question:
given a cyclic peptide's permeability and affinity, does it reach useful target occupancy
inside a plate-based cellular assay, and what is actually stopping it.

Built by Lemna Bio. Audience is discovery biologists and computational chemists, internal
first, possibly public later.

## Current state

`permeability-occupancy-explorer.html`. One self-contained file. Vanilla JS, hand-rolled
SVG charts, no build step, no npm, no framework. Only external request is Google Fonts
(Poppins). It works as-is; treat it as the spec.

## The model

One well-mixed intracellular compartment behind one membrane. Passive flux proportional to
the free concentration difference.

```
k     = Pe / (V/A)_cell                          influx rate constant, 1/s
tau   = 1 / (k · ER)
dL/dt = k · (C_med,free − ER · L_free) − k_deg · L
L     = L_free/fu_cell + [T] · L_free/(L_free + KD)
occ   = L_free / (L_free + KD)
```

`L` is total drug inside, `L_free` is what the target sees. `L_free` is recovered from `L`
by bisection (monotonic). Medium is a fixed reservoir decaying only by the stability
half-life. Integration is forward Euler with geometrically growing dt.

Units internally: concentrations µM, time seconds, Pe cm/s, lengths cm.

### Inputs

| Symbol | Control | Range | Meaning |
|---|---|---|---|
| logPe | Permeability | −9 to −4 | log10 of Pe in cm/s |
| KD | Affinity | 0.05 nM to 10 µM | dissociation constant |
| t½ | Stability in medium | 6 min to stable | first-order decay of compound |
| dose | Dose in medium | 0.01 to 200 µM | total, before serum binding |
| inc | Incubation | 5 min to 48 h | readout time |
| (V/A) | Cell volume/surface | 0.5 to 5 µm | 1 to 2.5 µm is physiological |
| fu | Free fraction in medium | 0.001 to 1 | serum protein binding |
| ER | Efflux ratio | 1 to 200 | fold reduction in steady-state cytosol |
| fu_cell | Free fraction in cytosol | 0.01 to 1 | intracellular non-specific binding |
| [T] | Target concentration | 5 nM to 100 µM | intracellular target abundance |
| goal | Occupancy you want | 5 to 99% | the question being asked |

Plus cells per well and medium volume, used only for the reservoir depletion check.

## Three claims the page exists to argue

Do not break these while refactoring. They are the reason the tool exists.

1. Permeability sets the rate, not the endpoint. Without efflux, steady-state free cytosol
   equals free medium at any logPe. Low permeability only kills a plate experiment when the
   equilibration time exceeds the incubation.
2. Efflux is different in kind from every other loss. It is the only term that lowers the
   steady-state ceiling rather than delaying arrival. It also speeds equilibration, since it
   adds to the total rate of leaving.
3. Target abundance does not change steady-state occupancy. It buffers the approach, so an
   abundant target loads slowly and lands in the same place. It only caps occupancy if the
   well runs out of compound, which the reservoir check tests separately.

## Deliberately not modelled

Endocytosis and endosomal entrapment, lysosomal trapping, active uptake, pH partitioning,
membrane potential opposing a charged species, intracellular metabolism, and binding kinetics
slower than transport.

Endosomal uptake is the significant omission. It produces cell-associated compound that
engages target only after lysis, which is the main way a live-cell PELSA readout misleads.
It is named in the assumptions panel. If this page goes public it should be promoted to a
visible line near the verdict.

## Regression checks

Any change to `simulate()` must still pass these. All at V/A = 2 µm, dose 1 µM, serum-free,
no efflux, trace target unless stated.

| Case | Expected |
|---|---|
| logPe −7, KD 50 nM, 60% goal | 156 s, matches closed form `−tau·ln(1 − 0.075)` |
| logPe −7, run to completion | settles at 95.2% |
| logPe −7, free at t = tau | 0.632 µM (1 − 1/e) |
| ER = 8, KD 50 nM, 1 µM | ceiling 71.4% |
| fu 0.08 + ER 8 | ceiling 16.7%, goal 60% unreachable |
| [T] 0.5 / 2 / 10 / 40 µM, logPe −7 | 60% at 13 min / 44 min / 3.5 h / 14 h, all settling at 95.2% |

## Design tokens

Already in the CSS variables at the top of the file. Navy `#0D143E`, indigo `#5364F1`,
teal `#50CBC3`, pale teal `#D1EEEC`, yellow `#FBC942`, red `#E34539`, off-white `#F9F9F9`.
Poppins throughout, tabular numerals on the readouts so live values do not jitter.

The dark navy cross-section panel is the one bold element. Keep everything around it quiet.

## What to build

Priority order:

1. Repo scaffold and static hosting with deploy on push. Static host, custom domain under
   lemna.bio.
2. URL-encoded state, so a scientist can send a colleague a link with the parameters already
   set. This is the highest-value feature and it is why the tool becomes shareable rather
   than just visitable.
3. Mobile layout. Currently degrades to a single column but the charts are not tuned for it.
4. Named presets per real target, replacing the current placeholder set.

## Constraints

- Keep it dependency-free and single-file if possible. Portability is a feature here.
- No analytics that sets cookies without a banner.
- Do not "improve" the physics without checking the regression table above.
- Sliders are independent by design. Nothing should move another slider's position.

## Open decisions (owner: Moustafa)

- Public or internal-only. Changes the copy, not the code.
- Whether the `Our PAMPA set` preset keeps real measured values or is genericised before
  anything is published.
- Domain.
