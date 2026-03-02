---
name: flex-analyzer
version: 2.1.0
description: AI-powered marketing analytics and campaign optimization engine
author: FlickClaw Engineering
tags: [marketing, ai, automation, analytics]
dependencies:
  - python>=3.9
  - openai>=4.0
  - pandas>=2.0
  - numpy>=1.24
  - scikit-learn>=1.2
  - matplotlib>=3.7
  - seaborn>=0.12
  - plotly>=5.14
  - jupyter>=1.0
  - google-analytics-data>=0.19
  - facebook-business>=22.0
  - tweepy>=4.14
  - requests>=2.31
required_env_vars:
  - OPENAI_API_KEY
  - GA4_CREDENTIALS_PATH (optional)
  - FB_ACCESS_TOKEN (optional)
  - TWITTER_BEARER_TOKEN (optional)
system_requirements:
  memory_min_gb: 4
  disk_min_gb: 10
  network: outbound HTTPS required
platforms: [linux, macos]
---

# Flex Analyzer

AI-powered marketing analytics and campaign optimization engine for data-driven decision making.

## Purpose

Flex Analyzer provides deep marketing intelligence by analyzing campaign performance, customer behavior patterns, content effectiveness, and competitive positioning. Used for:
- Predicting campaign ROI before launch
- Identifying content gaps and opportunities
- Automating A/B test analysis
- Generating weekly marketing performance reports
- Detecting audience segmentation anomalies
- Optimizing ad spend allocation
- Analyzing sentiment across social channels
- Creating data-backed content briefs

## Scope

### Primary Commands

- `flex-analyze campaign --source <csv/json/ga4/fb> --metrics <list> --period <dates>`
- `flex-analyze content --type <blog/social/email> --performance <csv> --generate-brief`
- `flex-analyze audience --segments <file> --behavior <events> --predict <metric>`
- `flex-analyze competitor --domains <list> --metrics <market-share/sentiment/backlinks>`
- `flex-analyze roi --campaigns <ids> --attribution <model> --budget <amount>`
- `flex-analyze sentiment --source <twitter/fb/comments> --query <string> --timeline <days>`
- `flex-analyze ab-test --results <csv> --confidence <0.95> --recommend`
- `flex-analyze forecast --historical <csv> --model <prophet/arima> --horizon <days>`
- `flex-analyze dashboard --export <html/pdf> --email <address> --schedule <cron>`

### Configuration Commands

- `flex-config init --api-key <key> --brand <name> --industry <sector>`
- `flex-config connectors --add-ga4 <credential-path> --add-fb <token> --add-twitter <token>`
- `flex-config models --select <gpt-4/claude-3/gpt-3.5-turbo> --temperature <0.0-1.0>`
- `flex-config alerts --thresholds <yaml> --notify <email/slack/webhook>`

## Work Process

### 1. Initialization & Validation
```bash
flex-config init --api-key sk-... --brand "Acme Corp" --industry "ecommerce"
```
Validates API keys, tests connections, creates `~/.flex-analyzer/config.yaml` with:
- Brand voice parameters
- Industry benchmarks
- KPI definitions
- Data retention policies

### 2. Data Ingestion
- For CSV/JSON: validate schema, check for nulls, normalize dates
- For GA4: authenticate with service account, query with `run_report()` using defined metrics
- For Facebook: use Marketing API to fetch campaign insights with breakdowns
- For Twitter: fetch tweets with query parameters, rate limit handling (450 req/15min)

### 3. AI Analysis Pipeline
1. **Preprocessing**: Outlier removal with IQR (1.5x), missing value imputation (median), log transform skewed metrics
2. **Feature Engineering**: Create derived metrics (CTR, CPC, ROAS, engagement rate), calculate rolling averages (7/30d)
3. **Segmentation**: K-means clustering (elbow method for k) on customer behavior vectors
4. **Insight Generation**: Prompt engineering with context:
   ```
   System: You are a senior marketing analyst with 10 years experience.
   Context: Brand {brand} in {industry}, analyzing {period} performance.
   Data: {summary_stats, top_performers, trends}
   Task: Provide 5 actionable insights with confidence scores (0-1).
   Format: JSON with fields: insight, confidence, supporting_data, recommended_action
   ```
5. **Visualization**: Generate Plotly charts (interactive HTML) or static PNGs with brand colors

### 4. Output Generation
- **JSON Report**: `analysis_<timestamp>.json` with all metrics, insights, predictions
- **HTML Dashboard**: Embedded charts, executive summary, exportable tables
- **Email Brief**: HTML formatted with CTA buttons for recommended actions
- **Slack Message**: Pre-formatted blocks with key metrics and thread link to full report

