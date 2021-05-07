# Generate Your Client-Facing and AKS Ingress Controller TLS Certificates

Now that you have the [prerequisites](./01-prerequisites.md) met, follow the steps below to create the TLS certificates that Azure Application Gateway will serve for clients connecting to your web app as well as the AKS Ingress Controller. If you already have access to an appropriate certificates, or can procure them from your organization, consider doing so and skipping the certificate generation steps. The following will describe using a self-signed certs for instructive purposes only.

## Steps

1. Generate the wildcard certificate for the AKS Ingress Controller

   > :book: We will also procure another TLS certificate, a standard cert, to be used with the AKS Ingress Controller. This one is not EV, as it will not be user facing. Finally the app team decides to use a wildcard certificate of `*.aks-ingress.aks-sb.com` for the ingress controller.

   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out traefik-ingress-internal-aks-ingress-aks-sb-com-tls.crt -keyout traefik-ingress-internal-aks-ingress-aks-sb-com-tls.key -subj "/CN=*.aks-ingress.aks-sb.com/O=Secure Baseline Aks Ingress"
   ```

2. Base64 encode the AKS Ingress Controller certificate

   :bulb: No matter if you used a certificate from your organization or you generated one from above, you'll need the public certificate (as `.crt` or `.cer`) to be base 64 encoded for proper storage in Key Vault later.

   ```bash
   export AKS_INGRESS_CONTROLLER_CERTIFICATE_BASE64=$(cat traefik-ingress-internal-aks-ingress-aks-sb-com-tls.crt | base64 | tr -d '\n')
   ```

### Next step

:arrow_forward: [Prep for Azure Active Directory integration](./03-aad.md)
