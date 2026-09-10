# Permeability → occupancy explorer

Internal interactive tool (Lemna Bio). Given a cyclic peptide's permeability and
affinity, it shows whether the compound reaches useful intracellular target
occupancy inside a plate-based cellular assay — and what is actually limiting it.

**Status: internal-only.** This build contains real measured PAMPA values in the
`Our PAMPA set` preset. Do not host it on a world-readable URL as-is. See
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
serum-free, no efflux, trace target unless stated):

| Case | Expected |
|---|---|
| logPe −7, KD 50 nM, 60% goal | 156 s, matches `−tau·ln(1 − 0.075)` |
| logPe −7, run to completion | settles at 95.2% |
| logPe −7, free at t = tau | 0.632 µM (1 − 1/e) |
| ER = 8, KD 50 nM, 1 µM | ceiling 71.4% |
| fu 0.08 + ER 8 | ceiling 16.7%, goal 60% unreachable |
| [T] 0.5 / 2 / 10 / 40 µM, logPe −7 | 60% at 13 min / 44 min / 3.5 h / 14 h, all settle at 95.2% |

Do not "improve" the physics without checking this table.

## Hosting

Not wired yet (deliberate).

**GitHub Pages will not keep this private.** On Free/Pro/Team plans a Pages site
is world-readable by anyone with the URL even when the repo is private; access-
controlled Pages needs GitHub Enterprise Cloud. Since this build carries real
PAMPA data, do not deploy it to Pages until the preset is genericised.

For an internal build with real data, host behind an auth gate — e.g. Cloudflare
Pages + Cloudflare Access (free for a small team, deploy-on-push). Decision pending.

## Roadmap (from BRIEF.md)

1. ~~Repo scaffold~~ / static hosting with deploy on push — *hosting pending*
2. ~~URL-encoded shareable state~~ — done
3. Mobile layout — charts not yet tuned for narrow screens
4. Named presets per real target — pending public/internal + PAMPA decision
