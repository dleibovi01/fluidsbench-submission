# DrivAerNet++ dataset-owner approval (interim record)

The dataset owner's scientific approval is recorded here while the scoring support is still
`owner_review_required`. It **cannot** yet be written into `submission-spec.json` as the
`owner_approval` field: `scripts/validate_scoring_supports.py` lists `owner_approval` and
`publication_validation` among the fields a non-official specification must not publish. The
values below move into the specification in the same change that sets
`scoring_support.status` to `official`.

## Approval

| Field | Value |
|---|---|
| `approved_by` | Mohamed Elrefaie (DrivAerNet++ dataset author) |
| `approved_at` | 2026-09-04 |
| `pull_request_url` | https://github.com/neilashton/fluidsbench-submission/pull/2 |

## What is approved

The scientific definition of the pressure-only v1 benchmark as it stands on this branch:

- the pinned public source (Harvard Dataverse `doi:10.7910/DVN/K7PWNJ` version 1.0,
  `SurfacePressureVTK/DrivAer_<ID>.vtk`, PointData `p`), and the splits pinned at
  `github.com/Mohamedelrefaie/DrivAerNet` commit `8b8c053`;
- the pressure convention: kinematic **gauge** pressure in m²/s² with the freestream
  reference at zero, so `Cp = p / 450` is exact at U∞ = 30 m/s;
- the official evaluation split: the 1,154-case test list, ordered lexicographically;
- the scoring support: every native surface point exactly once, with mass-lumped dual-area
  weights, and the area-weighted relative L2 as the primary ranking metric;
- `overall_score = 100 − surface_pressure_rel_l2` for v1;
- retaining all four pressure-profile stations, subject to the extraction rules in
  `profiles/STATION_DEFINITIONS.md` being validated on real geometries.

## What is not yet approved

Approval of the definition does **not** by itself open submissions. Status as of
2026-10-18:

1. **Scoring-support release: candidate produced, approval outstanding.** The
   full-split publisher run produced `drivaernetplusplus-native-pressure-support-v1-candidate`
   (manifest, case index, chunks, 1,154 per-case records and 24 archives). See
   `REAL_DATA_VALIDATION.md`. Still needed: hosting of the archives, owner approval,
   and the `publication_validation` receipt at release.
2. **Profiles: extraction validated, owner confirmation outstanding.** Three stations
   are frozen and extract on all 1,154 designs (`profiles/STATION_DEFINITIONS.md`).
   `front_wheelhouse` is proposed for retirement, which departs from "retaining all
   four stations" above and needs the owner's confirmation. Profiles are report-only.
   Public profile ground truth for the website repository is not yet exported.
3. **Prototype split and placeholder rows: retired.** `splits/default.json` and the
   twelve prototype leaderboard submissions were removed.
4. **The 29 unassigned designs: reason outstanding.** The release has 8,150 pressure
   files; 8,121 appear in the splits (5,819 train, 1,148 validation, 1,154 test) and
   in `DrivAerNetPlusPlus_Cd_8k_Updated.csv`. The other 29 are in neither:

   `E_S_WWC_WM_338`, `E_S_WWC_WM_619`, `E_S_WWC_WM_626`, `E_S_WWC_WM_630`,
   `F_D_WM_WW_0132`, `F_D_WM_WW_0603`, `F_D_WM_WW_0797`, `F_D_WM_WW_1118`,
   `F_D_WM_WW_1421`, `F_D_WM_WW_1556`, `F_D_WM_WW_1891`, `F_D_WM_WW_2353`,
   `F_D_WM_WW_2459`, `F_D_WM_WW_3199`, `F_S_WWC_WM_264`, `F_S_WWS_WM_017`,
   `F_S_WWS_WM_032`, `F_S_WWS_WM_041`, `F_S_WWS_WM_052`, `F_S_WWS_WM_061`,
   `F_S_WWS_WM_491`, `F_S_WWS_WM_516`, `F_S_WWS_WM_518`, `F_S_WWS_WM_534`,
   `F_S_WWS_WM_650`, `N_S_WWC_WM_496`, `N_S_WWC_WM_497`, `N_S_WWC_WM_518`,
   `N_S_WWS_WM_176`.

   Their pressure files look like the rest: 5 connected components, no non-finite
   values, stagnation `p` of about 450 m²/s². The reason for their exclusion is not
   visible in the pressure data, and the owner will record it here. The official test
   list is used unchanged either way.
5. **Localized pressure extremes: decision outstanding.** 768 points in 79 test designs
   lie outside the physical pressure range (see `README.md`). The approved definition
   scores every point as stored, and the candidate support does so. The owner should
   confirm this, or define a masking rule before the support becomes official.
