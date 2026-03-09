# colab Command Reference

Source: https://github.com/xeodou/colab-cli
Binary name: `colab`

## Commands

### auth
Authenticate with Google account via OAuth2.
```bash
colab auth
```
Opens browser for consent. Token cached at `~/.config/colab/token.json`.

### exec
Execute code on a Colab runtime.
```bash
# Execute a notebook (all cells)
colab exec notebook.ipynb

# Execute a Python file
colab exec script.py

# Execute inline code
colab exec -c "import torch; print(torch.cuda.is_available())"
```
Output is streamed in real-time via WebSocket connection to the Jupyter kernel.

### upload
Upload a local file to the Colab runtime filesystem.
```bash
colab upload local_file.ipynb
colab upload data.csv
```

### download
Download a file from the Colab runtime to local.
```bash
colab download /content/results.csv ./results.csv
colab download /content/drive/MyDrive/output.png ./output.png
```

### quota
Display remaining Colab compute quota.
```bash
colab quota
```
Shows GPU hours remaining, current runtime type.

### status
Show current runtime information.
```bash
colab status
```
Shows: connection state, RAM usage, disk usage, GPU type (if any).

### stop
Release the current Colab runtime.
```bash
colab stop
```
Frees GPU/TPU resources. Data in `/content/` is lost (use Drive for persistence).

## Requirements

- Go 1.21+ (for installation)
- Google account with Colab access
- Network access to Google services

## Limitations

- OAuth2 test app: max 100 users, "unverified app" warning on first auth
- Colab free tier: ~12 hour session limit, 90 min idle timeout
- GPU availability depends on Colab quota and demand
