# lnplay.live requirements

This is what we intend to accomplish as a MINIMUM VIABLE PRODUCT for the tabconf-2023 hackathon. `lnplay.live` is a public website allowing anyone to purchase (via lightning-only) an ephemeral regtest lightning environment that can be used to educate small bitcoin meetups, bitcoin conference attendees, board-rooms, etc. It's a fun an educational experience helpful in orange-pilling your target audience.

# Product Definition

|PRODUCT_SKU|CLN_COUNT|REQUIRED/OPTIONAL|
|---|---|---|
|A|8|REQUIRED|
|B|16|OPTIONAL|
|C|32|OPTIONAL|
|D|64|OPTIONAL|

For the MVP, the backend will CALCULATE the expiration date of the deployment based on the AMOUNT_PAID (REQUIRED). Invoices associated with a particular BOLT12 Product SKU determines the CLN_COUNT (and thus VM sizing). These BOLT12 offers get embedded into the front-end during build time and are used internally only (i.e., the user never sees the BOLT12 offer). They are used to fetch BOLT11 invoices (via [`fetchinvoice`](https://docs.corelightning.org/reference/lightning-fetchinvoice)) from the backend CLN node, which are then shown to the user at checkout.

# lnplay-frontend

The front-end is a [lnmessage-enabled](https://github.com/aaronbarnardsound/lnmessage) PWA that interfaces with a backend core lightning node (CLN) over the `--experimental-websocket-port` (HTTP for local, 443/HTTPS/TLS-1.3 for remote hosts). Embedded in the front-end code is a well-known rune that authenticates client requests to the CLN node. The front-end application SHOULD accept this rune AND a list of BOLT12 offers at build-time if possible (see future work). Each BOLT12 Offer represents a Product SKU.

OPTIONAL - the front-end MAY create a PDF containing connection QR codes encoding connection URLs. Upon payment, the user is redirected to domain.TLD/orders/preimage. They SHOULD be directed to store the URL in their in their password manager so they can reference the order later. 

## Rune

A rune needs to be issued by a backend CLN node in accordance with least privilege and should be rate-limited (admin rune OK for demo). This rune gets embedded in the front-end and is used for authenticating client requests to the backend websocket endpoint. Method authorization should be based on WHITELIST with the following methods: [`fetchinvoice`](https://docs.corelightning.org/reference/lightning-fetchinvoice) and [`waitinvoice`](https://docs.corelightning.org/reference/lightning-waitinvoice), and `lnplaylive-orderstatus`.

# lnplay-backend

The backend consists of the following efforts:

## Plugin Provisioning Infrastructure Requirements

Before the hackathon, a LXD cluster providing compute, memory, and storage will be provisioned and accessible at `backend.lnplay.live:8443` (access is IP white-listed). The LXC client in the provisioning plugin accesses this service to create projects, provision VMs, and deploy clams-server. This should be in place BEFORE the hackathon.

## CLN Provisioning Plugin

A cln plugin written in bash with two primary functions:  
  
  a) an event that that gets [executed whenever a BOLT11 invoice is paid](https://docs.corelightning.org/docs/event-notifications). The plugin will determine if the payment is associated with known BOLT12 Product Offer ([example](https://github.com/daGoodenough/bolt12-prism/blob/main/prism-plugin.py)) representing product SKUs. If it is, the following occurs:

     i) when the plugin runs for the first time, there may be no remotes available. These are passed in by setting environment variables and get created before the plugin continues. So a new remote gets created and the LXD client is switched to it.
     ii) the plugin will create a new LXD project and switch to it. The project name includes the expiration date (in unix timestamp).
     iii) the plugin will spin up a new VM using [`ss-up`](https://www.sovereign-stack.org/ss-up/) on a remote LXD cluster using a custom environment file.
     iv) As a last step, the stores the connection strings in the CLN database as a JSON structure, retrievable using (b).
  
  b) a rpcmethod `lnplaylive-orderstatus <pre_image>` that allows the frontend web app to check on the status of an order. This method would take as an argument the payment pre-image and return a JSON document containing connection strings for the deployment. The front-end can poll `lnplaylive-orderstatus` and display connection details as be become available.

## Integrate front-end into roygbiv-stack/tabconf2023

The front-end web app will need to be dockerized and an option added `DEPLOY_LNPLAYLIVE_PLUGIN=true` in ROYGBIV-stack for deploying the web-UI at the root of the app.

## Hosting for `lnplay.live`

To serve the `lnplay.live` webapp to the public, a VM will be created on AWS and ROYGBIV-stack will deployed to the VM with DEPLOY_LNPLAYLIVE_PLUGIN=true.

## A script that culls instances

Each project name includes the expiration date (in UNIX timestamp). So, a script needs to be created that identifies expired projects and prunes them (culling). This involves de-provisioning clams-server and running [`ss-down`](https://www.sovereign-stack.org/ss-down/) .

# Architecture Diagram

TODO

# Development Environment

Backend development requires the following repo and a local docker engine. Debian-based is usually best. Recommend to `git clone --recurse-submodules https://github.com/farscapian/roygbiv-stack ~/git/tabconf2023` then checkout the `tabconf2023` branch.

# Future work

* Implement [lightningaddress for bolt12](https://github.com/rustyrussell/bolt12address) on the backend, allowing the front-end to dynamically fetch the BOLT12 product offers using names (e.g., product-a@domain.tld).
* Generate QR codes from connnection strings, provided as a PDF.
* Allow customer to submit custom branding for wallet and QR codes. 
* monitor the load of your various remotes so the front-end webapp can display product availability.
