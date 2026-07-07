# Test Case Document

Converted from the project's pytest suite (`tests/`). Each pytest function maps to one readable test case below.

## Basic Communication

### TC-BASIC-01
- **Objective**: Confirm the UART link works at the simplest level.
- **Preconditions**: Two connected UART endpoints (or loopback), ports open and configured.
- **Steps**: 1) Send one normal payload from TX. 2) Read it on RX.
- **Expected Result**: Data received matches data sent.

### TC-BASIC-02
- **Objective**: Verify each side can transmit and receive.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send data A→B, verify. 2) Send data B→A, verify.
- **Expected Result**: Both directions transfer data correctly.

### TC-BASIC-03
- **Objective**: Verify simultaneous two-way (full-duplex) communication.
- **Preconditions**: Two independent UART endpoints (not run on loopback).
- **Steps**: 1) Send from both ends at nearly the same time. 2) Read both sides.
- **Expected Result**: Both transfers complete correctly without interference.

## Configuration

### TC-CFG-01
- **Objective**: Confirm compatibility across supported UART configuration combinations.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) For each of 60 combinations of baud rate, data bits, parity, and stop bits, open ports with that config. 2) Send and receive a payload.
- **Expected Result**: Data transfers correctly for every valid combination (60 parameterized cases).

### TC-CFG-02
- **Objective**: Verify a bad baud-rate pairing does not behave like a correct transfer.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Configure TX and RX with different baud rates. 2) Send data and read the result.
- **Expected Result**: Received data is corrupted/garbled or the mismatch is otherwise detectable (not a clean match).

### TC-CFG-03
- **Objective**: Verify extended serial options are forwarded to the underlying serial layer.
- **Preconditions**: None (no hardware required).
- **Steps**: 1) Open a port with `xonxoff`, `rtscts`, `dsrdtr`, and `exclusive` set. 2) Inspect the underlying serial object's configuration.
- **Expected Result**: All four options are correctly applied on the serial object.

## Integrity

### TC-INT-01
- **Objective**: Check data integrity during a large transfer.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send a large payload. 2) Compare received bytes to sent bytes.
- **Expected Result**: Received data exactly matches sent data, no truncation or corruption.

### TC-INT-02
- **Objective**: Catch corruption that simple text payloads might miss.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Generate random byte data. 2) Send and receive it.
- **Expected Result**: Received data matches the random payload exactly.

### TC-INT-03
- **Objective**: Verify non-text (raw binary) payload handling.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send a payload of raw binary byte values. 2) Read it back.
- **Expected Result**: Binary payload is transferred without alteration.

### TC-INT-04
- **Objective**: Validate zero-length payload edge case.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send an empty payload. 2) Attempt to read a response.
- **Expected Result**: The system handles the empty send/read without error or crash.

### TC-INT-05
- **Objective**: Confirm control characters and special bytes survive the link correctly.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send bytes including `0x00`, `0xFF`, newline, carriage return, and tab. 2) Read the result.
- **Expected Result**: All special bytes are received unchanged.

## Timing

### TC-TIM-01
- **Objective**: Check transfer latency stays within a reasonable bound.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send a payload and record the timestamp. 2) Record the timestamp on receipt. 3) Compute the delay.
- **Expected Result**: Measured delay is within the acceptable threshold for the configuration.

### TC-TIM-02
- **Objective**: Verify the receiver handles slow, byte-by-byte delivery.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send one byte at a time with a delay between bytes. 2) Reassemble on the receive side.
- **Expected Result**: All bytes are received in order and reassembled correctly despite delays.

### TC-TIM-03
- **Objective**: Confirm timeout and empty-read behavior.
- **Preconditions**: Two connected UART endpoints (or one open port with no incoming data).
- **Steps**: 1) Attempt a read when no data is being sent. 2) Wait for the configured timeout.
- **Expected Result**: The read returns empty/times out cleanly rather than hanging or erroring unexpectedly.

### TC-TIM-04
- **Objective**: Provide a throughput/performance benchmark signal.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send a larger payload. 2) Measure total transfer time and compute bytes/second.
- **Expected Result**: Throughput is calculated and falls within an expected range for the configured baud rate.

## Protocol

