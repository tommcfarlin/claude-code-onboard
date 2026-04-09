<!--
SUMMARY TEMPLATE — used by the onboard skill.

This defines the format of the cached summary stored at
<git-common-dir>/claude-onboard.md. The skill fills this
template during full scans and preserves its structure
during delta updates.

Remove this comment block from the generated summary.
-->

---
sha: {{SHA}}
branch: {{BRANCH}}
timestamp: {{TIMESTAMP}}
mode: {{MODE}}
---

## Project Summary

{{PROJECT_SUMMARY}}

## Technologies

{{TECHNOLOGIES}}

## Directory Structure

{{DIRECTORY_STRUCTURE}}

## Key Entry Points

{{KEY_ENTRY_POINTS}}

## Architecture Notes

{{ARCHITECTURE_NOTES}}
