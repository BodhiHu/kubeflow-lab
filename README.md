# Kubeflow Helm Charts

Install [Kubeflow](https://www.kubeflow.org/) as [Helm](https://helm.sh/) charts.

## Install

1. `git clone git@github.com:BodhiHu/kubeflow-ops.git`
1. `helm install [-f your-value-overrides.yaml] kubeflow ./kubeflow-ops/helm-charts`

> for **The Middle Land** 中土 mirrors:

```bash
helm install -f ./kubeflow-ops/helm-charts/values-zhong.yaml kubeflow ./kubeflow-ops/helm-charts
```

Start istio port-forward gateway:

```bash
kubectl port-forward svc/istio-ingressgateway -n istio-system --address=0.0.0.0 8080:80
```

Now open `http://localhost:8080/`.

### Using a private registry:

If you're using private image registry:

```yaml
global:
  imageCredentials: ""
  useRegistryCredentials: false
  registry: your-registry.io
  username: your_name
  password: Pas8Wo!d
  email: your@email.com
minio:
  useKubeflowImagePullSecrets: true
```

## Uninstall

Run `helm delete kubeflow` to uninstall Kubeflow.

## Deploy Kubeflow in production

### Setup HTTPS

You ***Must*** use HTTPS to access Kubeflow web UI, except when you deploy it under `localhost`.

To setup HTTPS, configure `tlsCrt` and `tlsKey` to the HTTPS `.crt` and `.key` file content (base64 encoded).

### Setup Access To the UI

Kubeflow uses [Istio](https://istio.io/) as the service mesh control, you should configure you cluster load balancer or ingress to redirect to istio-ingressgateway:

- Through port-forward (***not recommended***):
  - HTTP: `kubectl port-forward svc/istio-ingressgateway -n istio-system --address=0.0.0.0 8080:80`, then browse `http://ip:8080/`.
  - HTTPS: `kubectl port-forward svc/istio-ingressgateway -n istio-system --address=0.0.0.0 443:443`, then browse `https://ip/`
- Through [NodePort](https://kubernetes.io/zh/docs/concepts/services-networking/service/#type-nodeport): first checkout if your `ingressgateway` service is running with NodePort: `kubectl -n istio-system get svc istio-ingressgateway`. Then access the NodePort to open Kubeflow web UI.
- Through Ingress: If Ingress is available in your cluster, then set `enableIngress: true` and
  `kubeflowHost` to your domain name in `values.yaml`, e.g. `kubeflowHost: "kubeflow.test.info"`.

The default username and password is: `user@example.com`, `12341234`

### Setup Auth/SSO with Dex (Optional)

If you need setup Kubeflow SingleSignOn/Auth with Dex deployment, you may need to setup below sections under `values.yaml`:

1. change `dex: enabled: false`
2. Setup below sections to connect to your Dex:

```
oidcAuthURL: /dex/auth
oidcProvider: http://dex.auth.svc.cluster.local:5556/dex
oidcRedirectURL: /login/oidc
skipAuthURI: "/dex"
useridClaim: email
useridHeader: kubeflow-userid
useridPrefix: "\"\""
oidcScopes: "profile email groups"

clientID: <your-dex-clientID>
clientSecret: <your-dex-client-secret>
```
