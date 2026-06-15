# PlainPlan GitHub Action

Analyze Terraform plan JSON with PlainPlan and optionally post a PR-ready comment.

The API now returns `analysis.summary` by default. Richer fields like `pr_markdown`, `risk_flags`, `reviewer_checklist`, and `resources` are opt-in via `?include=` query parameters. This action requests `pr_markdown` automatically to minimize payload size.

## Usage

```yaml
- name: Analyze with PlainPlan
  uses: adamnmcc/plainplan-action@v1
  with:
    plan_file: plan.json
    api_key: ${{ secrets.PLANPLAIN_API_KEY }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

Generate `plan.json` first:

```yaml
- uses: hashicorp/setup-terraform@v3
  with:
    terraform_wrapper: false

- run: terraform init
- run: terraform plan -out=tfplan
- run: terraform show -json tfplan > plan.json
```

## Inputs

- `plan_file` (optional): Path to Terraform/OpenTofu plan JSON file. Default: `plan.json`
- `api_key` (required): PlainPlan API key from https://plainplan.click
- `api_url` (optional): API endpoint override. Default: `https://api.plainplan.click/api/analyze`
- `github_token` (optional): Token used to post PR comments. Default: `${{ github.token }}`
- `post_comment` (optional): `true` or `false`. Default: `true`

## Query Parameters

The action by default requests only `pr_markdown`. You can customize the API call by modifying `api_url` to request additional fields:

```yaml
# Request all optional fields
api_url: 'https://api.plainplan.click/api/analyze?include=all'

# Request multiple specific fields
api_url: 'https://api.plainplan.click/api/analyze?include=pr_markdown,risk_flags,resources'
```

Available include parameters:
- `pr_markdown`: Markdown-formatted analysis with emoji styling
- `risk_flags`: Detailed risk assessment array
- `reviewer_checklist`: List of action items for pull request review
- `resources`: Organized breakdown of created, updated, destroyed, and replaced resources
- `all`: Include all optional fields

## Outputs

- `summary`: Plain-English summary
- `risk_level`: Highest risk level (`HIGH`, `MEDIUM`, `LOW`, `NONE`)
- `pr_markdown`: Full markdown payload

## Required Workflow Permissions

```yaml
permissions:
  contents: read
  pull-requests: write
```

## Publish This Action

`action.yml` lives at the repository root, so users reference it directly:

- `adamnmcc/plainplan-action@v1`

To list it on the GitHub Marketplace: push a tag, then draft a release from it and tick
"Publish this Action to the GitHub Marketplace".

Release process:

1. Commit changes to `main`.
2. Create or move the major tag to latest stable commit:

```bash
git tag -a v1 -m "PlainPlan action v1"
git push origin v1
```

3. For updates, create a new immutable tag and then move `v1` to it:

```bash
git tag -a v1.0.1 -m "PlainPlan action v1.0.1"
git push origin v1.0.1

git tag -f v1
git push origin v1 --force
```

This gives users stable major version pinning while still allowing patch updates.
