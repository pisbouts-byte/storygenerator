# Sizing Calibration Template

Empty until a team fills it in from a completed project's actuals. Referenced by SKILL.md §6 — when
a filled-in version exists (copy this file to `sizing_calibration_<platform>.md` and fill in real
rows), the skill matches each new story's Sizing Drivers against the closest archetype row here and
uses the actual hours as a sanity check on the generated Fibonacci size, instead of applying one flat
points-to-hours conversion factor across every story regardless of what it actually involved.

Without a filled-in version of this file, the skill sizes from first principles and says so plainly —
don't let it silently invent calibration data.

## How to fill this in

After a project closes (or at each sprint retro, incrementally), pull completed stories and record:
the story's actual archetype (not its title — a reusable category like "single-screen field
validation" or "new REST integration, single data object, synchronous"), its Sizing Drivers as
originally estimated, the size it was given, and the actual hours it took. Over a few projects on the
same platform this becomes a real reference instead of a guess.

| Archetype | Typical Sizing Drivers | Generated Size (Fibonacci) | Actual Hours (observed) | Notes |
|---|---|---|---|---|
| _(example)_ Single-screen field validation, no integration | 1 screen, 1–3 rules, 0 integrations | 2 | 4–6 | Straightforward DX component + validation rule |
| _(example)_ New REST integration, single data object, synchronous | 0 screens, 1 rule, 1 integration, 1 data object | 5 | 12–18 | Includes connector build + data page + error handling |
| _(example)_ New REST integration, multiple data objects, retry/reconciliation logic | 0 screens, 2–3 rules, 1 integration, 3+ data objects | 8–13 | 30–45 | Wide range depending on reconciliation complexity — split further if it lands at 13 |
| _(example)_ Correspondence template + content rule, single variant | 1 template, 2–4 rules, 0 integrations | 3 | 8–10 | Per variant — see platform_patterns_pega.md's correspondence guidance for why this is usually many stories, not one |

Delete the example rows once real data replaces them, or keep them clearly marked as examples if the
team wants a placeholder scale to start from on day one of a new platform.
