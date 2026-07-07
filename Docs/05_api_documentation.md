# API Documentation



## Health

### GET /api/health
- **Purpose**: Basic liveness check for the backend.
- **Method**: GET
- **URL**: `/api/health`
- **Request**: none
- **Response** (inferred): `{ "status": "ok" }`
- **Possible errors**: none expected — if this fails, the backend process itself isn't running.

## Ports

### GET /api/ports
- **Purpose**: List serial ports available on the host so the user can pick TX/RX devices.
- **Method**: GET
- **Request**: none
- **Response** (inferred): `{ "ports": ["/dev/ttyUSB0", "/dev/ttyUSB1", "/dev/serial0"] }`
- **Possible errors**: 500 if the underlying `pyserial` port-listing call fails.

## Communication

### POST /api/communicate
- **Purpose**: Send data from one configured UART port to another and return what was received.
- **Method**: POST
- **Request** (inferred): `{ "tx_port": "/dev/ttyUSB0", "rx_port": "/dev/ttyUSB1", "baudrate": 9600, "data": "hello" }`
- **Response** (inferred): `{ "sent": "hello", "received": "hello", "success": true, "duration_ms": 12 }`
- **Possible errors**:
  - 400 — invalid/missing port or bad UART configuration
  - 500 — serial exception (e.g. permission denied, device disconnected)

## Test Profiles

### GET /api/test-profiles
- **Purpose**: Return the predefined scenario groups (basic, configuration, integrity, timing, protocol, negative, stress/soak) available to run.
- **Method**: GET
- **Request**: none
- **Response** (inferred): `{ "profiles": ["basic", "config", "integrity", "timing", "protocol", "negative", "stress"] }`
- **Possible errors**: none expected (static/derived list).

## Saved Profiles

### GET /api/saved-profiles
- **Purpose**: List user-saved custom test profiles.
- **Method**: GET
- **Request**: none
- **Response** (inferred): `{ "profiles": [ { "id": 1, "name": "Quick check", "scenarios": ["basic", "protocol"] } ] }`
- **Possible errors**: 500 on database read failure.

### POST /api/saved-profiles
- **Purpose**: Create/save a new custom test profile.
- **Method**: POST
- **Request** (inferred): `{ "name": "Quick check", "scenarios": ["basic", "protocol"] }`
- **Response** (inferred): `{ "id": 1, "name": "Quick check", "scenarios": ["basic", "protocol"] }`
- **Possible errors**: 400 — invalid/empty profile data; 500 — database write failure.

### DELETE /api/saved-profiles/{profile_id}
- **Purpose**: Remove a saved test profile.
- **Method**: DELETE
- **Request**: `profile_id` in the URL path
- **Response** (inferred): `{ "deleted": true }`
- **Possible errors**: 404 — profile ID not found; 500 — database error.

## Start Tests

### POST /api/tests/run
- **Purpose**: Start a pytest run — either all scenarios or a selected group/profile.
- **Method**: POST
- **Request** (inferred): `{ "scope": "all" }` or `{ "scope": "protocol" }` or `{ "profile_id": 1 }`
- **Response** (inferred): `{ "run_id": "abc123", "status": "started" }`
- **Possible errors**: 400 — invalid scope/profile; 409 — a run is already in progress; 500 — failure to launch pytest subprocess.

## Get Test Run Status

### GET /api/tests/{run_id}
- **Purpose**: Poll the status of a running or completed test run.
- **Method**: GET
- **Request**: `run_id` in the URL path
- **Response** (inferred): `{ "run_id": "abc123", "status": "running" | "completed" | "failed", "progress": "12/31" }`
- **Possible errors**: 404 — unknown `run_id`.

## Get Test Results for a Run

### GET /api/tests/{run_id}/results
- **Purpose**: Return detailed pass/fail results for a completed run.
- **Method**: GET
- **Request**: `run_id` in the URL path
- **Response** (inferred): `{ "run_id": "abc123", "results": [ { "test_name": "test_basic_send_receive", "status": "passed", "duration_ms": 45 } ] }`
- **Possible errors**: 404 — unknown `run_id`; 409 — run still in progress (results not ready).

## Test Run History and Dashboard

### GET /api/test-runs
- **Purpose**: List past test run history.
- **Method**: GET
- **Request**: none (may support pagination/filter query params)
- **Response** (inferred): `{ "runs": [ { "run_id": "abc123", "started_at": "...", "status": "completed", "pass_count": 28, "fail_count": 3 } ] }`
- **Possible errors**: 500 on database read failure.

### GET /api/dashboard
- **Purpose**: Return a summary of overall test/communication activity for the UI dashboard.
- **Method**: GET
- **Request**: none
- **Response** (inferred): `{ "total_runs": 10, "last_run_status": "completed", "pass_rate": 0.91 }`
- **Possible errors**: 500 on database read failure.

## Communication Logs

### GET /api/communications
- **Purpose**: Return recent manual communication attempts (from `/api/communicate`) for review.
- **Method**: GET
- **Request**: none (may support a `limit` query param)
- **Response** (inferred): `{ "logs": [ { "timestamp": "...", "tx_port": "/dev/ttyUSB0", "rx_port": "/dev/ttyUSB1", "success": true } ] }`
- **Possible errors**: 500 on database read failure.

## Download CSV Results

### GET /api/results
- **Purpose**: Export stored test results as a CSV file (`uart_results.csv`).
- **Method**: GET
- **Request**: none
- **Response**: CSV file download (`Content-Type: text/csv`)
- **Possible errors**: 500 — export/query failure.
