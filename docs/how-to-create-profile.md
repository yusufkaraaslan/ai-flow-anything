# How to Create a Profile

Add support for your tech stack by creating a new profile.

---

## When to Create a Profile

Create a profile when:
- Your tech stack isn't covered by existing profiles
- Your team has unique conventions
- You want to share your workflow with others

---

## Profile Structure

```
profiles/your-profile/
├── README.md              # Profile manifest
├── discovery.md           # Discovery instructions
├── rules.md               # Profile-specific rules
└── skeletons/
    ├── design-flow.md
    ├── implement-flow.md
    ├── free-flow.md
    ├── parallel-implement-flow.md
    ├── pr-flow.md
    ├── test-flow.md
    ├── deploy-flow.md
    ├── docs-flow.md
    └── kb-sync-flow.md
```

---

## Step 1: Copy Generic Profile

Start with the generic profile as a template:

```bash
cp -r profiles/generic profiles/your-profile
```

---

## Step 2: Edit README.md

Update:
- **Description:** What tech stack this covers
- **Detection hints:** How to identify this project type
- **Confidence scoring:** How sure the AI should be

Example:
```markdown
# Profile: Rust CLI

## Description
Command-line tools written in Rust.

## Detection Hints
- `Cargo.toml` exists
- `src/main.rs` or `src/lib.rs`
- `Cargo.lock` exists

## Confidence
- 3/3 matches: "Definitely a Rust CLI"
- 2/3 matches: "Likely Rust — ask"
```

---

## Step 3: Edit discovery.md

Update discovery instructions:
- What files to read
- What patterns to search for
- What to ask the developer

Example:
```markdown
## Step 1: Framework

**Slot: framework**
- Check `Cargo.toml` dependencies:
  - `clap` → CLI parser
  - `structopt` → Derive-based CLI
  - `tokio` → Async runtime

## Step 2: Architecture

**Slot: architecture**
- Check `src/` structure:
  - `main.rs` + `lib.rs` → Simple
  - `bin/`, `commands/` → Multi-command
  - `modules/` → Modular
```

---

## Step 4: Edit rules.md

Update conventions:
- Language-specific rules
- Framework-specific rules
- Team conventions

Example:
```markdown
# Rules: Rust CLI

## Code Style

### Rule R1: Error Handling
Use `anyhow` or `thiserror` for error handling.

```rust
// ✓ Good
fn read_config() -> Result<Config> {
    let contents = fs::read_to_string("config.toml")
        .context("Failed to read config")?;
    toml::from_str(&contents)
        .context("Invalid config format")
}
```

## CLI Rules

### Rule R2: Help Text
Every command must have `--help` text.

### Rule R3: Exit Codes
Use appropriate exit codes:
- 0: Success
- 1: General error
- 2: Invalid arguments
```

---

## Step 5: Edit Flow Skeletons

Customize flows for your tech stack:

### design-flow.md
Add tech-specific design patterns:
```markdown
## Rust CLI Patterns

### Command Structure
```rust
#[derive(Parser)]
#[command(name = "myapp")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    Add { name: String },
    Remove { id: u32 },
}
```
```

### implement-flow.md
Add implementation patterns to Phase 1 sub-task 1.3 IMPLEMENT (per the canonical 2-phase pattern):
```markdown
**1.3 IMPLEMENT** — Create Rust modules:
```rust
// src/commands/add.rs
use clap::Args;

#[derive(Args)]
pub struct AddArgs {
    name: String,
}

pub fn execute(args: AddArgs) -> Result<()> {
    println!("Adding {}", args.name);
    Ok(())
}
```
```

---

## Step 6: Test Your Profile

1. Create a test project with your tech stack
2. Initialize ai-flow-anything for it
3. Verify detection works
4. Run each flow
5. Check generated output

---

## Step 7: Submit (Optional)

To share with the community:

1. Fork the ai-flow-anything repository
2. Add your profile to `profiles/`
3. Update main README with your profile
4. Submit a pull request

---

## Tips

- **Start simple:** Copy generic, make minimal changes
- **Test thoroughly:** Run on real projects
- **Document well:** Help others use your profile
- **Keep updated:** Update as your tech stack evolves
