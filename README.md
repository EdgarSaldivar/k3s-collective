# k3s-collective

Collection of all of my kubernetes resources created for my k3s cluster, hosted on a 2 nodes in my home office.

| Name           | CPU            | GPU        |
| -------------- | -------------- | ---------- |
| Minas-Tirith   | Ryzen 9 5950X  | RTX-2080   |
| Pelargir       | ARM Cortex-A72 | N/A        |

## FluxCD Installation

**Will need to create a GitHub Personal Access Token**

```sh
flux bootstrap github \
  --owner=EdgarSaldivar \
  --repository=k3s-collective \
  --branch=main \
  --path=clusters/k3s \
  --personal=true \
  --private=false #\
  # --reconcile # use if repository already exists
```

## Infrastructure

### Base

- [x] [Atuin Self Hosted](https://github.com/atuinsh/atuin)
- [x] [Traefik](https://artifacthub.io/packages/helm/traefik/traefik)
- [x] [Traefik Forward Auth](https://github.com/thomseddon/traefik-forward-auth)
- [x] [Cert Manager](https://github.com/cert-manager/cert-manager)
- [x] [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
- [x] [Reflector](https://github.com/emberstack/kubernetes-reflector)
- [x] [Reloader](https://github.com/stakater/Reloader)
- [x] [Keycloak](https://github.com/keycloak/keycloak)
- [x] [HomeAssistant](https://github.com/home-assistant)
- [x] [Nextcloud](https://github.com/nextcloud/server) + [Nextcloud Exporter](https://github.com/xperimental/nextcloud-exporter)
- [x] [Wiki.js](https://github.com/requarks/wiki)
- [x] [Pi-hole](https://github.com/MoJo2600/pihole-kubernetes)
- [x] [Cloudnative-PG](https://github.com/cloudnative-pg/cloudnative-pg)
- [x] [Error Pages](https://github.com/tarampampam/error-pages)

### Monitoring & Logging

- [x] [Prometheus](https://github.com/prometheus/prometheus)
- [x] [Grafana](https://github.com/grafana/grafana)
- [x] [Alert Manager](https://github.com/prometheus/alertmanager)
- [x] [Kube State Metrics](https://github.com/kubernetes/kube-state-metrics)
- [x] [Node Exporter](https://github.com/prometheus/node_exporter)
- [x] [Loki](https://github.com/grafana/loki)
- [x] [Alloy](https://github.com/grafana/alloy)

### Security

- [x] [CrowdSec](https://github.com/crowdsecurity/crowdsec)

### Games

- [x] [Valheim](https://artifacthub.io/packages/helm/geek-cookbook/valheim)
- [x] [Minecraft](https://artifacthub.io/packages/helm/minecraft-server-charts/minecraft)
- [x] [V-Rising](https://truecharts.org/charts/stable/v-rising/)

### Media 

*In Installation Order*

- [x] [nvidia-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)
- [x] [Plex](https://github.com/plexinc/pms-docker/blob/master/charts/plex-media-server/README.md)
- [x] [Deluge](https://github.com/binhex/arch-delugevpn)
- [x] [Radarr](https://github.com/Radarr/Radarr)
- [x] [Sonarr](https://github.com/Sonarr/Sonarr)
- [x] [Lidarr](https://github.com/Lidarr/Lidarr)
- [x] [Bazarr](https://github.com/morpheus65535/bazarr)
- [x] [Prowlarr](https://github.com/Prowlarr/Prowlarr)
- [x] [Recyclarr](https://github.com/recyclarr/recyclarr)
- [x] [Overseerr](https://github.com/sct/overseerr)
- [x] [Tautulli](https://github.com/Tautulli/Tautulli)

### TBD

- [ ] [Scrutiny](https://github.com/AnalogJ/scrutiny) - **find alternative** as this is not compatible with k8s, [helm chart current under development](https://github.com/AnalogJ/scrutiny/pull/363)
- [ ] [Gotify](https://github.com/gotify/server) - **find alternative** as is not possible with iOS
- [ ] [Uptime Kuma](https://github.com/louislam/uptime-kuma) - On the fence, kinda like the idea tho
- [ ] [Readarr](https://github.com/Readarr/Readarr)
- [ ] [Kavita](https://github.com/Kareadita/Kavita)
- [ ] [Jellyfin](https://github.com/jellyfin/jellyfin)

## Secrets Management

1. Register Helm Repo

```sh
flux create source helm sealed-secrets \
  --interval=1h \
  --url=https://bitnami-labs.github.io/sealed-secrets
```

2. Create HelmRelease to install Sealed-Secrets Controller

```sh
flux create helmrelease sealed-secrets \
  --interval=1h \
  --release-name=sealed-secrets-controller \
  --target-namespace=flux-system \
  --source=HelmRepository/sealed-secrets \
  --chart=sealed-secrets \
  --chart-version=">=2.8.0 <3.0.0" \
  --crds=CreateReplace
```

3. Retrieve the public key:

```sh
kubeseal --fetch-cert \
  --controller-name=sealed-secrets-controller \
  --controller-namespace=flux-system \
  > pub-sealed-secrets.pem
```

4. Create a secret

```sh
kubectl -n default create secret generic basic-auth \
  --from-literal=user=admin \
  --from-literal=password=change-me \
  --dry-run=client \
  -o yaml > basic-auth.yaml
```

5. Seal the Secret

```sh
kubeseal --format=yaml --cert=pub-sealed-secrets.pem \
  < basic-auth.yaml > basic-auth-sealed.yaml
```

6. Apply the Sealed Secret

```sh
kubectl apply -f basic-auth-sealed.yaml
```

## References

Heavily inspired by the degen https://github.com/BriianPowell
