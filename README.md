# Private Kubernetes OpenVPN Helm chart

TL;DR: This Chart is intended for deploying a private VPN server **without access to other Pods in the cluster**.
Think of it as roll-your-own Nord/Express VPN in your Kubernetes cluster.

## Usage

```bash
$ helm repo add k8s-ovpn https://raw.githubusercontent.com/suda/k8s-ovpn-chart/master
$ helm repo update
# To use UDP use `--set service.protocol=UDP` otherwise below defaults to TCP
$ helm install k8s-ovpn/k8s-ovpn-chart --set service.type=LoadBalancer \
    --set service.protocol=UDP
```

### Generate necessary secrets

```bash
$ git clone https://github.com/suda/k8s-ovpn-chart.git
$ cd k8s-ovpn-chart
$ export VPN_HOSTNAME=vpn.example.com
# If using UDP, make sure to set VPN_PROTOCOL=udp
$ export VPN_PROTOCOL=udp
$ export VPN_PORT=1194
# Generate basic OpenVPN config
$ ./bin/generate-config
# Repeat this step for all the clients you need
$ CLIENT_NAME=my-client ./bin/add-client
# Set the Kubernetes secrets. Prepend with REPLACE=true to update existing ones
$ REPLACE=true ./bin/set-secrets
```

After generating the secrets above, you'll have all the config, certificates **and the keys** on your machine, in the `ovpn0` directory. You need it to add more clients later but also isn't very secure to keep those keys just laying around on your machine.

### Allowing traffic inside of the cluster

If you want to make debugging the cluster easier and use VPN to access the pods, you might prefer using [`stable/openvpn` chart](https://github.com/helm/charts/tree/master/stable/openvpn).
But if you really want to, you can enable it by setting `limitTraficToNamespace` value to `false`.

## Configuration

The following table lists the configurable parameters of the `k8s-ovpn` chart and their default values.

| Parameter                | Description                                         | Default             |
| ------------------------ | --------------------------------------------------- | ------------------- |
| `image.repository`       | container image repository                          | `kylemanna/openvpn` |
| `image.tag`              | container image tag                                 | `2.3`               |
| `image.pullPolicy`       | container image pull policy                         | `IfNotPresent`      |
| `tolerations`            | node taints to tolerate (requires Kubernetes >=1.6) | `[]`                |
| `affinity`               | node/pod affinities (requires Kubernetes >=1.6)     | `{}`                |
| `nodeSelector`           | node labels for pod assignment                      | `{}`                |
| `resources`              | pod resource requests & limits                      | `{}`                |
| `limitTraficToNamespace` | limit network traffic just to OpenVPN namespace     | `true`              |
| `limitedCidr`            | CIDR to be blocked out                              | `10.0.0.0/8`        |

## Acknowledgements

This chart is based on [`chepurko/k8s-ovpn`](https://github.com/chepurko/k8s-ovpn) which is using the great [`kylemanna/docker-openvpn`](https://github.com/kylemanna/docker-openvpn) Docker image.
