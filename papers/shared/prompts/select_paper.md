---
name: select_paper
description: Register and activate a paper by filename
---

INPUT:
- manuscript_path (required)
- optional: paper_id
- optional: journal_id

OUTPUT:
- ai/config/active_paper.json
- ai/config/papers.json (updated if needed)

STEPS:

1) Resolve path
- Verify manuscript_path exists
- Normalize to repo-relative path

2) Derive paper_id
- If provided → use it
- Else → slugify filename (lowercase, hyphens, no extension)

3) Load registry
- ai/config/papers.json
- If invalid JSON → BLOCKED

4) Upsert entry
- If paper_id exists:
  - If manuscript path differs → ask for confirmation (or BLOCK in non-interactive mode)
- Else:
  - Add:
    {
      "id": "<paper_id>",
      "manuscript": "<normalized_path>"
    }

5) Write active paper
- ai/config/active_paper.json:
  { "active_paper_id": "<paper_id>" }

6) Optional journal
- If journal_id provided:
  - Validate via journal_profile_resolver
  - Write ai/config/active_journal.json

7) Ensure directories
- ai/out/peer_review/

8) Ensure journal profiles
- Check ai/config/journal_profiles.json exists
- If missing → WARN: copy shared/schemas/journal_profiles.json from the papers module into ai/config/
- Do not block on this; journal_profile_resolver will block if needed

RULES:
- Do not overwrite existing entries without confirmation
- Do not create duplicate IDs
- Do not move files automatically
- Fail if manuscript not found

STATUS:
- SUCCESS or BLOCKED with reason