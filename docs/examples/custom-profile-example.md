# Example: Custom Profile

Building a profile from scratch for a unique tech stack.

---

## Scenario

**Tech Stack:** Rust CLI tool with custom architecture

**Why Custom Profile:**
- No existing Rust CLI profile
- Team uses specific patterns
- Want to share with other Rust CLI projects

---

## Step 1: Create Directory

From the ai-flow-anything repo root (clone it first if you haven't: `git clone https://github.com/yusufkaraaslan/ai-flow-anything.git`):

```bash
cp -r profiles/generic profiles/rust-cli
```

---

## Step 2: Edit README.md

```markdown
# Profile: Rust CLI

## Description
Command-line tools written in Rust.

## Detection Hints
- `Cargo.toml` exists
- `src/main.rs` or `src/lib.rs`
- `Cargo.lock` exists

## Confidence
- 3/3: "Definitely Rust"
- 2/3: "Likely Rust"

## Capabilities
All 8 flows with Rust-specific patterns.

## Conventions
- Error handling with `anyhow`
- CLI parsing with `clap`
- Async with `tokio`
```

---

## Step 3: Edit discovery.md

```markdown
## Step 1: Framework

**Slot: cli-framework**
- Check `Cargo.toml`:
  - `clap` → Clap
  - `structopt` → StructOpt
  - `argh` → Argh

## Step 2: Async Runtime

**Slot: async-runtime**
- Check `Cargo.toml`:
  - `tokio` → Tokio
  - `async-std` → Async-std
  - None → Synchronous

## Step 3: Architecture

**Slot: architecture**
- Check `src/` structure:
  - `main.rs` + `lib.rs` → Simple
  - `commands/` → Multi-command
  - `bin/` → Multiple binaries
```

---

## Step 4: Edit rules.md

```markdown
# Rules: Rust CLI

## Error Handling

### Rule R1: Use anyhow for Errors
```rust
use anyhow::{Context, Result};

fn read_config() -> Result<Config> {
    fs::read_to_string("config.toml")
        .context("Failed to read config")?
}
```

## CLI Design

### Rule R2: Descriptive Help Text
Every command and argument must have help text.

### Rule R3: Sensible Defaults
Provide defaults for optional arguments.
```

---

## Step 5: Edit Flows

### design-flow.md

Add Rust-specific design:
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
```

### Error Propagation
```rust
fn main() -> Result<()> {
    let cli = Cli::parse();
    match cli.command {
        Commands::Add(args) => add(args)?,
        Commands::Remove(args) => remove(args)?,
    }
    Ok(())
}
```
```

### implement-flow.md

Add Rust implementation guidance to Phase 1 sub-task 1.3 IMPLEMENT (per the canonical 2-phase pattern):
```markdown
**1.3 IMPLEMENT** — Create Rust modules:
```rust
// src/commands/add.rs
use clap::Args;

#[derive(Args)]
pub struct AddArgs {
    #[arg(help = "Name of the item to add")]
    name: String,
}

pub fn execute(args: AddArgs) -> Result<()> {
    println!("Adding {}", args.name);
    Ok(())
}
```
```

---

## Step 6: Test

1. Create test Rust project
2. Initialize ai-flow-anything for it
3. Verify detection
4. Ask the AI to design a task called `test-task`
5. Verify output

---

## Step 7: Submit

```bash
git add profiles/rust-cli/
git commit -m "feat: add Rust CLI profile"
git push origin main
```

Create pull request to ai-flow-anything repository.

---

## Tips

- **Start with generic** — Copy and modify
- **Test on real projects** — Not just toy examples
- **Document clearly** — Help others use your profile
- **Include examples** — Show expected code output
- **Keep updated** — Update as Rust ecosystem evolves
