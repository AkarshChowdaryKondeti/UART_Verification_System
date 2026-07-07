# Software Requirements Specification (SRS)

## 1. Functional Requirements

| ID | Requirement |
|---|---|
| FR-1 | The system shall list available serial ports so the user can select TX/RX devices. |
| FR-2 | The system shall allow sending data from one configured UART port to another and display the result (`POST /api/communicate`). |
| FR-3 | The system shall provide a set of predefined test profiles (scenario groups: basic, configuration, integrity, timing, protocol, negative, stress/soak). |
| FR-4 | The system shall allow users to save and reuse custom test profiles (`saved-profiles` endpoints). |
| FR-5 | The system shall allow starting a pytest-based test run (all scenarios or a selected group) via the API (`POST /api/tests/run`). |
| FR-6 | The system shall report the status of a running or completed test run (`GET /api/tests/{run_id}`). |
| FR-7 | The system shall return detailed results for a given test run (`GET /api/tests/{run_id}/results`). |
| FR-8 | The system shall store test runs, their results, and communication logs in a persistent SQLite database. |
| FR-9 | The system shall provide test run history and a dashboard summary (`GET /api/test-runs`, `GET /api/dashboard`). |
| FR-10 | The system shall provide recent communication logs (`GET /api/communications`). |
| FR-11 | The system shall allow exporting results as CSV (`GET /api/results`). |
| FR-12 | The frontend shall provide a Communication panel to configure ports, send data, and view received output. |
| FR-13 | The frontend shall provide a Testing panel to run all or selected scenario groups and see live run updates. |
| FR-14 | The frontend shall provide a Reports panel to browse run history and filtered results. |
| FR-15 | The UART library shall support duplex transfer, chunked transmission, CRC-framed commands, and sequence-numbered streams. |

## 2. Non-Functional Requirements

| ID | Requirement |
|---|---|
| NFR-1 | **Reliability** — communication and test execution must correctly detect and report errors (invalid ports, disconnects, bad CRC, timeouts) rather than failing silently. |
| NFR-2 | **Portability** — the backend must run on standard Linux and Raspberry Pi OS without code changes, only device path configuration. |
| NFR-3 | **Usability** — a developer should be able to start the full stack with one command (`./run.sh`) and use the UI without reading test code. |
| NFR-4 | **Performance** — timing/throughput tests exist specifically to catch unacceptable latency or slow transfer, so the core comm layer must not introduce artificial delay. |
| NFR-5 | **Maintainability** — UART logic is centralized in a single reusable module (`uart/comm.py`) rather than duplicated across tests and backend code. |
| NFR-6 | **Data persistence** — test and communication history must survive backend restarts (stored in SQLite, not memory). |

## 3. Assumptions
- The machine running the project has physical access to the serial devices (USB-UART adapters or Pi GPIO UART) — this is not a simulated/mocked environment for hardware-tagged tests.
- The user has permission to access serial devices (`dialout`/`tty` group membership on Linux).
- Only one test run / communication session is expected at a time (no concurrent multi-user use case is described).
- Python 3 and Node.js are available on the host machine.

## 4. Constraints
- UART hardware setups are limited to point-to-point configurations (loopback, dual-adapter, USB+GPIO) — no multi-drop bus protocols.
- Frontend expects the backend at `http://127.0.0.1:8000/api` by default (overridable via `VITE_API_BASE`).
- Some tests require specific hardware (marked `hardware` in the test reference) and are skipped automatically when the setup doesn't provide two independent endpoints (e.g. loopback).
- Long-running soak tests require an explicit duration flag (`--soak-seconds`) and are not run by default.
