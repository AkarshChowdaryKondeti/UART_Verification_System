# Maintenance Guide

## 1. Troubleshooting

| Symptom | Likely Cause | Action |
|---|---|---|
| Backend fails to start | Missing Python deps or wrong venv | Re-run `pip install -r requirements.txt` inside `.venv` |
| Frontend fails to start | Missing node modules | Re-run `npm install` in `frontend/` |
| API calls fail from the UI | Backend not running, or wrong `VITE_API_BASE` | Confirm backend is up at `http://127.0.0.1:8000`, check env var |
| Test run stuck in "running" | pytest subprocess crashed or hung | Check backend logs/process; restart backend and re-trigger the run |
| CSV export is empty/stale | No results in the database yet, or export not re-generated | Confirm a completed test run exists before calling `GET /api/results` |

## 2. Common UART Issues

| Issue | Cause | Fix |
|---|---|---|
| `Permission denied` opening a serial device | User not in `dialout`/`tty` group | `sudo usermod -aG dialout $USER` (and `tty` if needed), then re-login |
| No data received (loopback) | TX not physically tied to RX, or GND not connected | Recheck wiring: TX→RX on same adapter, GND connected |
| No data received (dual adapter) | Cross-wiring mistake | Confirm A.TX→B.RX, A.RX→B.TX, A.GND→B.GND |
| Garbled data | Baud rate mismatch between TX and RX | Ensure both sides use the same baud/parity/stop-bit/data-bit configuration |
| Pi UART not appearing | UART disabled in OS config | Enable Serial Port in `raspi-config`, disable serial console/login shell |
| Duplex tests always skipped | Running on `usb_loopback` (only one endpoint) | Expected behavior — use `dual_usb` or `usb_gpio` for duplex testing |
| Intermittent drops during long runs | Cabling/adapter quality, or system load | Investigate via soak test (`test_duration_based_soak`) with realistic duration |

## 3. Log / Data Locations

| Data | Location |
|---|---|
| Test runs, test results, communication logs, saved profiles | `uart_control_center.db` (SQLite) |
| CSV export of results | `uart_results.csv` (generated on demand from the database) |
| Backend process output | Terminal running `uvicorn` / `./run.sh` (not written to a dedicated log file per the README) |

## 4. Database Maintenance
- `uart_control_center.db` is the single source of truth — back it up before major schema changes or bulk deletions.
- If the database grows large from many test runs, periodically archive or prune old `test_runs`/`test_results` rows (e.g. keep the last N runs).
- The CSV export (`uart_results.csv`) is regenerated from the database on request, so it does not need independent backup — the database is authoritative.
- If the schema changes, saved profiles and historical results may need a migration step, since no migration tooling is mentioned in the current setup.

## 5. Future Maintenance Considerations
- Add a dedicated log file for backend errors and pytest subprocess output, rather than relying on console output only.
- Consider a lightweight schema migration approach if the database structure evolves (e.g. Alembic for SQLAlchemy models, if adopted).
- Add automated cleanup/retention for old test runs to keep the SQLite file from growing indefinitely.
- Consider containerizing the backend for easier deployment across different Linux/Pi environments, if the tool needs to be shared beyond a single dev machine.
- Extend hardware support notes if new UART setups (e.g. RS-485, network serial) are added later, since the current design assumes point-to-point UART only.
