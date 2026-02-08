# ADR-0001: Replace live README stats service with workflow-generated static assets

## Status

Accepted (2026-02-08)

## Context

The GitHub profile README previously embedded live-generated SVG cards (GitHub stats / top languages)
served via a public `github-readme-stats` deployment (Vercel). This introduced an external runtime dependency
outside of GitHub’s control plane.

Observed failure mode: the upstream public deployment can be paused or rate-limited, resulting in broken
images on the GitHub profile. This is a reputational issue (profile content degrades) and introduces
unnecessary operational fragility for non-critical, non-interactive content.

Constraints / goals:

- High reliability of profile rendering (no broken external embeds)
- Minimal operational surface area (no public service to maintain)
- Low/no ongoing cost
- Ability to support dark/light variants
- Preserve optional private-repo counting via token usage (where permitted)

## Decision

Generate README stats cards as static SVG assets via GitHub Actions on a schedule (and on-demand),
commit the generated SVGs to the repository, and reference them using local paths in the profile README.
Provide separate dark/light assets and use `prefers-color-scheme` / theme selectors for automatic switching.

## Rationale

- Live public endpoints introduce availability and rate‑limit risk for non‑critical profile content.
- Static assets eliminate runtime dependencies while preserving the value of the stats signal.
- GitHub Actions provides a native, low‑cost scheduling mechanism with minimal operational surface area.
- Dark/light variants can be generated deterministically without client‑side services.
- Tradeoff: eventual consistency is acceptable for profile metrics in exchange for reliability and simplicity.

## Consequences

Positive:

- Eliminates external runtime dependency and availability failures for profile rendering
- No public service to secure/monitor; reduced attack surface
- Predictable updates with caching and scheduled regeneration
- Dark/light support via pre-generated assets
- Cost bounded to GitHub Actions usage (typically negligible)

Negative / tradeoffs:

- Stats are eventually consistent (updated on schedule, not live)
- Requires committing generated assets (repository contains derived artifacts)
- Workflow requires a token for private counts; without token, counts may be partial

## Related

- GitHub Actions workflow for static card generation (profile repository)
- Public github-readme-stats service reliability issues motivating the pivot

## Implementation notes

- GitHub Actions workflow generates `stats-{dark,light}.svg` and `top-langs-{dark,light}.svg`
- README references local `./profile/*.svg` assets with theme selection
