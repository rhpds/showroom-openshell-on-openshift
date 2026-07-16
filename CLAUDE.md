# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

An RHDP (Red Hat Demo Platform) Showroom lab built with Antora. The lab content covers **NVIDIA OpenShell and OpenClaw on Red Hat OpenShift** — installing the Agent Sandbox controller and OpenShell gateway, integrating the OpenClaw AI agent application, and configuring security policy enforcement for supervised sandbox pods.

The repo was created from the [showroom_template_nookbag](https://github.com/rhpds/showroom_template_nookbag) template. The upstream template also serves as the Showroom platform documentation.

### Source Lab

The original content lives in `../openshell-on-openshift-lab-main/` — a plain GFM Markdown + Kubernetes manifests repo with 5 lab modules (Levels 0–4) covering: lab overview, OpenShell installation, OpenClaw integration, policy enforcement, and policy profiles. The port converts these to Showroom AsciiDoc format.

## Build and Preview

No `package.json` or `Makefile` exists. The build uses Antora with extensions.

```bash
# Recommended: container-based preview (hot-reload on file changes)
podman run --rm --name antora -v $PWD:/antora -p 8080:8080 -i -t ghcr.io/juliaaano/antora-viewer
# SELinux: append :z to volume mount

# Manual: install extensions then build
npm i -g @antora/cli@3.1 @antora/site-generator@3.1 @sntke/antora-mermaid-extension @andrew-jones/antora-tabs-extension
antora --fetch site.yml
# Output lands in ./www/
```

CI builds via `.github/workflows/gh-pages.yml` on push/PR to `main` when `content/**`, `site.yml`, or the workflow file changes. The CI adds `@antora/lunr-extension` for search (not in `site.yml`).

## Key Files

| File | Purpose |
|------|---------|
| `site.yml` | Antora playbook — **do not modify** (managed by template) |
| `content/antora.yml` | Component descriptor — title, nav, AsciiDoc attributes, variable defaults |
| `content/modules/ROOT/nav.adoc` | Sidebar navigation |
| `content/modules/ROOT/pages/*.adoc` | Lab content pages |
| `content/modules/ROOT/assets/images/` | Image assets (create if needed) |
| `content/modules/ROOT/partials/` | Reusable AsciiDoc includes (create if needed) |
| `ui-config.yml` | Showroom right-pane tabs — terminal config, OCP console, external links |

## AsciiDoc Conventions for Showroom

### Execute Blocks (copy-to-terminal button)
```asciidoc
[source,bash,role="execute"]
----
oc get pods
----
```
Variants: `role="execute-top"` / `role="execute-bottom"` for split terminals.

### Variable Substitution
Define defaults in `content/antora.yml` under `asciidoc.attributes`. At deploy time, Showroom replaces these with real runtime values from `user_data`. Use in source blocks with `subs="attributes"`:
```asciidoc
[source,bash,role="execute",subs="attributes"]
----
ssh {bastion_ssh_user_name}@{bastion_public_hostname}
----
```

### Images
```asciidoc
image::screenshot.png[alt text,link=self,width=700]
```
Use `link=self` for lightbox popout. Do **not** use `window=blank`.

### Admonitions
```asciidoc
WARNING: Do not run this on a production cluster.
```
No blank line after `====` admonition delimiter (breaks attribute substitution).

### Tabs
```asciidoc
[tabs]
====
Linux::
+
--
[source,bash,role="execute"]
----
curl -LsSf https://example.com/install.sh | sh
----
--
macOS::
+
--
[source,bash,role="execute"]
----
brew install example
----
--
====
```

### Mermaid Diagrams
```asciidoc
[mermaid]
....
flowchart LR
    A --> B --> C
....
```
For split-panel readability, consider pre-rendering as SVG instead.

## Markdown-to-AsciiDoc Porting Patterns

When converting the source lab from GFM Markdown:

| Markdown | AsciiDoc |
|----------|----------|
| `# Title` | `= Title` |
| `## Section` | `== Section` |
| `**bold**` | `*bold*` |
| `` `inline` `` | `` `inline` `` (same) |
| ` ```bash ` | `[source,bash,role="execute"]` + `----` delimiters |
| `> **Warning:** text` | `WARNING: text` |
| `[text](url)` | `https://url[text]` or `link:url[text]` |
| `[text](other.md)` | `xref:other.adoc[text]` |
| `- item` | `* item` |
| `1. item` | `. item` |
| `| table |` | AsciiDoc table with `[cols="...",options="header"]` |
| ` ```mermaid ` | `[mermaid]` + `....` delimiters |

### Shell Variable Handling

The source lab uses shell variables (`${OPENSHELL_NAMESPACE}`, `${OPENCLAW_NAMESPACE}`) set in Level 0 and reused throughout. Two options:
1. **Keep as shell variables** — user exports them in the terminal; source blocks remain plain `role="execute"` without `subs="attributes"`
2. **Convert to Antora attributes** — define in `antora.yml`, use `{openshell_namespace}` with `subs="attributes"` on source blocks

Prefer option 1 for variables the user sets dynamically; use option 2 only for values known at deploy time (guid, hostnames, passwords).

## ui-config.yml

Controls the Showroom split-pane right panel. Supports:
- Terminal tabs (`path: /wetty`, `port: 443`)
- OCP Console (`url: 'https://console-openshift-console.${DOMAIN}'`)
- Split terminals (`secondary_name`, `secondary_path`, `secondary_port`)
- External links (`url: 'https://...'`)

URL variables `${DOMAIN}` and `${GUID}` are substituted at deploy time.

## Content Structure for This Lab

The navigation should follow the source lab's 5-level progression:
- Level 0: Overview (concepts, architecture, CLI install)
- Level 1: OpenShell on OpenShift (Agent Sandbox + gateway install)
- Level 2: OpenClaw integration (deploy OpenClaw, delegated execution)
- Level 3: Policy enforcement (filesystem, network, process policies)
- Level 4: Policy profiles (deny-all, managed, allowlist comparison)
- Troubleshooting (shared reference page)

Each module follows a consistent structure: goal/time estimate, concepts, starting-state checklist, exercises, verification, troubleshooting, cleanup, optional challenge.