### 5. Automation (Optional)
- Cron job syntax: `0 8 * * 1 flex-analyze campaign --weekly --email team@company.com`
- Webhook integration: POST results to Notion/Asana/Trello
- Slack bot: `/flex-analyze last-week` triggers instant report

## Golden Rules

1. **Data Privacy**: Never log PII (email, phone, address). Hash user IDs with SHA-256 before storage. Delete raw data after 30 days (configurable).
2. **API Quotas**: Respect rate limits—implement exponential backoff (1s, 2s, 4s, 8s max). Alert when 80% quota consumed.
3. **Model Consistency**: Use same model version for comparable analyses. Track model ID in report metadata.
4. **Statistical Significance**: For A/B tests, require p-value < 0.05 AND minimum sample size 1000 per variant. Flag otherwise.
5. **Bias Mitigation**: Check for Simpson's paradox in segmented reports. Include diversity metrics (audience gender/age distribution).
6. **Attribution Accuracy**: Default to data-driven attribution (Markov chain). Only use last-click if explicitly requested.
7. **Export Security**: Encrypt all exports with `openssl enc -aes-256-cbc -salt -in file -out file.enc`. Store decryption keys separate.
8. **Audit Trail**: Log every command with user ID, timestamp, input hash, and output checksum in `~/.flex-analyzer/audit.log`.

## Examples

### Example 1: Campaign Analysis (Last 30 Days)
```bash
$ flex-analyze campaign \
  --source ga4 \
  --metrics "sessions,conversions,revenue,cost" \
  --period "2024-01-01,2024-01-31" \
  --breakdown "campaign,source_medium" \
  --confidence 0.95 \
  --output json
```
Output: `campaign_analysis_20240131_143022.json` containing:
```json
{
  "period": "2024-01-01 to 2024-01-31",
  "total_spend": 45678.90,
  "total_revenue": 234567.12,
  "roas": 5.14,
  "top_campaigns": [
    {"name": "Summer Sale FB", "roas": 8.42, "spend": 12345.00}
  ],
  "insights": [
    {
      "confidence": 0.94,
      "text": "Video ads CTR 47% higher than static images. Recommend shifting 30% budget to video.",
      "supporting_data": {"video_ctr": "2.3%", "image_ctr": "1.6%"}
    }
  ]
}
```

### Example 2: Content Brief Generation
```bash
$ flex-analyze content \
  --type blog \
  --performance top100_posts.csv \
  --generate-brief \
  --target-audience "small business owners" \
  --keywords ["email marketing", "automation", "ROI"]
```
Output: `content_brief_20240131.md`:
```markdown
# Content Brief: Email Marketing for SMBs

## Top Performing Patterns (based on 100 posts)
- Avg word count: 1,247 (optimal range: 1,200-1,400)
- Header structure: 3-5 H2s, 2 internal links
-CTA placement: 2nd paragraph + conclusion

## Recommended Topic Clusters
1. "Email Automation ROI Calculator" (search volume: 2,400/mo, difficulty: 45)
2. "Small Business Email Templates" (volume: 1,800/mo, difficulty: 38)

## Title Options (CTR predicted > 3.2%)
- "Email Marketing ROI: How We Generated $45K in 30 Days"
- "7 Email Automation Workflows for Busy Entrepreneurs"

## Content Outline
1. Problem statement (pain points)
2. Framework introduction
3. Step-by-step implementation
4. Template gallery
5. Expected outcomes (with calculator)

## SEO Recommendations
- Target keyword in first 100 words
- Include 3 FAQ schema questions
- Internal link to: /tools/email-calculator
```

### Example 3: Sentiment Analysis Dashboard
```bash
$ flex-analyze sentiment \
  --source twitter \
  --query "#FlexAnalyzer OR from:flex_claw" \
  --timeline 7 \
  --generate-dashboard \
  --api-key $TWITTER_BEARER
```
Creates `sentiment_dashboard_20240131.html` with:
- Real-time sentiment polarity chart (positive/neutral/negative)
- Top mentioned topics (word cloud)
- Influencer engagement metrics
- Peak activity hours heatmap
- Suggested response templates for negative mentions

### Example 4: A/B Test Recommendation
```bash
$ flex-analyze ab-test \
  --results ab_test_landing_page.csv \
  --confidence 0.99 \
  --recommend \
  --min-effect-size 0.05
```
CSV format:
```
variant,visitors,conversions
control,10470,843
treatment_a,10521,912
treatment_b,10512,876
```

