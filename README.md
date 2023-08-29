# lnplay.live requirements

This is what we intend to accomplish as a MINIMUM VIABLE PRODUCT for the tabconf-2023 hackathon.
# lnplay-frontend

The front-end is a [lnmessage-enabled](https://github.com/aaronbarnardsound/lnmessage) PWA that interfaces with a backend core lightning node (CLN) over the `--experimental-websocket-port` (HTTP for local, 443/HTTPS/TLS-1.3 for remote hosts). Embedded in the front-end code is a well-known rune that authenticates client requests to the back end CLN node. People who visit `lnplay.live` will see information about the project as well as be given an opportunity to purchase (via lightning-only) an ephemeral regtest lightning environment that can be used for small bitcoin meetups, bitcoin conferences, or orange-pilling your family.

The front-end application SHOULD accept a list of BOLT12 offers at build-time if possible.
## Product Definition

The Product Definition is shown below.

|PRODUCT|CLN_COUNT|REQUIRED/OPTIONAL for tabconf hackathon|
|---|---|---|
|A|8|REQUIRED|
|B|16|OPTIONAL|
|C|32|OPTIONAL|
|D|64|OPTIONAL|

For the MVP, the backend will CALCULATE the expiration date of deployed environment based on the AMOUNT_PAID. Invoices associated with a particular BOLT12 Product SKU determines the CLN_COUNT (and thus VM sizing). Product SKUs are represented by BOLT12 offers. These BOLT12 offers get embedded into the front-end and are used to fetch BOLT11 invoices from the backend CLN node.
# lnplay-backend [RESPONSIBILITY: farscapian, etc]

The backend consists of the following efforts:

## Infrastructure Requirements

1. Stand up a LXD cluster for providing compute, memory, and storage for the that exposes an LXD interface on a restricted port. The LXC client in the provisioning plugin accesses this service to provision VMs.

3. A rune needs to be issued on this CLN node in accordance with least privilege and should be rate-limited (admin rune OK for demo). This rune gets embedded in the front-end for authenticating client requests. Method authorization should be based on WHITELIST with the following method: [`fetchinvoice`](https://docs.corelightning.org/reference/lightning-fetchinvoice) and [`waitinvoice`](https://docs.corelightning.org/reference/lightning-waitinvoice).

4. The CLN node SHALL run a bash plugin that does the following:  
  a) a method that that gets [executed whenever a payment is received](https://docs.corelightning.org/docs/event-notifications). The plugin will determine if the payment is associated with known BOLT12 offers representing product SKUs. If indeed it does, the following occurs:
     i) the plugin will spin up a new VM on a remote LXD cluster (i.e., remote) using [Sovereign Stack](https://www.sovereign-stack.org/). The LXD remote is defined by environment variables passed to the CLN instance at runtime. 
     ii) deploying roygbiv-stack to it in accordance with the specification submitted by the customer. As a last step, the function stores the connection strings in the CLN database as a JSON structure.
  c) an rpcmethod that allows the web app to check on the status of their order. This method would take as an argument the payment preimage and return the JSON structure stored in the CLN database for the order. The web app would then display the information returned from the call: 1 the connection strings (REQUIRED) and 2) a PDF with QR representations of the connections.

5. Stand up a VM in AWS that will host the `lnplay.live` website. That specific deployment will have 
5. The front-end web app will need to be dockerized and an option added in ROYGBIV-stack for deploying the web-UI at the root of the app.

Upon payment, the user is redirected to domain.TLD/orders/preimage. They SHOULD be directed to store the URL in their in their password manager so they can reference order later. 


As a side note, we need a separate script that culls instances. Instances should have a expired_after date upon which to make a decision. the roygbiv down script should be executed, then the VM turned off and deleted. Docker vol should remain.

# Architecture Diagram

TODO

# Development Environment

Backend development requires this repo and a local docker engine. Debian-based is usually best.

git clone --recurse-submodules https://github.com/farscapian/roygbiv-stack ~/git/tabconf2023

And checkout the `tabconf2023` branch.

# Future work

List some ideas of future work for the project that IS NOT part of the MVP.