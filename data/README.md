# AIDev-pop Dataset

Raw `AIDev-pop` files across the 5 AI coding agents analyzed in this project (`Claude_Code`, `Copilot`, `Cursor`, `Devin`, `OpenAI_Codex`) are excluded from Git via `.gitignore` so third-party scraped PR bodies and oversized JSON tables (`100 MB - 893 MB`) stay out of repository history.

## Official Upstream Dataset

`AIDev-pop` is the >100-star popular repository subset of the **AIDev** dataset:

- **Hugging Face Dataset:** [hao-li/AIDev](https://huggingface.co/datasets/hao-li/AIDev)
- **Research Replication Repo:** [SAILResearch/AI_Teammates_in_SE3](https://github.com/SAILResearch/AI_Teammates_in_SE3)

Download the dataset into `data/aidev_pop/scope/<Agent>/` (or point `AIDEV_DATA_DIR` to your extracted `scope/` folder) before running the notebook.

## Files Used by the Notebook

| File | Description |
|---|---|
| `prs.csv` | Pull request metadata (`id`, `title`, `body`, `created_at`, `merged_at`, `state`) |
| `pr_reviews.json` | Review events per PR ID, used to compute `review_cycles` and `target_heavy_review` (`>= 2` cycles) |
| `pr_commit_details.json` | Per-commit stats (`additions`, `deletions`) and modified filenames (`files_touched`) |

## Engineered ML Matrix (`33,600 x 18`)

- **Target:** `target_heavy_review` (`0` = 0-1 review cycles, `1` = >=2 review cycles)
- **Numeric features:** `pr_desc_length`, `hours_to_merge`, `total_additions`, `total_deletions`, `files_touched`, `total_changes`
- **One-hot encoded:** `task_type_*` (5 categories) and `agent_identity_*` (5 AI agents)