Output:
```
✅ Statistically significant results detected (p = 0.023)

RECOMMENDATION: Treatment A
- Conversion rate: 8.66% (vs Control: 8.05%)
- Relative improvement: +7.6%
- Confidence interval: [95% CI 1.8% to 13.2%]
- Sessions needed for 95% power: 12,341 (current: 10,521)
- Expected monthly revenue uplift: $12,450

ROLLBACK TRIGGER: If conversion rate after 7 days < 7.8%, revert to Control.
```

## Dependencies & Requirements

**Python Packages** (installed via `pip install flex-analyzer`):
```
openai>=4.0  # GPT-4 for insights
pandas>=2.0   # Data manipulation
scikit-learn>=1.2  # Clustering, anomaly detection
plotly>=5.14  # Interactive charts
google-analytics-data>=0.19  # GA4 API
facebook-business>=22.0  # Meta Ads API
```

**External APIs** (require credentials):
- OpenAI API key (gpt-4-turbo-preview recommended, 4K context)
- Google Analytics 4 (service account JSON with Analytics Readonly)
- Facebook Marketing API (system user token with ads_read)
- Twitter API v2 (Bearer token with tweet.read, users.read)

**System**:
- Python 3.9+ with venv support
- 4GB RAM minimum (8GB recommended for >100K rows)
- 10GB disk for cache and reports
- Outbound HTTPS (port 443) to all API endpoints

**Configuration Files**:
- `~/.flex-analyzer/config.yaml` (auto-generated)
- `~/.flex-analyzer/credentials/` (encrypted tokens)
- `~/.flex-analyzer/cache/` (API response cache, TTL 1h)

## Verification

### Post-Installation Check
```bash
flex-analyzer --version
# Expected: Flex Analyzer 2.1.0

flex-config validate --all
# Expected checks:
# ✓ OpenAI API key: valid
# ✓ GA4 credentials: accessible
# ✓ Facebook token: permissions OK
# ✓ Disk space: 12.4GB free (min 10GB)
# ✓ Memory: 8.2GB available (min 4GB)
```

### Test Dataset
```bash
flex-analyzer test-run --sample marketing_sample_2024.csv
# Should produce:
# - analysis_test_<timestamp>.json
# - dashboard_test_<timestamp>.html
# - Log: All insights confidence > 0.75
# - No warnings about missing data
```

### Health Monitor
```bash
flex-analyzer health
# Output:
# {
#   "status": "healthy",
#   "last_successful_run": "2024-01-31T14:30:22Z",
#   "api_quotas": {
#     "openai": "12% used",
#     "ga4": "3% used",
#     "facebook": "18% used"
#   },
#   "cache_size": "2.3GB",
#   "disk_available": "12.1GB"
# }
```

## Troubleshooting

### Error: `OpenAI API quota exceeded`
**Symptom**: `openai.RateLimitError: You exceeded your current quota`
**Fix**:
1. Check usage: https://platform.openai.com/usage
2. Reduce batch size: add `--batch-size 5` (default 10)
3. Switch to cheaper model: `flex-config models --select gpt-3.5-turbo`
4. If critical, use `--no-ai` flag for statistical-only analysis

### Error: `GA4 property not found`
**Symptom**: `google.api_core.exceptions.NotFound: Not found: Property`
**Fix**:
1. Verify service account has `Analytics Readonly` role in GA4 Admin
2. Check `GA4_CREDENTIALS_PATH` points to valid JSON (not expired)
3. Test connection: `flex-config connectors --test-ga4`
4. Ensure property ID matches: `properties/123456789` (not just `123456789`)

### Error: `Facebook Marketing API permission denied`
**Symptom**: `(#200) The user hasn't authorized the application...`
**Fix**:
1. Regenerate system user token with `ads_read` permission
2. Confirm token hasn't expired (90-day tokens)
3. Assign token to ad account in Business Settings
4. Test: `flex-config connectors --test-fb`

### Error: `MemoryError during large dataset processing`
**Symptom**: Process killed with exit code 137
**Fix**:
1. Enable chunked processing: `--chunk-size 10000`
2. Increase swap: `sudo fallocate -l 8G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile`
3. Filter data: `--filters "date >= '2024-01-01' and spend > 0"`
4. Sample: `--sample 0.1` (10% of data)

### Error: `Invalid confidence threshold`
**Symptom**: `ValueError: Confidence must be between 0.8 and 0.999`
**Fix**: Use valid range: `--confidence 0.95` (default) or `--confidence 0.99` for strict tests. Do not use 0.99 for small samples (<10K).

### Error: `Certificate verify failed`
**Symptom**: `ssl.SSLCertVerificationError` during API calls
**Fix**:
```bash
# Update CA bundle
sudo apt-get install ca-certificates  # Ubuntu/Debian
sudo update-ca-certificates

# Or bypass (not recommended): export SSL_CERT_FILE=/path/to/cacert.pem
```

