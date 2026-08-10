# Data Formulator Compliance Remediation

**Date**: 2026-08-10
**Branch**: `remediate/data-formulator-compliance`
**Base**: `backup/pre-reconcile-2026-07-22` (`fc297e77`), the production-hardened branch

## Scope

Bring the deployed Data Formulator workload toward compliance without deploying
the unsafe MCP revision or changing the Service360 repository.

## Evidence

- Baseline Container App image: `azd-deploy-1784045589`
  (`sha256:34755ba63b62236cf2bb023a00c9b9cae6a89acf361fff5cef8041c17cbbf482`).
- Baseline image findings: `linux`, `aiohttp`, `wheel`, `jaraco.context`,
  `setuptools`, and `pip`.
- Current active revision: `ca-dataformulator--compliance01` at 100% traffic,
  using candidate digest
  `sha256:47476614c89d98f1d14c7e21831770a7590754211e6982b0d85bba8dcf02ea6b`.
- Non-image Defender findings: ACR unrestricted network access, ACR private
  link, policy-created storage private link, and Azure Firewall for the VNet.
- S360 reports one NonProd public IP without a service or virtual tag. Data
  Formulator does not rely on that outbound IP for privileged ACL, UDR, NSG, or
  firewall trust, so the official route is a Virtual Tag request.

## Execution Ledger

- [x] Create an isolated branch from the production-hardened base.
- [x] Add scanner-required package floors to the Docker build and Python
  dependency surfaces.
- [x] Resolve dependencies against the configured corporate package proxy;
  require `aiohttp>=3.14.3` to coexist with `litellm==1.91.1`.
- [x] Run Python tests and frontend build/tests from the isolated branch:
  2,188 backend tests passed, 13 skipped; 277 frontend tests passed; production
  frontend build succeeded.
- [x] Build candidate image `compliance-20260810-01` in ACR on a fresh remote
  agent. Run `chg` succeeded; immutable digest is
  `sha256:47476614c89d98f1d14c7e21831770a7590754211e6982b0d85bba8dcf02ea6b`.
- [x] Create zero-traffic revision `ca-dataformulator--compliance01` from the
  candidate digest. Revision is healthy with one replica; rollback revision
  `ca-dataformulator--0000002` remains healthy at 100% traffic.
- [x] Smoke-test the candidate revision directly: root returned HTTP 200;
  authentication behavior matches the current revision; 13 loaders and one
  connected built-in connector were discovered; all three configured Azure
  models passed bounded connectivity pings; candidate logs contain no startup
  errors.
- [x] Run a 10% canary: 40/40 public requests returned HTTP 200, both revisions
  remained healthy, and all three Azure models connected through public ingress.
- [x] Shift traffic to `ca-dataformulator--compliance01` at 100%; 25/25
  post-cutover requests returned HTTP 200 and all three Azure models connected.
  Keep `ca-dataformulator--0000002` active at 0% for rollback.
- [x] Submit the `/NonProd` Virtual Tag request for platform-managed outbound
  public IP `72.153.117.98`; Fabio confirmed form submission on 2026-08-10.
- [ ] Re-query Defender after assessment records appear for the new digest and
  S360 after a newer publisher ingestion processes the Virtual Tag request.

**Current gate**: Wait for Defender to scan candidate digest
`sha256:47476614c89d98f1d14c7e21831770a7590754211e6982b0d85bba8dcf02ea6b`
and for the S360 mitigation publisher to process the Virtual Tag request before
re-querying both controls. The new digest had no Defender assessment records at
the immediate post-deployment check, so its state is scan pending, not confirmed
clean. The immediate post-submission S360 query still returned the NonProd item
active with no resolved-history record; its source ingestion timestamp remained
`2026-08-10T19:22:55.1524752Z`, predating the form submission. Do not resubmit.

**Traffic safeguard**: revision copy inherited a `latestRevision = 100%`
traffic selector. It was immediately replaced with explicit revision weights
before smoke testing. All subsequent checks used named revision weights.

## Separate Approval Gates

These controls are not bundled into the active-image patch:

- Upgrade ACR from Basic to Premium, add a private endpoint and private DNS, and
  disable public network access. This creates recurring cost and changes the
  image-pull path.
- Determine whether the policy-created `ailogspjvf4mtttada2` storage account can
  safely receive a private endpoint without breaking its external producer.
- Delete superseded vulnerable ACR manifests only after the candidate revision
  is proven and rollback retention is agreed.
- Azure Firewall for this isolated DEV VNet is a low-severity, high-cost
  architecture decision; assess applicability before accepting or implementing
  it.

## Rollback

Keep `ca-dataformulator--0000002` and image `azd-deploy-1784045589` intact until
Defender has assessed the new digest and rollback retention is explicitly
closed. If runtime validation regresses, return traffic to `--0000002`; do not
deploy the dormant MCP image `mcp-20260722-1804`.
