# Formatting Guide

This page demonstrates the Markdown formatting features supported by TechDocs. Use it as a reference when writing documentation.

## Text Formatting

Regular text, **bold text**, *italic text*, and ***bold italic text***.

You can also use `inline code` for short code references like variable names or commands.

~~Strikethrough text~~ is supported too.

## Headings

Headings from `##` through `######` create sections with anchor links. You can link directly to any heading — for example, [jump to Tables](#tables) or [jump to Code Blocks](#code-blocks).

### This is H3

#### This is H4

##### This is H5

###### This is H6

## Lists

### Unordered Lists

- First item
- Second item
  - Nested item A
  - Nested item B
    - Deeply nested
- Third item

### Ordered Lists

1. First step
2. Second step
   1. Sub-step A
   2. Sub-step B
3. Third step

### Task Lists

- [x] Set up repository
- [x] Create mkdocs.yml
- [x] Write initial documentation
- [ ] Add diagrams
- [ ] Set up CI/CD for docs

## Links

**Internal links** (relative paths between doc pages):

- [Home page](../index.md)
- [Architecture Overview](../architecture/overview.md)
- [API Reference](api.md)

**External links:**

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Backstage TechDocs](https://backstage.io/docs/features/techdocs/)

**Anchor links** (to a heading on the current page):

- [Jump to Code Blocks](#code-blocks)
- [Jump to Tables](#tables)

## Tables

Tables are useful for reference documentation, comparisons, and structured data.

### Simple Table

| Name | Role | Team |
|------|------|------|
| Alice | SRE Lead | Platform |
| Bob | Developer | Orders |
| Carol | QA Engineer | Testing |

### Alignment

| Left-aligned | Center-aligned | Right-aligned |
|:-------------|:--------------:|--------------:|
| Left | Center | Right |
| Data | Data | Data |
| More | More | More |

## Code Blocks

### Inline Code

Use backticks for inline code: `oc get pods` or `kubectl apply -f manifest.yaml`.

### Fenced Code Blocks

Specify the language after the opening backticks for syntax highlighting.

**Bash:**

```bash
#!/bin/bash
echo "Hello from TechDocs"
oc get pods -n my-namespace --no-headers | wc -l
```

**Python:**

```python
def get_cluster_status(cluster_name: str) -> dict:
    """Fetch the current status of an OSD cluster."""
    response = ocm.get(f"/api/clusters_mgmt/v1/clusters/{cluster_name}")
    return response.json()
```

**YAML:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: my-service
data:
  LOG_LEVEL: "info"
  DB_HOST: "postgres.internal"
```

**Go:**

```go
func healthCheck(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(map[string]string{"status": "ok"})
}
```

**JSON:**

```json
{
  "name": "techdocs-test",
  "version": "1.0.0",
  "description": "Example TechDocs component"
}
```

## Blockquotes

> This is a blockquote. Use it to highlight important information or call out notes.

> **Note:** Blockquotes with bold prefixes are a common convention for callouts in Markdown.

> **Warning:** Be careful when running commands in production namespaces.

## Horizontal Rules

Use `---` to create a horizontal rule / section divider:

---

Content after the divider.

## Images

Images can be included with standard Markdown syntax. Place image files in the `docs/` directory:

```markdown
![Alt text](./images/diagram.png)
```

## Escaping Special Characters

Use backslashes to display literal Markdown characters:

- \*not italic\*
- \`not code\`
- \# not a heading

## Tips for Writing Good Documentation

1. **Start with the reader** — Who is this page for? What do they need to accomplish?
2. **Use descriptive headings** — Readers scan headings to find relevant sections
3. **Keep pages focused** — One topic per page, link to related pages
4. **Include examples** — Code samples, commands, and expected output
5. **Maintain cross-links** — Help readers navigate between related pages

## Related Pages

- [Configuration Reference](configuration.md) — mkdocs.yml options
- [Getting Started](../getting-started.md) — initial TechDocs setup
- [Home](../index.md) — documentation home page
