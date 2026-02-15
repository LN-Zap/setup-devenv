# setup-devenv

Reusable activation step for workflows that rely on Devenv-defined tooling.

It provides a fast path for pre-baked runner images and a fallback path for dynamic activation, then verifies required commands/files so jobs fail early with clear errors.

## Counterpart action

`setup-devenv` is the runtime counterpart to `bake-devenv-image`:
- Use `bake-devenv-image` when preparing custom runner images.
- Use `setup-devenv` when consuming those images (or when dynamically activating Devenv).

Repository:
- https://github.com/LN-Zap/bake-devenv-image

## Scope and prerequisites

This action only handles environment activation and verification.

It does not by itself:
- check out your repository
- install Nix
- install `devenv`
- configure binary caches (for example via Cachix)

Typical prerequisite setup is still required in your workflow before using this action. For reference, see:
- https://devenv.sh/integrations/github-actions/

## Usage

```yaml
- uses: actions/checkout@v5

# Typical prerequisites on GitHub-hosted runners.
- uses: cachix/install-nix-action@v31
- uses: cachix/cachix-action@v16
  with:
    name: devenv
- run: nix profile install nixpkgs#devenv

- uses: LN-Zap/setup-devenv
  with:
    mode: auto
    verify_files: README.md
    verify_commands: |
      devenv
```

Mode behavior:
- `image`: requires a pre-baked activation script at `activation_script_path`
- `install`: requires `devenv` already available on `PATH`
- `auto`: uses activation script when present, otherwise falls back to dynamic `devenv` activation

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `mode` | no | `auto` | `auto`, `image`, or `install`. |
| `activation_script_path` | no | `/home/runner/copilot-devenv-activate.sh` | Path to pre-baked activation script. |
| `verify_files` | no | `''` | Newline-delimited files that must exist. |
| `verify_commands` | no | `devenv` | Newline-delimited commands that must resolve. |
| `runner_label_hint` | no | `''` | Optional runner label used in error messages. |

## Outputs

| Name | Description |
| --- | --- |
| `activation_mode` | `script`, `fallback`, or `install`. |
| `activation_seconds` | Activation + verification duration in seconds. |
| `devenv_bin` | Resolved `devenv` binary path. |
| `verification_status` | `ok` or `failed`. |
