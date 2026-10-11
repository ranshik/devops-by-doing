
> This file is the **single source of truth** for the content rendered by `index.html`. Follow this structure exactly and the page will pick up everything automatically — no code changes required.

---

## 📐 Architecture Overview

`index.html` parses `main.md` using this logic:

1. **Splits on `## ` headings** → each `##` section becomes one tool card
2. **Reads metadata from the first paragraph** after the title — looks for `**Difficulty:**` and `**Time:**`
3. **Picks an icon + color** from the title keywords (see [Icon Mapping](#-icon-mapping--colors))
4. **Extracts `### Overview`** → card description
5. **Extracts `### Why Learn This`** → not displayed on card (used in detail view)
6. **Extracts all `#### Step N:` blocks** → renders with numbering in detail view
7. **Extracts `### Practice Project`** → numbered list items become checkable tasks
8. **Converts markdown to HTML** with full code-block + copy-button support

---

## 🔤 Naming Rules

| Element | Format | Example |
|---|---|---|
| Tool title | `## 🐧 Tool Name` | `## 🐳 Docker` |
| Step | `#### Step 1: Title Here` | `#### Step 3: Write Your First Dockerfile` |
| Section | `### Section Name` | `### Why Learn This` |
| Metadata | `**Key:** value` (inline) | `**Difficulty:** Beginner \| **Time:** 6-8 hours` |

✅ **Do:**
- Start every tool with `## ` followed by an emoji + space + name
- Use the EXACT `### Overview`, `### Why Learn This`, `### Practice Project` section names (parser looks for them)
- Use `#### Step N: Title` for steps (parser counts them, renders step badges)
- Put the `**Difficulty:**` and `**Time:**` line in the **second line** of the section (right after the title)
- Use fenced code blocks with language: ` ```bash `, ` ```yaml `, ` ```dockerfile `, etc.
- Use `-` or `*` for unordered lists, `1.` for ordered lists

❌ **Don't:**
- Don't put `## ` headings inside code blocks
- Don't use `### Practice Project` for any other purpose
- Don't skip the emoji at the start of tool titles (it sets the visual identity)
- Don't nest tool sections — one `## ` = one tool

---

## 🎨 Icon Mapping & Colors

Icons and colors are auto-assigned based on keywords in the title. Override by adding an explicit icon in the title (see below).

| Keyword in title | Icon | Color |
|---|---|---|
| `linux`, `bash`, `shell` | `terminal` | Green |
| `git`, `github` | `git-branch` | Orange |
| `python` | `code` | Orange |
| `docker`, `container`, `podman` | `container` | Cyan |
| `kubernetes`, `k8s`, `helm` | `box` | Cyan |
| `jenkins` | `git-branch` | Orange |
| `gitlab`, `ci/cd`, `actions` | `git-merge` | Orange |
| `aws`, `azure`, `gcp`, `cloud` | `cloud` | Yellow |
| `terraform`, `pulumi`, `iac` | `layers` | Orange |
| `ansible`, `chef`, `puppet` | `settings` | Orange |
| `prometheus`, `grafana`, `monitoring` | `bar-chart-3` | Yellow |
| `vault`, `secrets` | `lock` | Yellow |
| `nginx`, `haproxy`, `load balancer` | `network` | Cyan |
| `redis`, `memcached`, `cache` | `database` | Green |
| `postgres`, `mysql`, `mongodb`, `database` | `database` | Green |
| `security`, `devsecops`, `trivy` | `shield` | Red |
| `nginx`, `apache`, `web server` | `globe` | Green |
| `react`, `frontend`, `vue` | `layout` | Cyan |
| default | `box` | Orange |

**Override icon** in the title by writing `## 🐳 Docker [icon=package]` — the `[icon=...]` suffix is stripped from the displayed title and used as the icon. See [Available Icons](#-available-icons) below.

---

## 📋 Complete Tool Template

Copy this block, replace the placeholders, and paste it into `main.md` between existing tools (or at the end):

```markdown
## 🔧 TOOL_NAME_HERE

**Difficulty:** Beginner | **Time:** 6-8 hours

### Overview
Write 2-4 sentences explaining what this tool/technology is. This appears on the tool card. Keep it concise — under 200 words. The first sentence is used as the card description preview.

### Why Learn This
- Bullet point 1: A concrete reason (industry demand, foundation for X, etc.)
- Bullet point 2: Another reason
- Bullet point 3: Career impact or practical use case
- Bullet point 4: Connection to other tools in the path

### Learn by Doing — Step by Step

#### Step 1: Set Up Your Environment
1. **Action item** — sub-step with a bolded action and explanation
2. **Install the tool:**
   ```bash
   # Multi-line commands work
   sudo apt install package-name
   systemctl enable package-name
   ```
3. **Verify it works:**
   ```bash
   tool-name --version
   ```
4. **Troubleshooting tip:** If you see error X, run Y to fix.

#### Step 2: Core Concepts
1. **Concept 1** — explanation paragraph
   ```bash
   # Example command
   tool-name run --option value
   ```
2. **Concept 2** — another explanation
3. **Hands-on exercise:** Try modifying the example and see what happens

#### Step 3: Real-World Usage
1. **Build something practical:**
   ```yaml
   # Configuration example
   setting: value
   another_setting: 42
   ```
2. **Apply the configuration:**
   ```bash
   tool-name apply config.yaml
   ```
3. **Inspect the result:**
   ```bash
   tool-name get status
   ```

#### Step 4: Advanced Features
1. Feature A with example
2. Feature B with example
3. Performance tip: use `--cache` for repeated runs

#### Step 5: Production Best Practices
- **Security:** Always rotate credentials, use least-privilege access
- **Monitoring:** Set up health checks and alerts
- **Scaling:** Use horizontal scaling, not vertical
- **Backup:** Snapshot state before upgrades

### Practice Project
- Set up a complete environment from scratch
- Build a working example that demonstrates 3+ concepts
- Write automation scripts to test your setup
- Document the architecture in a README
- Deploy to a cloud free tier and verify it works
- Set up basic monitoring and logging
- Add CI/CD pipeline that auto-deploys on git push
- Write troubleshooting runbook for common failures

### Key Takeaways
- Bullet summarizing the most important concept
- Another takeaway
- Connection to next steps in the learning path
- One specific skill you can put on a resume
```

---

## 🧩 Field Reference

### Required Fields (in every tool section)

| Field | Location | Format | Purpose |
|---|---|---|---|
| Title | First line of section | `## 🎯 Tool Name` | Card title + sets icon/color |
| Difficulty | Line 2 of section | `**Difficulty:** Beginner\|Intermediate\|Advanced` | Card tag, filter |
| Time | Line 2 of section | `**Time:** 6-8 hours` | Card meta, total time calc |
| Overview | First `###` section | Multi-line text | Card description |
| Practice Project | Last `###` section before `---` | Bulleted list | Checkable tasks (drives progress) |

### Optional Fields

| Field | Format | Purpose |
|---|---|---|
| Why Learn This | `### Why Learn This` + bullets | Motivation shown in detail view |
| Steps | `#### Step N: Title` blocks | Main learning content |
| Key Takeaways | `### Key Takeaways` + bullets | Summary shown in detail view |
| Override icon | `## 🐳 Docker [icon=package]` | Custom icon |
| Override color | `## 🐳 Docker [color=purple]` | Custom color (theme name) |

### Supported Colors for Override
`orange` (default), `green`, `cyan`, `yellow`, `red`, `purple`, `pink`, `blue`

---

## 🛠️ Available Icons

All icons come from [Lucide](https://lucide.dev/icons). Use these in `[icon=...]` overrides:

`terminal`, `code`, `code-2`, `git-branch`, `git-merge`, `git-commit`, `container`, `box`, `boxes`, `package`, `layers`, `cloud`, `cloud-upload`, `cloud-download`, `database`, `server`, `globe`, `network`, `wifi`, `settings`, `sliders`, `cog`, `wrench`, `hammer`, `shield`, `shield-check`, `lock`, `key`, `unlock`, `eye`, `activity`, `pulse`, `bar-chart`, `bar-chart-3`, `line-chart`, `pie-chart`, `trending-up`, `gauge`, `zap`, `rocket`, `flame`, `trophy`, `target`, `compass`, `map`, `navigation`, `layout`, `layout-grid`, `layout-template`, `monitor`, `smartphone`, `tablet`, `cpu`, `hard-drive`, `memory-stick`, `binary`, `hash`, `file-code`, `file-text`, `folder`, `folder-open`, `archive`, `play`, `play-circle`, `pause`, `square`, `circle`, `triangle`, `hexagon`, `octagon`, `star`, `heart`, `bookmark`, `tag`, `flag`, `bell`, `mail`, `message-circle`, `send`, `share`, `link`, `paperclip`, `pencil`, `edit`, `trash`, `trash-2`, `save`, `download`, `upload`, `refresh-cw`, `rotate-cw`, `search`, `zoom-in`, `zoom-out`, `filter`, `arrow-right`, `arrow-left`, `arrow-up`, `arrow-down`, `chevron-right`, `chevron-left`, `chevron-up`, `chevron-down`, `plus`, `minus`, `x`, `check`, `check-circle`, `check-circle-2`, `alert-circle`, `alert-triangle`, `info`, `help-circle`, `loader`, `loader-2`, `clock`, `calendar`, `timer`, `hourglass`, `sun`, `moon`, `cloud-rain`, `cloud-snow`, `wind`, `droplet`, `flame`, `leaf`, `tree-pine`, `mountain`, `waves`

---

## 📚 Content Patterns That Render Beautifully

### Code Blocks
Always specify a language for syntax highlighting + the copy button label:

````markdown
```bash
# Comments work
sudo apt install nginx
systemctl start nginx
```

```yaml
# YAML configs
apiVersion: v1
kind: Service
metadata:
  name: my-service
```

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "index.js"]
```

```python
def hello(name):
    return f"Hello, {name}!"

print(hello("DevOps"))
```

```sql
SELECT * FROM users WHERE active = true;
```

```json
{
  "name": "myapp",
  "version": "1.0.0"
}
```
````

**Supported languages:** `bash`, `sh`, `shell`, `zsh`, `powershell`, `yaml`, `yml`, `json`, `toml`, `xml`, `html`, `css`, `javascript`, `js`, `typescript`, `ts`, `python`, `ruby`, `go`, `rust`, `java`, `kotlin`, `swift`, `c`, `cpp`, `csharp`, `php`, `sql`, `graphql`, `markdown`, `dockerfile`, `docker`, `makefile`, `nginx`, `apache`, `terraform`, `hcl`, `hcl2`, `groovy`, `ini`, `env`, `diff`, `text`

### Tables
```markdown
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Data A   | Data B   | Data C   |
| Data D   | Data E   | Data F   |
```

### Callouts
```markdown
> **Pro tip:** Use `Ctrl+R` to reverse search your command history.
> **Warning:** Never run `rm -rf /` — it will delete your entire filesystem.
> **Note:** This step requires sudo access.
```

### Inline Code
Use backticks for `commands`, `filenames`, `paths`, and `variable_names`. They render with monospace + accent color.

### Ordered vs Unordered Lists
- Use `-` for unordered (default)
- Use `1.` for ordered steps
- Use indented sub-lists (2 spaces) for nested items

### Images & Diagrams (optional)
```markdown
![Architecture diagram](./images/my-diagram.png)
```

Place image files in an `images/` subfolder. Keep file sizes < 200KB for fast loading.

### Links
```markdown
[Link text](https://example.com)
[Link with title](https://example.com "Hover title")
```

External links get a small underline. Make sure they're real and safe.

---

## 🧪 Example: Minimal Valid Tool

The smallest valid tool that will render correctly:

```markdown
## 🦀 Rust

**Difficulty:** Intermediate | **Time:** 10 hours

### Overview
Rust is a systems programming language focused on safety, speed, and concurrency. It prevents memory errors at compile time without garbage collection.

### Why Learn This
- Memory safety without garbage collection
- Performance comparable to C/C++
- Growing adoption in systems tooling (used by Cloudflare, Discord, AWS)

### Learn by Doing — Step by Step

#### Step 1: Install Rust
1. Install via rustup:
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   source $HOME/.cargo/env
   ```
2. Verify:
   ```bash
   rustc --version
   cargo --version
   ```

#### Step 2: First Program
1. Create a new project:
   ```bash
   cargo new hello_rust
   cd hello_rust
   ```
2. Run it:
   ```bash
   cargo run
   ```
3. Edit `src/main.rs` and experiment.

### Practice Project
- Build a CLI tool that processes CSV files
- Add unit tests for all functions
- Publish to crates.io
- Write documentation with examples

### Key Takeaways
- Rust's ownership model prevents entire classes of bugs
- Cargo is a best-in-class package manager
- Great for CLI tools, web services, and embedded systems
```

---

## 📐 Example: Full-Featured Tool

This shows every available feature in use:

```markdown
## ☁️ AWS Lambda [icon=zap]

**Difficulty:** Intermediate | **Time:** 4-6 hours

### Overview
AWS Lambda runs your code in response to events without provisioning servers. Pay only for compute time consumed — ideal for APIs, data processing, and automation.

### Why Learn This
- Foundation of serverless architecture
- Auto-scales from 0 to thousands of requests
- Reduces ops burden — no servers to manage
- Integrates with 200+ AWS services via event triggers
- Industry-standard skill for cloud engineers

### Learn by Doing — Step by Step

#### Step 1: Create Your First Function
1. **Open the Lambda console** and click "Create function"
2. **Choose "Author from scratch":**
   - Name: `my-first-function`
   - Runtime: `Node.js 20.x`
   - Architecture: `x86_64`
3. **Write the handler:**
   ```javascript
   export const handler = async (event) => {
     console.log('Event:', JSON.stringify(event));
     return {
       statusCode: 200,
       body: JSON.stringify({ message: 'Hello from Lambda!' })
     };
   };
   ```
4. **Test it** with a sample event payload

#### Step 2: Trigger with API Gateway
1. **Add HTTP trigger:**
   - Method: `GET`
   - Path: `/hello`
2. **Deploy the API**
3. **Curl your endpoint:**
   ```bash
   curl https://abc123.execute-api.us-east-1.amazonaws.com/default/hello
   ```

#### Step 3: Environment Variables & Secrets
1. **Add config:**
   ```bash
   aws lambda update-function-configuration \
     --function-name my-first-function \
     --environment "Variables={DB_HOST=db.example.com,LOG_LEVEL=info}"
   ```
2. **Use AWS Secrets Manager** for sensitive values

#### Step 4: Monitoring with CloudWatch
1. **View logs** in `/aws/lambda/my-first-function`
2. **Add custom metrics:**
   ```javascript
   const AWS = require('aws-sdk');
   const cloudwatch = new AWS.CloudWatch();
   await cloudwatch.putMetricData({
     Namespace: 'MyApp',
     MetricData: [{ MetricName: 'Invocations', Value: 1 }]
   }).promise();
   ```
3. **Create alarms** for errors and throttles

#### Step 5: Production Best Practices
> **Pro tip:** Always set a timeout and memory size. Default 3s timeout often kills long-running jobs.

- **Use layers** for shared dependencies
- **Enable X-Ray tracing** for debugging
- **Set reserved concurrency** to prevent noisy-neighbor issues
- **Use provisioned concurrency** for latency-sensitive functions
- **Tag everything** for cost tracking

### Practice Project
- Build a REST API with 3 endpoints using Lambda + API Gateway
- Connect to DynamoDB for data persistence
- Add authentication with Cognito or API keys
- Set up CI/CD with GitHub Actions
- Write unit tests and integration tests
- Add structured logging and error tracking
- Document the API in OpenAPI spec
- Deploy to multiple regions for global users
- Set up CloudWatch alarms with SNS notifications
- Optimize cold starts with provisioned concurrency

### Key Takeaways
- Serverless reduces ops but introduces cold-start latency
- Always use infrastructure-as-code (SAM, CDK, or Terraform)
- Monitor costs — Lambda can become expensive at scale
- Event-driven architecture is the future of cloud apps
```

---

## 🎯 Quality Checklist

Before saving your `main.md`, verify:

- [ ] Every tool has `## ` heading with emoji
- [ ] Every tool has `**Difficulty:**` line
- [ ] Every tool has `**Time:**` line
- [ ] Every tool has `### Overview` section (non-empty)
- [ ] Every tool has `### Practice Project` section (with `- ` list items)
- [ ] All code blocks have a language identifier
- [ ] All steps use `#### Step N: Title` format
- [ ] Steps are numbered sequentially (1, 2, 3, ...)
- [ ] Links point to real, accessible URLs
- [ ] Commands are copy-pasteable and tested
- [ ] No `## ` headings inside code blocks

---

## 🚀 Adding a New Tool (Step-by-Step)

1. **Open `main.md`** in your editor
2. **Find the right place** — order tools by recommended learning path:
   - **Beginner tier:** Linux, Git, Python, basic networking
   - **Intermediate tier:** Docker, CI/CD, AWS basics, Ansible
   - **Advanced tier:** Kubernetes, Terraform, advanced networking, security
3. **Add a `---` separator** before your new tool (visual break)
4. **Copy the minimal template** above
5. **Fill in each section** following the field reference
6. **Save `main.md`** and refresh `index.html` in your browser
7. **Verify** the new tool card appears in the grid
8. **Click into it** and check that steps render, code blocks have copy buttons, and practice tasks are checkable

---

## 🔮 Future-Proofing Notes

This template is designed to support:

- ✅ Unlimited number of tools (no hardcoded list)
- ✅ Unlimited number of steps per tool
- ✅ Unlimited practice tasks per tool
- ✅ Mixed code languages per tool
- ✅ Custom icons via `[icon=name]` suffix
- ✅ Custom colors via `[color=name]` suffix
- ✅ Tables, images, callouts, links
- ✅ Markdown extensions (GFM-style)
- ✅ Internationalization (UTF-8 emoji + non-ASCII titles)

**If the parser needs new behavior later**, edit only the `parseMarkdown()` function in `index.html`. The data model (`STATE.tools[]`) is the contract — each tool object has:

```js
{
  id: string,         // auto-generated from title
  title: string,      // displayed name
  icon: string,       // lucide icon name
  color: string,      // CSS color variable
  colorBg: string,    // CSS color for backgrounds
  difficulty: string, // "beginner" | "intermediate" | "advanced"
  time: number,       // hours (first number from "X-Y hours")
  overview: string,   // raw overview text
  why: string,        // raw why-learn-this text
  steps: Array<{ number, title, content }>,
  checklist: string[], // practice project items
  content: string,    // full markdown section (for re-rendering)
}
```

**Never** edit `STATE.tools` directly — always modify `main.md`.

---

## 📞 Quick Reference Card

```
TOOL TEMPLATE
=============
## 🎯 Tool Name

**Difficulty:** Beginner|Intermediate|Advanced | **Time:** X-Y hours

### Overview
[2-4 sentences]

### Why Learn This
- [reason 1]
- [reason 2]

### Learn by Doing — Step by Step

#### Step 1: Title
1. Action with **bold label**
   ```bash
   command here
   ```

#### Step 2: Title
...

### Practice Project
- [task 1]
- [task 2]

### Key Takeaways
- [summary 1]
- [summary 2]
```

---

**Last Updated:** 2026-10-11 | **Maintained by:** You + AI