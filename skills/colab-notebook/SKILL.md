---
name: colab-notebook
description: Execute and manage Jupyter notebooks on Google Colab from the terminal. Use when user asks to "run notebook", "execute on Colab", "push to Colab", "pull results", "check Colab status", or works with .ipynb files that need GPU/cloud execution.
license: MIT
metadata:
  author: bolkhovsky
  version: 1.0.0
---

# Google Colab Notebook Management

Manage Jupyter notebooks on Google Colab directly from Claude Code using `colab` CLI (xeodou/colab-cli).

## Prerequisites

Verify `colab` is installed:

```bash
colab --version
```

If not installed, install via Go:

```bash
go install github.com/xeodou/colab@latest
```

Or via curl:

```bash
curl -fsSL https://raw.githubusercontent.com/xeodou/colab/main/install.sh | bash
```

Then authenticate (one-time — opens browser for Google OAuth2):

```bash
colab auth
```

## Core Workflow

### Step 1: Edit notebook locally

Edit `.ipynb` files using standard tools. When editing notebooks programmatically, use Python's `json` module:

```bash
python3 -c "
import json
with open('notebook.ipynb') as f:
    nb = json.load(f)
# Modify cells...
with open('notebook.ipynb', 'w') as f:
    json.dump(nb, f, indent=1, ensure_ascii=False)
"
```

Common cell operations:
- **Change source**: `nb['cells'][N]['source'] = [line1, line2, ...]`
- **Clear outputs**: `nb['cells'][N]['outputs'] = []`
- **Add cell**: append dict with `cell_type`, `source`, `metadata`, `outputs`
- **Check for strings**: `''.join(cell['source'])` to search cell content

### Step 2: Upload and execute

```bash
# Upload notebook to Colab
colab upload notebook.ipynb

# Execute the notebook (runs all cells)
colab exec notebook.ipynb

# Execute specific code inline
colab exec -c "print('hello from Colab')"

# Execute a Python script
colab exec script.py
```

Output streams in real-time via WebSocket.

### Step 3: Check status and resources

```bash
# Check runtime status (connected, RAM, disk)
colab status

# Check GPU quota remaining
colab quota
```

### Step 4: Download results

```bash
# Download files from Colab runtime
colab download /content/output.csv ./local_output.csv

# Download from Google Drive (if notebook saves there)
colab download /content/drive/MyDrive/experiment/results.csv ./results.csv
```

### Step 5: Stop runtime

```bash
# Release the Colab runtime when done
colab stop
```

## Notebook Patterns

### Google Drive persistence

When notebooks need data that survives session disconnects, mount Google Drive:

```python
from google.colab import drive
drive.mount('/content/drive')

from pathlib import Path
DRIVE_ROOT = Path('/content/drive/MyDrive/my_experiment')
DATA_DIR   = DRIVE_ROOT / 'data'
OUT_DIR    = DRIVE_ROOT / 'outputs'
MODEL_DIR  = DRIVE_ROOT / 'models'

for d in [DATA_DIR, OUT_DIR, MODEL_DIR]:
    d.mkdir(parents=True, exist_ok=True)
```

Cache expensive computations on Drive:

```python
CACHE = DRIVE_ROOT / 'processed_data.npy'
if CACHE.exists():
    data = np.load(str(CACHE))  # seconds
else:
    data = expensive_computation()  # minutes
    np.save(str(CACHE), data)
```

### Fixing failed cells

When a cell fails:
1. Read the notebook to find the error: check `outputs` of the failed cell
2. Fix the source code in the cell
3. Clear stale outputs on dependent cells
4. Re-upload and re-execute

```bash
python3 -c "
import json
with open('notebook.ipynb') as f:
    nb = json.load(f)
# Find failed cell by checking outputs for 'error' output_type
for i, cell in enumerate(nb['cells']):
    for out in cell.get('outputs', []):
        if out.get('output_type') == 'error':
            print(f'Cell {i}: {out[\"ename\"]}: {out[\"evalue\"]}')
"
```

### Year/date range issues in data downloads

When downloading time-series data, verify the actual date range loaded matches expectations:

```python
# After loading dates, always verify:
print(f"Date range: {dates[0]} to {dates[-1]}")
print(f"Years present: {sorted(set(d.year for d in dates))}")
```

If expected years are missing, the data source path or filename pattern is likely wrong for that time period.

## Troubleshooting

### "colab: command not found"

Ensure Go bin is in PATH:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```

### Authentication expired

Re-run `colab auth` to refresh the OAuth2 token.

### "Runtime not connected"

The Colab runtime may have disconnected due to idle timeout (90 min) or session limit (12 hours). Run `colab exec` to reconnect, or use Google Drive persistence so data survives reconnection.

### Large file transfers are slow

For files over 100 MB, prefer saving to Google Drive from within the notebook rather than downloading via `colab download`. Access Drive files locally via Google Drive desktop sync or `rclone`.
