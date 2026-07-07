# Test Plan

## 1. Testing Objectives
- Verify that basic UART send/receive and bidirectional/duplex communication work correctly.
- Verify that different UART configurations (baud rate, data bits, parity, stop bits) behave as expected, including deliberately mismatched configurations.
- Verify data integrity across payload types: large, random, binary, empty, and special-character payloads.
- Verify timing behavior: latency, inter-byte delay handling, timeouts, and throughput.
- Verify protocol-level correctness: CRC-framed commands, sequence numbering, and rejection of invalid/partial frames.
- Verify negative/error-handling paths: invalid ports, unsupported configuration, misuse, and disconnects.
- Verify stability under stress and long-duration (soak) conditions.

## 2. Test Environment

| Item | Detail |
|---|---|
| OS | Linux / Raspberry Pi OS |
| Python | 3.x with `pytest`, `pyserial` |
| Test runner | `pytest` via `pytest.ini` config, or triggered through the backend (`POST /api/tests/run`) |
| Markers | `hardware` (requires physical UART), `slow` (long-running, excluded by default) |
| CLI options | `--tx-port`, `--rx-port`, `--soak-seconds` |

## 3. Hardware Setup

| Setup | Wiring | Use case |
|---|---|---|
| `usb_loopback` | One USB-to-UART adapter, TX→RX on the same adapter, GND connected | Quick checks; duplex and mismatched-baud tests are auto-skipped (only one endpoint) |
| `dual_usb` | Two USB-to-UART adapters cross-connected (TX↔RX, GND↔GND); default `/dev/ttyUSB0` / `/dev/ttyUSB1` | Full suite, including duplex |
| `usb_gpio` | USB-to-UART adapter cross-connected with Pi GPIO UART (`/dev/serial0`, `/dev/ttyAMA0`, or `/dev/ttyS0`) | Full suite on Raspberry Pi hardware |

## 4. Test Categories

| Category | Focus | Test count |
|---|---|---|
| Basic | Simple send/receive, bidirectional, duplex | 3 |
| Configuration | Baud/parity/stop/data-bit combinations, mismatches, extended params | 3 functions (60 parameterized cases from `test_uart_configurations`) |
| Integrity | Large, random, binary, empty, special-character payloads | 5 |
| Timing | Latency, inter-byte delay, timeout, throughput | 4 |
| Protocol | Valid/invalid/partial frames, CRC validation, sequence parsing | 6 |
| Negative | Invalid ports, unsupported params, misuse, disconnects | 5 |
| Stress/Soak | Continuous stream, bursts, reconnect cycles, sequence streams, duration-based soak | 5 |

**Total**: 7 groups, 31 test functions, 90 pytest cases (the extra cases come from the 60-way parameterization of `test_uart_configurations`).

## 5. Acceptance Criteria
- All non-hardware-tagged tests pass without physical UART hardware attached.
- All hardware-tagged tests pass on at least one of the three supported setups (loopback, dual-usb, usb+gpio), with hardware-dependent duplex tests correctly skipped on loopback.
- Negative-path tests correctly raise/report errors rather than passing silently.
- CRC and framing tests correctly reject corrupted or partial data.
- The soak test completes for the configured duration without a communication failure.
- Test runs triggered through the API produce the same pass/fail outcome as running `pytest` directly from the command line.
