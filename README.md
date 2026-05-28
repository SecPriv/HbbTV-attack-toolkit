# HbbTV Attack Toolkit

A modular research toolkit for analyzing the security of Smart TVs through the HbbTV runtime.

This repository contains the proof-of-concept toolkit used in our study of HbbTV-enabled Smart TVs. It combines:

- a Python control server
- an HbbTV client application served to the TV
- a Socket.IO control channel between the TV and the operator console

The toolkit was designed to support controlled, reproducible security experiments on lab-owned devices.

This project is intended **exclusively for research, testing, and educational use in controlled environments**. Do **not** use this toolkit against devices, networks, or broadcast infrastructure you do not own or explicitly have permission to test. Real-world broadcast injection and unauthorized device interaction may be illegal and harmful.

## What the toolkit can do

Once the HbbTV application is launched on a compatible Smart TV, the operator can interact with connected clients from the terminal menu.

Current capabilities include:

- display connected client information
- request HbbTV/browser/device metadata from the client
- trigger denial-of-service style UI disruption
- show and remove a fake banner overlay
- display a phishing-style popup and collect submitted input
- switch TV channels
- start a local network scan from the TV browser context
- issue HTTP requests from the TV to local or external targets
- evaluate custom JavaScript on the target
- redirect the target to another URL
- reload the target application

These actions are implemented through the Python entry point and helper modules in the repository. The HbbTV-side assets are located in the `hbbtv/` directory. The repository currently includes files such as `attack_toolkit.py`, `ClientManager.py`, `dos.py`, `fakenews.py`, `phishing.py`, `net_scan.py`, `http_request.py`, and the HbbTV client files under `hbbtv/`. 

## Repository structure

```markdown
.
├── attack_toolkit.py        # Main Flask/Socket.IO server and terminal UI
├── ClientManager.py         # Connected-client tracking
├── request_data.py          # Requests for configuration/app metadata
├── dos.py                   # DoS trigger
├── fakenews.py              # Fake banner controls
├── phishing.py              # Popup/phishing controls
├── net_scan.py              # Local network scan trigger
├── http_request.py          # HTTP requests from the target
├── channel_switch.py        # Channel switching actions
├── prompt.py                # Terminal menu helpers
├── hbbtv/
│   ├── entry.html           # HbbTV entry point
│   ├── entry.js             # Client-side logic
│   ├── rc-buttons.js        # Remote-control handling
│   ├── webscan.js           # Browser-based network scanning
│   ├── abort-controller.js  # Compatibility helper
│   ├── socket.io.min.js     # Socket.IO client
│   └── ...
└── requirements.txt         # Python dependencies
```

## Requirements

- Python 3.6 or newer
- A machine reachable by the target TV over the network
- A compatible HbbTV-capable Smart TV in a **controlled lab setup**
- A way to deliver the HbbTV application to the TV (for example, through an injected/custom DVB stream with an AIT that points to this server)

The current Python dependencies are listed in `requirements.txt` and include Flask, Flask-SocketIO, colorlog, and simple-term-menu.

## Installation

Create a virtual environment and install the dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Running the toolkit

Start the control server:

```bash
python attack_toolkit.py
```

By default, the server listens on:

- host: `0.0.0.0`
- port: `5000`

You can also choose a custom bind address and port:

```bash
python attack_toolkit.py -ip 0.0.0.0 -p 5000
```

After startup, the toolkit:

1. launches a Flask server
2. serves the HbbTV client from the `hbbtv/` directory
3. waits for incoming Socket.IO client connections
4. opens a terminal-based operator menu

## Serving the HbbTV application

The HbbTV application is served from the `/hbbtv/` path by the Python server.

The HTML entry point is delivered with the HbbTV MIME type and cache-control headers to improve compatibility with HbbTV runtimes.

In practice, the Smart TV must be directed to load this application through a broadcast-delivered AIT or another suitable HbbTV launch mechanism.

## Typical workflow

A typical experiment looks like this:

1. Start the toolkit server.
2. Make the server reachable from the target TV.
3. Launch the HbbTV application on the TV through your broadcast/test setup.
4. Wait for the client to connect back to the server.
5. Use the terminal menu to inspect the client and trigger actions.
6. Review output artifacts written to disk.

## Output artifacts

The toolkit stores results from different actions in separate directories, including:

- `config/`
- `appObject/`
- `appMgr/`
- `phishing/`
- `netscan/`
- `requests/success/`
- `requests/fail/`

These files can be useful for later analysis and for documenting experimental results.

## Notes on specific features

### Browser and environment fingerprinting

The toolkit can request browser-related and HbbTV-related information from the connected client. In our experiments, this was used together with feature-based checks in the HbbTV application to estimate the browser baseline and supported APIs.

### Local network scanning

The local network scan is browser-based and runs from inside the embedded HbbTV browser context. It does **not** provide a full network inventory. Instead, it identifies hosts that are reachable and observable through the networking primitives available to the target browser.

### HTTP requests

After a scan, the operator can select discovered targets and instruct the TV to send HTTP requests to them. Requests can also be sent directly to a manually entered URL.

### Dangerous actions

Some actions, such as JavaScript evaluation and channel switching, can cause you to lose access to the target or interrupt the current experiment. Use them carefully.

## Limitations

This toolkit is a research prototype, not a production framework.

Some capabilities are highly dependent on:

- the TV vendor and model
- the HbbTV version
- the embedded browser version
- vendor-specific runtime restrictions
- the network environment
- the broadcast delivery setup

As a result, behavior may differ significantly across devices.

## Reproducing experiments on a TV

To use the toolkit on a real device, launch the HbbTV application on the target TV. In our workflow, this requires creating a DVB stream with an injected/custom AIT pointing to the server hosting this toolkit.

For general background on broadcast-launched HbbTV applications, see the official HbbTV developer documentation:

- https://developer.hbbtv.org/guide/launching-hbbtv-applications-from-a-broadcast-channel/building-a-broadcast-ait/

You may also want to document your exact broadcast/modulation setup separately, since this repository focuses on the application/toolkit side.

## License

This project is licensed under the Apache License 2.0. See the `LICENSE` file for details.

