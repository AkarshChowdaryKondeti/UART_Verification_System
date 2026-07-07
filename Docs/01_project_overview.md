# UART Control Center — Project Overview

## Project Description
UART Control Center is a full-stack tool for validating UART (serial) communication on Raspberry Pi and Linux systems. It combines a reusable Python UART communication library, a pytest-based validation suite (90 test cases across 7 categories), a FastAPI backend for live communication and test orchestration, and a React frontend for interacting with hardware and reviewing results.

The project supports three common lab setups: a single USB-to-UART loopback, two USB-to-UART adapters cross-connected, or a USB-to-UART adapter paired with Raspberry Pi GPIO UART.

## Problem Statement
Testing UART communication is usually done manually — wiring up adapters, writing one-off scripts, and eyeballing serial output. This is slow, hard to repeat, and easy to get wrong (e.g. missing a baud mismatch or silently dropping bytes). There was no single tool that let a developer send/receive data live, run a structured regression suite against the same hardware, and review historical results, all from one place.

## Objectives
- Provide a reusable UART communication layer that handles configuration, duplex transfer, framing, and error cases consistently.
- Provide an automated pytest suite covering basic communication, configuration, data integrity, timing, protocol framing, negative cases, and stress/soak testing.
- Expose live UART communication and test execution through a web API.
- Give a simple UI to send/receive data manually and to trigger and review test runs, without touching the command line.
- Persist test runs, results, and communication logs so past runs can be reviewed later.

## Scope

**In scope:**
- Point-to-point UART communication over supported lab setups (loopback, dual-adapter, USB+GPIO).
- Manual send/receive via the web UI and API.
- Automated test execution (all scenarios or selected groups) triggered from the UI or API.
- Storage of test runs, test results, and communication logs in SQLite.
- CSV export of results.

**Out of scope:**
- Multi-drop or bus-based serial protocols (e.g. RS-485 multi-node).
- Communication over network-based virtual serial ports.
- User authentication / multi-user access control.
- Production deployment concerns (HTTPS, containerization, CI/CD pipelines) — this is a local validation tool.

## Technologies Used

| Layer | Technology |
|---|---|
| Backend API | FastAPI (Python), Uvicorn |
| UART communication | pyserial (via `uart/comm.py`) |
| Test suite | pytest |
| Frontend | React, Vite |
| Database | SQLite |
| Result export | CSV |
| Dev tooling | `run.sh` launcher script, Python venv, npm |

## Hardware Requirements

| Setup | Description | Notes |
|---|---|---|
| `usb_loopback` | One USB-to-UART adapter, TX tied to its own RX, GND connected | Simplest setup; duplex/mismatched-baud tests are skipped automatically since only one endpoint exists |
| `dual_usb` | Two USB-to-UART adapters, cross-connected (TX↔RX, GND↔GND) | Default ports: TX = `/dev/ttyUSB0`, RX = `/dev/ttyUSB1` |
| `usb_gpio` | One USB-to-UART adapter cross-connected with Raspberry Pi GPIO UART | Uses `/dev/serial0` (recommended), `/dev/ttyAMA0`, or `/dev/ttyS0` depending on the Pi |

Running on Raspberry Pi also requires the serial port to be enabled via `raspi-config` and the user to be in the `dialout` (and sometimes `tty`) group to avoid permission errors.

## Expected Outcome
A developer can plug in a supported UART setup, start the project with a single command (`./run.sh`), send/receive data live through a browser UI, run the full 90-case validation suite (or a subset) against the real hardware, and review pass/fail history and logs — without writing any test-runner code by hand.
