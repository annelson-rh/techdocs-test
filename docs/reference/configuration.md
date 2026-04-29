# Configuration Reference

This page documents the `mkdocs.yml` configuration options relevant to TechDocs.

## Minimal Configuration

The simplest working `mkdocs.yml`:

```yaml
site_name: My Service Docs
plugins:
  - techdocs-core
```

With just this, TechDocs will auto-discover all Markdown files in `docs/`.

## Full Configuration Example

```yaml
site_name: My Service Docs
site_description: Documentation for My Service
repo_url: https://github.com/org/my-service
edit_uri: edit/main/docs/

nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - Architecture:
      - Overview: architecture/overview.md
      - Deployment: architecture/deployment.md
  - Runbooks:
      - Incident Response: runbooks/incident-response.md
      - Troubleshooting: runbooks/troubleshooting.md
  - Reference:
      - API: reference/api.md
      - Configuration: reference/configuration.md
      - Formatting Guide: reference/formatting.md

plugins:
  - techdocs-core
```

## Configuration Options

### site_name

**Required.** The title shown in the TechDocs header.

```yaml
site_name: Order Service Documentation
```

### repo_url

Link to the source repository. Enables the "View source" link in the TechDocs UI.

```yaml
repo_url: https://github.com/org/my-service
```

### edit_uri

Path appended to `repo_url` to generate "Edit this page" links. Set relative to the repo root.

```yaml
edit_uri: edit/main/docs/
```

> **Tip:** The `edit/main/` prefix opens GitHub's web editor on the `main` branch. Adjust if your default branch has a different name.

### nav

Defines the navigation structure. Without `nav`, MkDocs auto-generates navigation from the file structure.

**Flat navigation:**

```yaml
nav:
  - Home: index.md
  - About: about.md
```

**Nested sections:**

```yaml
nav:
  - Home: index.md
  - Architecture:
      - Overview: architecture/overview.md
      - Deployment: architecture/deployment.md
```

**Rules:**

- Paths are relative to the `docs/` directory
- Section labels (like "Architecture") create collapsible groups
- The first item in `nav` is used as the landing page

### plugins

For TechDocs, you must include `techdocs-core`:

```yaml
plugins:
  - techdocs-core
```

> **Warning:** Do not add other MkDocs plugins unless they are supported by TechDocs. Unsupported plugins may cause build failures.

## catalog-info.yaml Annotations

The TechDocs annotation in `catalog-info.yaml` tells RHDH where to find the docs source:

| Annotation | Value | Description |
|-----------|-------|-------------|
| `backstage.io/techdocs-ref` | `dir:.` | Docs are in the same repo (most common) |
| `backstage.io/techdocs-ref` | `url:https://...` | Docs are in an external repo |

## Related Pages

- [Getting Started](../getting-started.md) — step-by-step setup guide
- [Formatting Guide](formatting.md) — what Markdown features work in TechDocs
