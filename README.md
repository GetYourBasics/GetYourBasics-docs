# GetYourBasics — Documentation

Official documentation for all **GetYourBasics** plugins for Unreal Engine 5.

Every plugin in this catalog is built around the same philosophy: add a component,
configure it, and it works. No boilerplate, no complex setup, no days lost reading
source code to understand what's happening.

> Browse the full catalog on [FAB](https://www.fab.com/sellers/Get%20Your%20Basics).

## Plugins

### GYBSaveLoadSystem — Save & Load System

Mark your variables with `SaveGame`. Call `GYB_SaveGame`. Done.
Handles actors, actor references, runtime-spawned actors, GameInstance, and more.

→ [Documentation](GYBSaveLoadSystem/)

---

*More plugins coming soon.*

## How this repo is organized

Each plugin has its own folder with the same structure:

| File | What's inside |
|------|---------------|
| `quickstart.md` | Get the plugin working in under 10 minutes. |
| `blueprint-reference.md` | Every exposed node, its inputs, outputs and return values. |
| `integration-guide.md` | Add the plugin to an existing project step by step. |
| `faq.md` | Common questions, known gotchas and troubleshooting. |

The `shared/` folder contains guides that cover multiple plugins working together.

## Requirements

- Unreal Engine 5.3 or newer
- Works with Blueprint-only and C++ projects

## Support

Found a problem or have a question? Open an issue on this repository
or reach out through the FAB product page of the relevant plugin.

---

*GetYourBasics — studio-quality, plug & play systems for UE5.*
