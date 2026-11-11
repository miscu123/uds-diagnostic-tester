# uds-diagnostic-tester

Modular UDS diagnostic tester with a configurable simulated ECU, running over a virtual CAN bus.

Built as a software engineering project: requirements, layered architecture, automated testing, CI and documentation. The implementation covers a selected subset of ISO 14229 (UDS) and ISO 15765-2 (ISO-TP) and is not a certified implementation.

## Features

- [ ] UDS tester (client) with a command-line interface
- [ ] ISO-TP layer: single-frame and multi-frame messages, flow control, timeouts
- [ ] UDS services:
  - [ ] DiagnosticSessionControl (0x10)
  - [ ] ReadDataByIdentifier (0x22)
  - [ ] ReadDTCInformation (0x19)
  - [ ] TesterPresent (0x3E)
  - [ ] Negative response handling
- [ ] Configurable ECU simulator (YAML/JSON profiles)
- [ ] Fault injection: delays, wrong responses, missing replies
- [ ] Replayable communication logs
- [ ] Scenario-based tests with pass/fail reports
- [ ] Unit and integration tests, CI on every commit

## Architecture

```
CLI / Application
        |
   UDS services
        |
 ISO-TP transport
        |
  CAN interface  <-->  virtual CAN (vcan / python-can virtual)
        |
   ECU simulator
```

Each layer talks to the next through a small, documented interface. UML diagrams are in `docs/`.

## Getting started

### Requirements

- Python 3.10+
- Linux with SocketCAN (optional, for `vcan`), or the python-can virtual interface

### Setup

```bash
git clone https://github.com/miscu123/uds-diagnostic-tester.git
cd uds-diagnostic-tester
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Optional: SocketCAN virtual bus

```bash
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

### Run the simulated ECU

```bash
python -m uds_sim --config profiles/example_ecu.yaml --channel vcan0
```

### Use the tester

```bash
python -m uds_tester --channel vcan0 read-did 0xF190   # read VIN
python -m uds_tester --channel vcan0 read-dtc
```

## ECU profile example

```yaml
ecu:
  tester_id: 0x7E0
  ecu_id: 0x7E8
  dids:
    0xF190: "WVWZZZ1JZXW000001"
  dtcs:
    - code: 0x123456
      status: 0x09
  faults:
    response_delay_ms: 0
    drop_probability: 0.0
```

## Testing

```bash
pytest tests/unit
pytest tests/integration
```

## Project structure

```
.
├── src/
│   ├── can_interface/
│   ├── isotp/
│   ├── uds/
│   ├── ecu_sim/
│   └── cli/
├── profiles/
├── scenarios/
├── tests/
├── docs/
└── .github/workflows/
```

## Out of scope

SecurityAccess, WriteDataByIdentifier, RoutineControl, ECU reset, firmware flashing, DoIP, CAN FD, K-Line, GUI. Real hardware use, if any, is read-only.

## Documentation

- User guide: `docs/user-guide.md`
- Developer guide: `docs/developer-guide.md`
- Requirements and user stories: `docs/requirements.md`

## License

MIT
