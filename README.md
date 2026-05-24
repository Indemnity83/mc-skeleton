# mc-skeleton

A production-grade multiloader Minecraft mod template for **Fabric** and **NeoForge**, targeting MC 26.1 (Java 25). Clone this to start a new mod with a full development workflow already configured.

## What's Included

- **Multiloader Gradle build** (common/fabric/neoforge subprojects via convention plugins)
- **Spotless formatting** with a pre-commit git hook
- **CI/CD workflows**: PR checks, pre-release publishing, full release publishing to Modrinth and CurseForge
- **Automated versioning** via release-please (semantic versioning, auto-changelog)
- **JUnit 5 + AssertJ** test infrastructure in the common module
- **Multi-branch strategy** with git worktree support documented in [CLAUDE.md](CLAUDE.md)

## Prerequisites

- **Java 25** (required by MC 26.1 — use [SDKMAN](https://sdkman.io/) or [Adoptium](https://adoptium.net/))
- A Minecraft account (for `runClient`)
- Git

## Getting Started

### 1. Clone and rename

```bash
git clone https://github.com/yourname/mc-skeleton my-mod
cd my-mod
# Disconnect from the skeleton's history
git remote remove origin
```

Then rename placeholders throughout the project:

| Find | Replace with |
|------|-------------|
| `template` | your mod ID (e.g., `coolmod`) |
| `Template` | your mod display name (e.g., `Cool Mod`) |
| `com.example.template` | your package (e.g., `com.yourname.coolmod`) |
| `com.example` | your base package (e.g., `com.yourname`) |
| `archives_base_name=template` | `archives_base_name=coolmod` |
| `Your Name (@yourhandle)` | your name |

Files to update:
- `gradle.properties` — `maven_group`, `archives_base_name`
- `settings.gradle` — `rootProject.name`
- `fabric/src/main/resources/fabric.mod.json` — id, name, description, authors, entrypoints
- `neoforge/src/main/resources/META-INF/neoforge.mods.toml` — modId, displayName, description, authors
- `release-please-config.json` — `package-name`
- All Java source files — package declarations and class names
- `CLAUDE.md` — update worktree paths and mod-specific references

### 2. Set up developer tooling

```bash
./gradlew installGitHooks
```

This installs the pre-commit hook that auto-formats your code with Spotless before every commit.

### 3. Build and test

```bash
./gradlew :common:test       # Run unit tests
./gradlew :fabric:jar        # Build the Fabric JAR
./gradlew :neoforge:jar      # Build the NeoForge JAR (may be unstable on beta MC versions)
./gradlew build              # Build everything + tests
```

### 4. Run in-game

```bash
# From repo root
./gradlew :fabric:runClient
./gradlew :neoforge:runClient
```

## Branch Structure

This template is designed for a **multi-branch, multi-version** strategy:

| Branch | MC Version | Purpose |
|--------|-----------|---------|
| `mc/1.21.1` | 1.21.1 | Stable maintenance |
| `mc/1.21.11` | 1.21.11 | Active development |
| `mc/26.1` | 26.1 | Snapshot forward-port |

Each branch targets a different MC version. Features are developed on one branch and **cherry-picked** to others. See [CLAUDE.md](CLAUDE.md) for the full development strategy including git worktree setup.

## CI/CD Setup

After creating your GitHub repository, configure these secrets and variables:

**Secrets** (Settings → Secrets and variables → Actions):
- `PERSONAL_TOKEN` — GitHub PAT with `contents:write` and `pull-requests:write` (for release-please)
- `GRADLE_ENCRYPTION_KEY` — any random string (encrypts Gradle cache in CI)
- `MODRINTH_TOKEN` — from modrinth.com → User Settings → API Tokens
- `CURSEFORGE_TOKEN` — from curseforge.com → API Tokens

**Variables**:
- `MODRINTH_PROJECT_ID` — your Modrinth project slug or ID
- `CURSEFORGE_PROJECT_ID` — your CurseForge project ID

**Workflows included:**

| Workflow | Trigger | Description |
|----------|---------|-------------|
| `check-pr.yml` | Every PR | Validates semantic title + checks loader isolation |
| `build-pr.yml` | PRs to `mc/*` | Format check, tests, build artifacts |
| `build-prerelease.yml` | Manual dispatch | Publishes a `-pre.N` build to Modrinth/CurseForge |
| `build-release.yml` | GitHub Release published | Publishes the release to Modrinth/CurseForge |
| `prepare-release.yml` | Push to `mc/*` | release-please creates/updates release PRs |

## Versioning

Versioning is fully automated. The release cycle:

1. Merge `fix:` or `feat:` PRs → release-please opens a release PR
2. Optionally publish a pre-release via `build-prerelease.yml` for community testing
3. Merge the release PR → GitHub Release is created automatically
4. `build-release.yml` triggers and publishes to Modrinth/CurseForge

Version format: `{semver}+mc{mc-version}.{loader}` (e.g., `0.1.0+mc26.1.fabric`)

Tag format: `mc{mc-version}-v{semver}` (e.g., `mc26.1-v0.1.0`)

## License

MIT — see [LICENSE](LICENSE). Replace `[YEAR]` and `[AUTHOR]` before publishing.
