Raspberry Pi 5 (8GB) temporary deployment guide, cloud-migration ready

Goal
- Members can use the simulator by opening one URL and pressing Predict.
- No Python setup on member PCs.
- Start on Raspberry Pi, migrate later to Render/Railway/Fly.io/Cloud Run.

Architecture
- Frontend: static files (GitHub Pages in future).
- Backend: FastAPI on Raspberry Pi now, cloud later.
- Simulator: backend/balloon_and_drift.py

0) What you need before SSH
- Raspberry Pi OS (64-bit) recommended.
- SSH access to Pi.
- Copernicus Marine account (username/password).

1) SSH to Raspberry Pi
- From your PC terminal:
  ssh pi@<RASPI_IP>

2) Install system dependencies on Pi
- Run on Pi:
  sudo apt update
  sudo apt install -y git python3-venv python3-pip build-essential pkg-config \
    libgeos-dev libproj-dev proj-data proj-bin libgdal-dev gdal-bin \
    libnetcdf-dev libhdf5-dev libopenblas-dev liblapack-dev

3) Clone project and create venv
- Run on Pi:
  cd ~
  git clone <YOUR_REPO_URL> leaflet_predictor
  cd leaflet_predictor
  python3 -m venv .venv
  source .venv/bin/activate

4) Install Python dependencies
- Run on Pi:
  pip install --upgrade pip setuptools wheel
  pip install -r backend/requirements.txt

5) Copernicus login (server side)
- Interactive login:
  copernicusmarine login
- If interactive login is difficult:
  export COPERNICUSMARINE_SERVICE_USERNAME="your_username"
  export COPERNICUSMARINE_SERVICE_PASSWORD="your_password"

6) Start API manually for quick test
- Run on Pi:
  source .venv/bin/activate
  uvicorn backend.server:app --host 0.0.0.0 --port 8000

7) Test from your PC browser
- Open:
  http://<RASPI_IP>:8000
- Health check:
  http://<RASPI_IP>:8000/health

8) Make backend persistent with systemd
- Copy service file:
  sudo cp deploy/raspi/leaflet-predictor.service /etc/systemd/system/
- Edit paths/user if needed:
  sudo nano /etc/systemd/system/leaflet-predictor.service
- Enable/start:
  sudo systemctl daemon-reload
  sudo systemctl enable leaflet-predictor
  sudo systemctl start leaflet-predictor
- Check logs:
  sudo journalctl -u leaflet-predictor -f

9) API URL for members (temporary)
Option A: Same LAN only
- URL is http://<RASPI_IP>:8000

Option B: Internet test URL (easy)
- Install cloudflared and run tunnel from Pi.
- This gives a public HTTPS URL without router port-forward.

Option C: Permanent domain
- Router port-forward 8000 to Pi.
- Use DDNS (DuckDNS, etc) + HTTPS proxy.

10) Frontend wiring (for GitHub Pages later)
- Frontend should call absolute API base URL from config.
- Recommended pattern:
  window.PREDICTOR_API_BASE = "https://your-api-url";
  fetch(window.PREDICTOR_API_BASE + "/api/simulate?..." )

11) Security baseline
- Do not expose Copernicus password in frontend.
- Set CORS_ALLOW_ORIGINS to your frontend domain, not *.
- Add rate limit in future (to protect Pi resources).

12) Migration to Render later
- render config already added: deploy/render.yaml
- Steps:
  1. Push repo to GitHub
  2. Create Render Web Service from repo
  3. Use render.yaml
  4. Set env vars in Render dashboard:
     COPERNICUSMARINE_SERVICE_USERNAME
     COPERNICUSMARINE_SERVICE_PASSWORD
     CORS_ALLOW_ORIGINS=https://<your-github-pages-domain>

13) Known operational note
- Simulations are CPU and network heavy.
- Pi works for testing and small group use.
- For larger usage, move backend to cloud and keep frontend static.
