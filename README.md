# Permeability → occupancy explorer

Internal interactive tool (Lemna Bio). Given a cyclic peptide's permeability and
affinity, it shows whether the compound reaches useful intracellular target
occupancy inside a plate-based cellular assay — and what is actually limiting it.

**Status: internal-only.** A Lemna team tool, not a public asset. It needs real
access control at the host (SSO or host-level password), not an unlisted URL. See
[Hosting](#hosting).

## Running it

It is one self-contained file. Open `index.html` in a browser — no build step,
no server, no npm. The only external request is Google Fonts (Poppins).

## Sharing state via link

Every control is encoded into the URL hash (`index.html#logPe=-7&kd=50&...`).
- **Copy link** button (top right) puts the current parameter set on the clipboard.
- Opening a link applies those parameters on load; back/forward and pasting a new
  link re-hydrate the controls.
- State lives in the hash, never the query string, so it is never sent to a server.
- Values are clamped to each control's range on read, so a stale or hand-edited
  link can never push a slider out of bounds. Sliders remain independent — a link
  sets positions, nothing moves another control.

## The model

One well-mixed intracellular compartment behind one membrane; passive flux
proportional to the free-concentration difference. Concentrations in µM, time in
seconds, Pe in cm/s, lengths in cm. See the "The model, and what it assumes" panel
in the page and `BRIEF.md` for the full specification.

### Regression contract — do not break

Any change to `simulate()` must still pass these (V/A = 2 µm, dose 1 µM,
serum-free, no efflux, trace target unless stated). All currently pass exactly:

| Case | Expected |
|---|---|
| logPe −7, KD 50 nM, 60% goal | 156 s, matches `−tau·ln(1 − 0.075)` |
| logPe −7, run to completion | settles at 95.2% |
| logPe −7, free at t = tau | 0.632 µM (1 − 1/e) |
| ER = 8, KD 50 nM, 1 µM | ceiling 71.4% |
| fu 0.08 + ER 8 | ceiling 16.7%, goal 60% unreachable |
| [T] 0.5 / 2 / 10 / 40 µM, logPe −7 | 60% at 13 min / 44 min / 3.5 h / 14 h, all settle at 95.2% |

**Integrator.** `simulate()` steps the linearised ODE `dL/dt = a − b·L` with the
exact `L(t+dt) = L∞ + (L−L∞)·e^(−b·dt)` (b uses the analytic slope of free wrt
total), and finds the goal crossing analytically within the step. It is stable at
any step size — a plain Euler step diverges above ~logPe −6. Do **not** swap it for
Euler/RK, and re-run this table if you touch it.

**Effective KD.** A competing partner shifts the apparent KD by
`KD·(1 + [partner]/KD_partner)`; with no partner it is unchanged, so every case
above and all prior behaviour are reproduced exactly.

Do not "improve" the physics without checking this table.

## Hosting

Not wired yet (deliberate).

**GitHub Pages will not keep this private.** On Free/Pro/Team plans a Pages site
is world-readable by anyone with the URL even when the repo is private; access-
controlled Pages needs GitHub Enterprise Cloud. This is internal-only, so do not
use Pages.

Host behind an auth gate — e.g. Cloudflare Pages + Cloudflare Access (free for a
small team, deploy-on-push, custom subdomain under lemna.bio). Domain is an open
decision for Moustafa.

## State of the spec

Done: exponential integrator, competing-partner term, "Goal for first hit"
defaults, five-control hierarchy with the rest in Advanced, goal-by-the-verdict
with a live ceiling readout, near-ceiling warning, "assumes no efflux" flag,
info-icon popovers (21 wired), mechanism-time marker, competing-partner inputs,
labelled presets (archetypes styled distinctly), URL-encoded shareable state.

Pending: access-controlled hosting + domain. Mobile is explicitly out of scope
(desktop only).
