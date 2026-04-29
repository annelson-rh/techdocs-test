# Getting Started

This guide walks through setting up TechDocs for a new component in Red Hat Developer Hub.

## Prerequisites

- Access to Red Hat Developer Hub
- A Git repository for your component
- A registered component with TechDocs annotations

## How TechDocs Works

TechDocs follows a simple pipeline:

1. Add the `backstage.io/techdocs-ref` annotation to your `catalog-info.yaml`
2. Create a `mkdocs.yml` configuration file at the root of your repo
3. Add Markdown files in a `docs/` directory
4. Register the component in the RHDH Catalog
5. TechDocs builds and serves the documentation automatically

## Step-by-Step Setup

### 1. Create the catalog-info.yaml

Every Backstage component needs a `catalog-info.yaml`. Add the TechDocs annotation to enable documentation:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-service
  description: My awesome service
  annotations:
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  lifecycle: production
  owner: team-name
```

> **Note:** The `dir:.` value tells TechDocs to look for `mkdocs.yml` in the repository root.

### 2. Create mkdocs.yml

This file configures how MkDocs builds your documentation:

```yaml
site_name: My Service Docs
repo_url: https://github.com/org/my-service
edit_uri: edit/main/docs/
nav:
  - Home: index.md
  - Getting Started: getting-started.md
plugins:
  - techdocs-core
```

### 3. Add your docs

Create a `docs/` directory and add an `index.md`:

```markdown
# My Service

Welcome to the documentation for My Service.
```

### 4. Register in RHDH

Import your component through the RHDH catalog. Once registered, TechDocs will automatically discover and build the documentation.

## Directory Structure

A typical TechDocs-enabled repository looks like this:

```
my-service/
├── catalog-info.yaml       # Component registration + TechDocs annotation
├── mkdocs.yml              # MkDocs configuration
├── docs/
│   ├── index.md            # Documentation home page
│   ├── getting-started.md  # Additional pages
│   └── architecture/
│       └── overview.md     # Nested pages for sections
├── src/                    # Application source code
└── README.md               # Repo README (separate from TechDocs)
```

## What's Next?

- See the [Architecture Overview](architecture/overview.md) for an example of structured documentation
- Check the [Formatting Guide](reference/formatting.md) to learn about supported Markdown features
- Read the [MkDocs Navigation](reference/configuration.md) page for nav configuration options

---

*See also: [Backstage TechDocs documentation](https://backstage.io/docs/features/techdocs/)*
