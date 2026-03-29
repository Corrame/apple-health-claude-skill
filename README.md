# apple-health-claude-skill

A Claude Code skill that analyzes Apple Health export data as a **health detective + personal doctor** workflow.

## What it does

- Scans your `export.xml` for all available health indicators
- Traces data back to source devices (Apple Watch model, third-party apps) before drawing conclusions
- Investigates anomalies: circadian drift, RHR trends, VO2Max changes, sleep quality
- Cross-validates findings with you — you are the ground truth
- Builds a complete health portrait across all measurable dimensions
- Switches to doctor mode after the investigation is complete, giving grounded, actionable suggestions

## Requirements

- **Claude Code** (or equivalent AI tool with file system access + script execution)
- Python 3 with `pandas` and `numpy`
- Your Apple Health export: iPhone → Health → profile icon → Export All Health Data

This skill does **not** work in regular chat mode (web Claude, ChatGPT, etc.). It needs to read local files and run scripts.

## How to use

1. Export your Apple Health data from iPhone and place `export.xml` somewhere accessible
2. Open Claude Code in the directory containing your export
3. Say: "Analyze my Apple Health data" — the skill will trigger automatically

## What gets analyzed

- Resting heart rate (RHR) and long-term trends
- HRV (heart rate variability)
- VO2Max
- Sleep timing and circadian rhythm
- Sleep staging (deep sleep, REM) — Watch 10 / Series 9+ recommended
- Step count and actual workout records
- Respiratory rate, blood oxygen, wrist temperature
- Walking gait metrics
- Headphone audio exposure
- And more, depending on what your devices have recorded

## Key design principles

- **Device attribution first**: different devices have different algorithms; cross-device comparisons are flagged
- **No assumptions about lifestyle**: asks before assuming weekday/weekend patterns, shift work, etc.
- **Detective before doctor**: facts first, recommendations only after the full picture is clear
- **Honest about limits**: Apple Watch cannot measure stress, diet, or emotions — the skill says so explicitly

## License

MIT