### TC-PROTO-01
- **Objective**: Confirm frame build/parse logic works for a valid CRC-framed command.
- **Preconditions**: None (logic-level, but typically run with hardware).
- **Steps**: 1) Build a valid CRC-framed command. 2) Send it. 3) Parse the received frame.
- **Expected Result**: Frame is parsed successfully and matches the original command.

### TC-PROTO-02
- **Objective**: Confirm invalid framing is rejected.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send a frame with corrupted start/end markers. 2) Attempt to parse it.
- **Expected Result**: Parser detects and rejects the invalid frame.

### TC-PROTO-03
- **Objective**: Confirm truncated/incomplete frames are handled safely.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send an incomplete frame (missing bytes). 2) Attempt to parse it.
- **Expected Result**: Parser handles the partial frame without crashing, reporting it as incomplete/invalid.

### TC-PROTO-04
- **Objective**: Confirm bad CRC is detected.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Build a frame and corrupt its CRC value. 2) Parse the received frame.
- **Expected Result**: CRC mismatch is detected and reported.

### TC-PROTO-05
- **Objective**: Validate the sequence-number parser logic itself.
- **Preconditions**: None (no hardware required).
- **Steps**: 1) Construct a single sequence-numbered payload. 2) Parse it.
- **Expected Result**: Sequence number and payload are extracted correctly.

### TC-PROTO-06
- **Objective**: Verify out-of-order sequence values can be detected.
- **Preconditions**: None (no hardware required).
- **Steps**: 1) Provide a series of sequence numbers with one out of order. 2) Run the reorder-detection check.
- **Expected Result**: The out-of-order sequence is correctly flagged.

## Negative

### TC-NEG-01
- **Objective**: Confirm invalid device paths fail properly.
- **Preconditions**: None.
- **Steps**: 1) Attempt to open a non-existent serial port.
- **Expected Result**: A clear error/exception is raised; no silent failure.

### TC-NEG-02
- **Objective**: Confirm unsupported UART configuration is rejected.
- **Preconditions**: None.
- **Steps**: 1) Attempt to open a port using unsupported data-bit settings.
- **Expected Result**: Configuration is rejected with a clear error.

### TC-NEG-03
- **Objective**: Confirm misuse (sending without initialization) is caught cleanly.
- **Preconditions**: None.
- **Steps**: 1) Attempt to send data using an uninitialized/unopened serial object.
- **Expected Result**: A clear error is raised rather than an unhandled crash.

### TC-NEG-04
- **Objective**: Verify disconnect errors are handled on the transmit side.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Close the TX port. 2) Attempt to write to it.
- **Expected Result**: A disconnect/closed-port error is raised and handled.

### TC-NEG-05
- **Objective**: Verify closed-receiver errors are handled.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Close the RX port. 2) Attempt to read from it.
- **Expected Result**: A closed-port error is raised and handled.

## Stress and Soak

### TC-STR-01
- **Objective**: Check stability under sustained continuous traffic.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send many packets continuously. 2) Verify all are received correctly.
- **Expected Result**: All packets arrive intact with no data loss over the continuous run.

### TC-STR-02
- **Objective**: Check burst handling and buffering.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Send a burst of packets in rapid succession. 2) Verify all are received.
- **Expected Result**: All burst packets are received correctly without overflow/loss.

### TC-STR-03
- **Objective**: Check stability of repeated connection lifecycles.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Repeatedly open a port, send data, then close it, several cycles in a row.
- **Expected Result**: Every open/send/close cycle completes successfully with no resource leaks or failures.

### TC-STR-04
- **Objective**: Check order and integrity over a repeated sequence-numbered stream.
- **Preconditions**: Two connected UART endpoints.
- **Steps**: 1) Stream multiple sequence-numbered packets repeatedly. 2) Verify order and content on receipt.
- **Expected Result**: All packets arrive in the correct sequence with correct content.

### TC-STR-05
- **Objective**: Catch intermittent long-run issues (final stability/endurance check).
- **Preconditions**: Two connected UART endpoints; soak duration configured via `--soak-seconds`.
- **Steps**: 1) Run continuous communication for the configured soak duration. 2) Monitor for failures throughout.
- **Expected Result**: Communication remains stable for the entire duration with no unexpected failures.

---

**Summary**: 31 documented test cases above map to the project's 31 pytest functions. `TC-CFG-01` alone expands into 60 parameterized runs (different baud/data-bit/parity/stop-bit combinations), bringing the total executed pytest case count to 90.
