# Cloud vs AMS Dispatcher Guide

This guide summarizes the main differences between AEMaaCS dispatcher work and AEM 6.5 / AMS dispatcher work.

## Purpose

Use this guide when:

- a user works across both variants
- a recommendation needs cross-variant comparison
- a migration or adaptation question comes up

## High-Level Difference

- AEMaaCS dispatcher work is shaped by cloud-managed defaults, validator-enforced topology, reserved probe paths, and CDN boundaries.
- AMS dispatcher work is shaped by multi-tier farm routing, AMS variables, flush ACL safety, and mutable-versus-immutable include practices in AMS layouts.

## Comparison Table

| Area | AEMaaCS Cloud | AEM 6.5 / AMS |
|---|---|---|
| Front-door architecture | CDN is part of the decision boundary | No cloud CDN baseline assumption |
| Repo root convention | typically `dispatcher/src` | commonly `src` or `dispatcher/src` |
| Guardrail emphasis | immutable defaults, required includes, symlink topology, probe safety | tier ordering, AMS variables, flush ACLs, author/publish separation |
| Runtime probes | `/systemready` and `/system/probes/*` are reserved and must remain safe | no equivalent cloud probe-path contract |
| Validator sensitivity | strong topology and alias expectations | less cloud-topology-specific, more AMS tier and variable concerns |
| Routing model | cloud vhost safety behavior and alias coverage matter | tier/farm ordering and host routing matter |
| Performance boundary | CDN vs Dispatcher ownership must be stated | performance work is mostly Dispatcher and backend-adjacent |
| Security boundary | separate Dispatcher findings from CDN/WAF findings | focus on Dispatcher/HTTPD and AMS topology controls |

## Migration Checklist

When adapting a pattern from AMS to cloud:

1. Re-check whether the concern belongs at the CDN layer instead of Dispatcher.
2. Preserve cloud-managed defaults and wrapper includes.
3. Verify required aliases, symlink topology, and reserved probe paths.
4. Re-validate host, cache, and rewrite assumptions under cloud guardrails.

When adapting a pattern from cloud to AMS:

1. Re-check author/publish/tier-specific farm ordering.
2. Re-map cloud assumptions to AMS variables and tier renders.
3. Re-check flush and invalidation ACL safety.
4. Validate that the recommendation still fits mutable AMS include points.

## Anti-Patterns

Do not assume:

- a cloud rewrite or host pattern can be copied directly into AMS
- an AMS tier or variable pattern can be copied directly into cloud layouts
- CDN/WAF decisions belong in Dispatcher config by default
- cloud validator expectations apply unchanged to AMS

## Recommended Workflow

For cross-variant questions:

1. Start with the variant-specific guardrail file.
2. Use the mode-specific verification matrix for both variants.
3. Produce a side-by-side delta plan.
4. Call out what must be re-validated after translation.
