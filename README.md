# DVC + GCP Lab (My Working Setup)

This repository is my hands-on Data Version Control lab where I track local data with DVC and sync versions to a Google Cloud Storage bucket.

## Project Goal

- Track dataset changes without storing raw data in Git
- Store versioned data artifacts in GCS using DVC
- Reproduce the latest tracked data with `dvc pull` / `dvc checkout`

## Current Configuration

- DVC remote name: `myremote`
- DVC remote URL: `gs://dvc-lab-pranathi`
- Tracked file: `data/data.txt`
- Metadata file: `data/data.txt.dvc`

## Repository Layout

- `data/data.txt` - working data file (ignored from normal Git data storage flow)
- `data/data.txt.dvc` - DVC pointer file committed to Git
- `.dvc/config` - shared DVC config (safe to commit)
- `.dvc/config.local` - local credential settings (must not be committed)
- `config/dvc-lab.json` - GCP service account key (private, ignored by Git)

## One-Time Setup

1. Install DVC with GCS support:
   ```bash
   pip install "dvc[gs]"
   ```
2. Initialize DVC if needed:
   ```bash
   dvc init --no-scm
   ```
3. Set remote storage:
   ```bash
   dvc remote add -d myremote gs://dvc-lab-pranathi
   ```
4. Configure credentials locally (not in tracked config):
   ```bash
   dvc remote modify --local myremote credentialpath config/dvc-lab.json
   ```

## Daily Workflow

### Track new/changed data

```bash
dvc add data/data.txt
git add data/data.txt.dvc .gitignore
```

### Push data to cloud

```bash
dvc push
```

### Pull/reproduce tracked data

```bash
dvc pull
dvc checkout
```

## Verification Commands

Use these commands to confirm the setup is healthy:

```bash
dvc remote list
dvc status
dvc status -c
```

Expected healthy output for cloud sync:

```text
Cache and remote 'myremote' are in sync.
```

## Security Notes

- Never commit service account keys
- Keep credentials only in `.dvc/config.local`
- Ensure these are ignored in `.gitignore`:
  - `config/dvc-lab.json`
  - `.dvc/config.local`
  - `.DS_Store`

## Troubleshooting

- **401 Invalid Credentials**: verify key file path and active service account permissions.
- **404 Bucket not found**: confirm bucket name in `.dvc/config` matches actual GCS bucket.
- **`dvc pull` blocked by local changes**: either commit/retrack local changes with `dvc add` or intentionally force pull if you want to discard local edits.