# betterai-skills

Skill/rule source for the BetterAI harness auto-sync (`BETTERAI_SKILLS_REPO_URL`).

Layout contract:
- `rules/<CATEGORY>/<domain>/<id>.md` and `skills/<category>/<id>.md` — corpus
  markdown with YAML frontmatter (`id`, `title`, `category`, ...). Every file
  must parse as a BetterAI Artifact or the whole sync aborts.
- The harness syncs the tarball of `main` on a TTL (stale-while-revalidate)
  into `rules/synced-github/` + `skills/synced-github/` in the corpus.
- Ids must NOT collide with artifacts that already live outside the synced
  dirs in a host's corpus — collisions abort the sync by design.
