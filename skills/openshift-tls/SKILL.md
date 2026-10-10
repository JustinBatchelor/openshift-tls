---
name: openshift-tls
description: Maintain or troubleshoot JustinBatchelor/openshift-tls cert-manager manifests for OpenShift ingress wildcard certificates using Let's Encrypt and Cloudflare DNS-01.
---

# OpenShift TLS

Work from the active `openshift-tls` checkout. This repository has three
cert-manager manifests; it does not install the operator, provision credentials,
or configure the ingress controller to consume a certificate.

## Resource map

- `clusterissuers.yaml`: separate staging and production ClusterIssuers, ACME
  account-key Secrets, and Cloudflare DNS-01 selection for `batchelor.live`.
- `certificate-staging.yaml`: staging Certificate and its separate TLS Secret
  in `openshift-ingress`.
- `certificate-production.yaml`: production Certificate targeting
  `ingress-wildcard-tls` in `openshift-ingress`.

Both Certificates cover the apps domain and its wildcard. Keep issuer references,
Certificate namespaces, and Secret names consistent when changing domains.
Staging and production must retain separate ACME account keys and TLS Secrets;
staging certificates are for issuance testing and are not browser-trusted.

## Credentials and validation

Cloudflare credentials are referenced as Secret `cloudflare`, key `token`.
Resolve the cert-manager controller's configured cluster-resource namespace
before locating or creating that Secret; do not assume that a ClusterIssuer's
credential belongs in `openshift-ingress`. Keep API tokens and private keys out
of YAML and Git. Inspect Secret names/key presence rather than printing values.

For manifest changes, review YAML and issuer/Secret/DNS-name consistency.
With authorized cluster access, verify cert-manager CRDs and use explicit
`oc apply --dry-run=server -f <file>` checks. No test runner or Helm chart is
provided. Dry-run acceptance alone does not establish successful ACME issuance.

For issuance troubleshooting, inspect Certificate, CertificateRequest, Order,
and Challenge conditions/events and the selected ClusterIssuer. Test DNS-01
through staging before requesting a new production issuance. Avoid repeated
production retries without resolving the reported cause.

## Ingress use

Before an authorized ingress-certificate switch, require a Ready production
Certificate and verify the issued certificate's SANs, validity, and chain without
exposing `tls.key`. Inspect the existing IngressController default-certificate
reference and retain its prior value for rollback. Applying the Certificate
manifest alone does not switch the router's default certificate. Do not delete
working TLS Secrets or ACME account keys as a routine troubleshooting step.
