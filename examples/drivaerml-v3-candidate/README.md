# DrivAerML FluidsBench schema-v3 candidate package

The `dev` branch provides the complete native DrivAerML candidate workflow.
Copy `package-config.template.json` and replace the participant `__REPLACE_...__`
fields. Its benchmark-owned release bindings are already filled in and must
match the checked-out dev specification.

Start with the [participant guide](../../benchmark-specs/drivaerml/PARTICIPANT_GUIDE.md).
It covers native prediction chunks, fixed and relative profile extraction,
complete-split scoring, packaging, and the pull request to `dev`. Official
submissions remain closed; registered dev results are non-rankable and non-citable.

Use the shared [hardware and compute fields](../../METHODOLOGY.md#hardware-and-compute)
for each training stage and inference campaign, including GPU model and allocation counts.

## Participant configuration

Copy `package-config.template.json` outside the repository, fill the
participant fields. Keep the supplied benchmark release values. Check what
participant information remains before attempting assembly:

```bash
python scripts/assemble_drivaerml_schema_v3_candidate.py \
  --config /path/to/package-config.json \
  --list-unresolved
```

Include the common [methodology record](../../METHODOLOGY.md) and the selected scope's outputs.
`surface_only` requires the two surface fields and eight fixed/relative Cp series; omit volume predictions and velocity series, retain fixed
zero contributions without renormalizing weights, and do not fill missing metrics with dummy values (maximum overall score 60).
`surface_and_volume` requires all four fields and 40 fixed/relative profile series.
Relative diagnostics carry zero score weight. Shared Cp aliases are generated automatically.

<details>
<summary>Required method configuration, checkpoint identity, stages, and parameter counts</summary>

The config-v2 format for schema-v3 packages requires a structured
`participant.methodology` record. Start from the placeholders in the template
and use [`methodology.example.json`](methodology.example.json) only as a shape
example. Replace all illustrative values with the submitted method's actual:

- named architecture components, exact total and submitter-trainable parameter
  counts, scoped hyperparameters and inputs, and the production path for all
  required fields for the selected scope (two surface fields in either scope, plus two volume fields only for `surface_and_volume`);
- normalization, preprocessing, and sampling plus every submitter or upstream
  training stage; submitter stages include their fitting procedure, runs,
  random seeds when stochastic, and measured compute;
- a raw-file SHA-256, byte description, component scope, role, and pre-test
  selection rule for every loaded checkpoint file; and
- complete-split inference campaign wall time, aggregate device time, and their
  inclusion boundaries.

`parameter_count_millions` is deliberately absent from the configuration. The
assembler derives it from `methodology.architecture.total_parameter_count`. The
checkpoint hashes bind the local bytes actually loaded for inference; they do
not require public model-weight upload and need not equal the hash of an
optional published model archive.

The template shows one jointly trained component and one gradient-based stage.
Add components, stages, and checkpoint-file entries when surface and volume use
different networks, when training is staged, or when a model is an ensemble.
For an upstream stage use `status=performed_upstream`, a component scope, and
an upstream reference. This provenance choice is independent of the top-level
target-data `training_regime`.

</details>

## Commands: assemble

The token report does not validate repository bindings or evaluator outputs;
successful assembly verifies the package bindings and schemas. After the
case evaluator and split reducer produce `metrics/cases.json`,
`regional-diagnostics.json`, and `profiles/`, assemble the
participant-owned files with:

```bash
python scripts/assemble_drivaerml_schema_v3_candidate.py \
  --config /path/to/package-config.json \
  --case-metrics /path/to/metrics/cases.json \
  --profiles /path/to/profiles \
  --regional-diagnostics /path/to/regional-diagnostics.json \
  --checkpoint <checkpoint-id>=/path/to/checkpoint.pt \
  --discretization-cases /path/to/discretization/cases.jsonl \
  --output /path/to/drivaerml-my-model-v1
```

## Outputs and validation

The assembler derives dataset and split identities from the checked-in
DrivAerML specification, canonicalizes the participant files, creates the
hash-bound `submission.json`, `evaluation-evidence.json`, and
`discretization.json`, and schema-checks the result. It refuses unresolved
tokens, incomplete or reordered cases, missing metrics or profiles, mismatched
release bindings, and an existing output directory.

Evaluator-produced `metrics/cases.json` and `profiles/index.json` must already
carry the exact submission, split, and case-set identities supplied to the
evaluator. `metrics/cases.json` must additionally carry the exact candidate
support release ID and manifest hash; the profile index has no release-identity
fields. The assembler rejects conflicts instead of rewriting them and verifies
every source profile chunk against the evaluator's input index before
canonicalizing it. Only the participant-authored per-case
`discretization/cases.jsonl` may omit its repeated schema/submission/dataset/
split identity fields; the assembler inserts those fields and rejects any
conflicting value that is present.

The frozen evaluator Git revision is benchmark-owned and is pinned by
`scoring_support.dataset_evaluator_binding` in the benchmark specification; it
is never written to `evaluation.code_revision`. That optional submission field
identifies the participant's public model code. The assembler omits it when
`participant.reproducibility.code` is absent; when that block is supplied, it
copies the participant code commit into both submission and evidence records.

The output contains no `approval`, `maintainer-validation.json`, or
`prediction-artifact-checks.json`; those are not contributor-owned records.
Complete native prediction artifacts are optional and are intentionally not
added by this initial assembler.

Repeat `--checkpoint ID=PATH` for every checkpoint declared in the method
record. The assembler checks the actual local file bytes against each SHA-256.
Keep checkpoint files outside the submission package.

Validate the new directory inside the repository:

```bash
python scripts/validate_submission.py --dev-intake --contributor-stage \
  submissions/drivaerml/<submission-id>
```

Open a PR targeting `dev` with only that new submission directory. After review,
a maintainer runs:

```bash
python scripts/register_drivaerml_pre_release.py \
  submissions/drivaerml/<submission-id>/submission.json --register
python scripts/manage_leaderboard.py build --dev-intake
python scripts/manage_leaderboard.py check --dev-intake
```

Registration binds the exact package tree and permits dev display. It does not
create official approval. Changed package bytes invalidate the registration.