### Warning: `High missing data in column X (45% null)`
**Implication**: Analysis results may be unreliable.
**Action**:
1. Inspect data source: `flex-analyze inspect --source raw_data.csv`
2. Impute nulls: `--impute median` or `--impute forward-fill`
3. Drop column: `--exclude-columns X,Y,Z`
4. Contact data provider if unexpected

## Rollback Commands

### Rollback Recent Analysis
```bash
# List last 10 analyses
flex-analyzer history --limit 10

# Restore previous version (by timestamp)
flex-analyzer rollback --timestamp 20240130_093022 \
  --restore-config \
  --notify-team
```
Rollback includes:
- Config file (`~/.flex-analyzer/config.yaml`)
- Model selection
- Credential references (not secrets)
- Alert thresholds

### Restore from Backup
```bash
# Full system restore (config + cache)
flex-analyzer restore --backup ~/.flex-analyzer/backups/20240128_120000.tar.gz \
  --verify-checksum \
  --dry-run  # Remove to execute
```

### Undo Last Configuration Change
```bash
flex-config undo --steps 1  # Revert last change
flex-config undo --steps 3  # Revert last 3 changes
```
Uses git-like history stored in `~/.flex-analyzer/.config_history/`.

### Emergency Stop (Kill Running Jobs)
```bash
flex-analyzer stop --all --force
# Sends SIGINT, waits 5s, then SIGKILL
# Clears temp files in /tmp/flex-analyzer-*
# Releases API locks
```

### Revert AI Model Version
```bash
flex-config models --list
# Shows: gpt-4-turbo (current), gpt-3.5-turbo, claude-3-opus

flex-config models --select gpt-3.5-turbo-1106  # Downgrade
# Re-runs last analysis with old model for comparison
```

### Revert to Previous Insight Style
```bash
flex-config insights --restore-defaults \
  --format markdown \
  --confidence-threshold 0.8
# Overrides any custom prompt templates
```

### Clean Cache & Re-fetch
```bash
flex-analyzer cache --clear --older-than 7d  # Clear 7+ day old
flex-analyzer cache --clear-all  # Nuclear option
flex-analyzer refresh --force --sources ga4,facebook  # Re-download
```

### Database Migration Rollback (if using PostgreSQL)
```bash
# Flex Analyzer can use local SQLite (default) or PostgreSQL
flex-db downgrade --target v1.2.0  # Last stable version
flex-db verify --integrity  # Check row counts and checksums
```

## Common Issues & Quick Fixes

| Symptom | Likely Cause | Command to Fix |
|---------|--------------|----------------|
| `401 Unauthorized` | Expired API token | `flex-config connectors --refresh-all` |
| `Slow GA4 queries` | Date range too large | Add `--max-rows 100000` or narrow period |
| `No social data` | Twitter API v2 not enabled | Enable in dev portal, recreate Bearer token |
| `Empty report` | All data filtered out | Remove `--filters`, check date format (YYYY-MM-DD) |
| `plt.show() blocked` | Non-interactive backend | Set `MPLBACKEND=Agg` in env or use `--no-display` |
| `JSON decode error` | API quota exceeded mid-run | Retry with `--retry 3 --backoff exponential` |
| `File not found` | Wrong GA4 credential path | `export GA4_CREDENTIALS_PATH=~/creds/ga4.json` |
| `Slack webhook failed` | Invalid URL format | Verify webhook ends with `/services/...` |

## Advanced Usage

### Multi-source Attribution Model
```bash
flex-analyze roi \
  --campaigns "fb_summer,tw_holiday,google_pmax" \
  --attribution MarkovChain \
  --lookback-window 90d \
  --custom-weights "direct:0.4,organic:0.3,social:0.2,email:0.1"
```

### Automated Weekly Report with Slack
```bash
# Set up in crontab:
0 8 * * 1 flex-analyze campaign \
  --weekly \
  --generate-dashboard \
  --slack-webhook https://hooks.slack.com/services/... \
  --email marketing@company.com
```

### Competitor Intelligence (Batch)
```bash
flex-analyze competitor \
  --domains "competitor1.com,competitor2.com,competitor3.com" \
  --metrics "backlinks,domain-authority,top-keywords,ad-spend-estimate" \
  --output competitors_analysis_$(date +%Y%m).xlsx
```

### Custom Prompt Injection
```bash
flex-analyze content \
  --performance posts.csv \
  --custom-prompt "
    Analyze these posts and identify gaps in keyword coverage.
    Focus on long-tail opportunities with KD < 30.
    Return JSON: {gaps: [{keyword, volume, kd, current_rank}]}
  " \
  --output json
```
```