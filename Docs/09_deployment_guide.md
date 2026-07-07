# Deployment Guide

## 1. Prerequisites
- Linux or Raspberry Pi OS host.
- Python 3 installed.
- Node.js and npm installed.
- Physical UART hardware set up in one of the supported configurations (`usb_loopback`, `dual_usb`, or `usb_gpio`).
- On Raspberry Pi: UART enabled via `raspi-config` (Interface Options → Serial Port), and the login-shell-over-serial option disabled if the UART is needed for the app.
- Current user is a member of the `dialout` (and sometimes `tty`) group, to access serial devices without permission errors.

## 2. Installation

### Python dependencies
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Node dependencies
```bash
cd frontend
npm install
cd ..
```

## 3. Running Backend and Frontend Together
The simplest option is the provided launcher script:
```bash
./run.sh
```
This will:
- Create `.venv/` automatically if it doesn't exist, and install Python requirements.
- Run `npm install` automatically if `frontend/node_modules` is missing.
- Start the FastAPI backend at `http://127.0.0.1:8000`.
- Start the React frontend at `http://127.0.0.1:5173`.
- Stop both processes on a single `Ctrl+C`.

You can override the virtual environment location:
```bash
VENV_DIR=/path/to/venv ./run.sh
```

## 4. Running Backend Manually
```bash
uvicorn backend.app:app --reload
```

## 5. Running Frontend Manually
```bash
cd frontend
npm run dev
```
By default, the frontend expects the backend at `http://127.0.0.1:8000/api`. Override with:
```bash
VITE_API_BASE=http://<host>:<port>/api npm run dev
```

## 6. Running Tests
Run the non-slow suite:
```bash
pytest tests -m "not slow" -q
```

Run hardware-tagged tests against specific ports:
```bash
pytest tests -m hardware -q --tx-port /dev/ttyUSB0 --rx-port /dev/ttyUSB1
```

Run a long soak test:
```bash
pytest tests/test_stress_uart.py::test_duration_based_soak -m hardware -q --soak-seconds 3600
```

Tests can also be triggered from the UI/API via `POST /api/tests/run`, which uses the same underlying pytest suite.

## 7. Common Issues

| Issue | Cause | Fix |
|---|---|---|
| `[Errno 13] could not open port ... Permission denied` | User not in the serial-access group | `sudo usermod -aG dialout $USER` (and possibly `sudo usermod -aG tty $USER`), then log out/in |
| Serial port not found (`/dev/ttyAMA0`, `/dev/serial0`) | UART not enabled on the Pi | Enable via `sudo raspi-config` → Interface Options → Serial Port |
| Duplex/mismatched-baud tests fail unexpectedly on loopback | Loopback only has one physical endpoint | These tests are expected to auto-skip on `usb_loopback`; if not skipping, check hardware/profile selection |
| Frontend can't reach backend | Wrong API base URL | Set `VITE_API_BASE` to match where the backend is actually running |
| `run.sh` doesn't pick up dependency changes | Cached venv/node_modules | Delete `.venv/` or `frontend/node_modules` and rerun `./run.sh` |
