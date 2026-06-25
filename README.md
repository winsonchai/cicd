# wf_backend (CI/CD Test Application)

This is a FastAPI-based backend application configured with a GitHub Actions CI/CD pipeline for automated testing and deployment to the on-prem server `tpt-mq-ssbe03.wdc.com`.

## Local Development

### 1. Prerequisites
- Python 3.10+
- Virtual environment tool

### 2. Setup
Create a virtual environment and install the dependencies:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: .\venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Running the App
Run the FastAPI application locally with Uvicorn:
```bash
uvicorn main:app --reload
```
The app will be available at `http://127.0.0.1:8000`.

### 4. Running Tests
Run the test suite using pytest:
```bash
pytest
```

---

## CI/CD Deployment Setup (On-Prem Server)

The CI/CD pipeline in `.github/workflows/cicd.yml` uses a **GitHub Self-Hosted Runner** to deploy directly to the on-prem server `tpt-mq-ssbe03.wdc.com`. Follow the steps below to prepare the server for deployments.

### 1. Install & Configure GitHub Self-Hosted Runner
On your `tpt-mq-ssbe03.wdc.com` server:
1. Navigate to your GitHub Repository -> **Settings** -> **Actions** -> **Runners**.
2. Click **New self-hosted runner** and select **Linux** (or your OS).
3. Follow the download and configuration instructions.
4. Add the labels `self-hosted` and `linux` during configuration.
5. Install the runner as a background service:
   ```bash
   sudo ./svc.sh install
   sudo ./svc.sh start
   ```

### 2. Configure Systemd Service
Create a systemd unit file on the server at `/etc/systemd/system/wf_backend.service`:

```ini
[Unit]
Description=FastAPI wf_backend Application
After=network.target

[Service]
User=runner  # Change to the user running the self-hosted runner
WorkingDirectory=/opt/wf_backend
ExecStart=/opt/wf_backend/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable wf_backend
sudo systemctl start wf_backend
```

### 3. Configure Sudo Privileges for the Runner User
To allow the runner to automatically restart the systemd service during deployment without prompting for a password, add the following line to `/etc/sudoers` or `/etc/sudoers.d/runner`:

```text
runner ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart wf_backend.service, /usr/bin/mkdir -p /opt/wf_backend, /usr/bin/chown -R runner\:runner /opt/wf_backend
```
*(Make sure to replace `runner` with the actual username running the GitHub self-hosted runner service).*
