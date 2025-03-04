# ns8-nextsec-controller

Setup and start an instance of [nextsecurity-controller](https://github.com/NethServer/nextsecurity-controller).

Each node can host multiple controller instances.

**Note**
This module is not yes present inside NS8 repository, so please install it manually following below instructions.

**Note**
This module implements the backup but not the restore procedure.

## Install

Instantiate the module with:

    add-module ghcr.io/nethserver/nextsec-controller:latest

The output of the command will return the instance name.
Output example:

    {"module_id": "nextsec-controller1", "image_name": "nextsec-controller", "image_url": "ghcr.io/nethserver/nextsec-controller:latest"}

## Configure

You can configure the controller directly from the UI or use the command line.

Let's assume that the nextsec-controller instance is named `nextsec-controller1`.

Launch `configure-module`, by setting the following parameters:
- `host`: a fully qualified domain name for the controller
- `lets_encrypt`: enable or disable Let's Encrypt certificate
- `ovpn_network`: OpenVPN network
- `ovpn_netmask`: OpenVPN netmasj
- `ovpn_cn`: OpenVPN Certificate CN
- `api_user`: controller admin user
- `api_password`: controller admin password

Example:

    api-cli run  module/nextsec-controller1/configure-module --data '{"host": "nscontroller.nethserver.org", "lets_encrypt": false, "ovpn_network": "172.19.64.0", "ovpn_netmask": "255.255.255.0", "ovpn_cn": "nextsec", "api_user": "admin", "api_password": "password"}'

The above command will:
- start and configure the nextsec-controller instance
- setup a route inside traefik to reach the controller
- setup a syslog receiver
- setup prometheus for metrics scraping

Send a test HTTP request to the nextsec-controller backend service:

    curl https://nscontroller.nethserver.org/

All logs are grouped the controller name (`ovpn_cn`). To query the logs, use:
```
logcli query '{controller_name="nextsec"}' --tail
```

## Module Overview

This multi-container module allows to connect the NS8 cluster to NextSecurity installations.

Here are all the module features:

### API Server

The [api server](https://github.com/NethServer/nextsecurity-controller/tree/master/api) gives NextSecurity the ability to register itself to NS8 (through [`ns-plug`](https://nethserver.github.io/nextsecurity/packages/ns-plug/)) and gives access to the on-demand generated credentials for the VPN.

The API even registers the endpoints for the [Traefik Proxy](#proxy-and-ui) that allows the interaction directly with the firewall even if it's not in the same network.

### VPN

The [OpenVPN container](https://github.com/NethServer/nextsecurity-controller/tree/master/vpn) tunnels connection from the NextSecurity to the NS8 through a VPN tunnel, due to [firewall configuration](https://github.com/NethServer/ns8-nextsec-controller/blob/main/imageroot/actions/configure-module/20configure#L87) in NS8, no client can be reached from other clients and only client-server communication is allowed.

### Proxy and UI

The [UI](https://github.com/NethServer/nextsecurity-controller/tree/master/ui) allows the browse of the interface directly off the NextSecurity installation, this is possible due to the [Traefik Proxy](https://github.com/NethServer/nextsecurity-controller/tree/master/proxy) server that redirects the urls to the correct IP inside the VPN.

### Promtail and Metrics

Using [`ns-plug`](https://nethserver.github.io/nextsecurity/packages/ns-plug/) the module automatically provides Prometheus and Loki endpoints so that NS8 can have all the data in the same place. You can browse the logs with [the command provided in configuration](#configure) while prometheus will most likely be already scraping off the NextSecurity data shortly after the first connection using [Service Discovery](https://github.com/NethServer/ns8-prometheus/#service-discovery).

## Uninstall

To uninstall the instance:

    remove-module --no-preserve nextsec-controller1
