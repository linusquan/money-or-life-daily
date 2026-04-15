# Daily Scrape Workflow — Money or Life 美股频道

This document defines the full workflow for scraping, summarizing, and publishing daily content from YouTube channel [@Money_or_Life](https://www.youtube.com/@Money_or_Life).

## Channel Context

- **Channel**: Money or Life (@Money_or_Life)
- **Host**: Ace
- **Focus**: US stock market — high tech, AI, space, healthcare
- **Language**: Chinese (Mandarin)
- **Content types**: Video analysis, community posts
- **Site**: https://linusquan.github.io/money-or-life-daily/
- **Repo**: /Users/liquan/code/money-or-life-daily

## Schedule

Run daily at 9:00 AM local time. Check for new content published since the last scrape.

## Step 1: Scrape YouTube Channel

Use Playwright browser automation (YouTube is fully client-side rendered — WebFetch won't work).

1. Navigate to `https://www.youtube.com/@Money_or_Life/videos`
2. Take a snapshot to get the list of recent videos
3. Identify videos published today (or since last scrape)
4. For each new video:
   a. Navigate to the video page
   b. Extract: title, URL, date, duration, views, likes, comments
   c. Click "...more" to expand description
   d. Click "Show transcript" button to open transcript panel
   e. Extract full transcript text from `ytd-transcript-renderer`
   f. If transcript unavailable, use description + title for summary
5. Navigate to `https://www.youtube.com/@Money_or_Life/community`
6. Extract community posts from today: date, content, likes, comments

## Step 2: Generate Summary (Chinese)

All output must be in Chinese. Use the transcript and video metadata to generate:

### Video Fields

| Field | Description |
|-------|-------------|
| `title` | Video title (as-is from YouTube) |
| `url` | Full YouTube URL |
| `date` | Publication date (YYYY-MM-DD) |
| `duration` | Video duration (MM:SS or HH:MM:SS) |
| `views` | View count (use "k" for thousands) |
| `likes` | Like count |
| `comments` | Comment count |
| `summary` | 2-3 sentence Chinese summary of the video content |
| `keyPoints` | Array of 5-10 bullet points covering core arguments, data points, and conclusions |
| `table` | Optional comparison table if video compares stocks/companies. Object with `title`, `headers` (array), `rows` (array of arrays) |
| `stocks` | Array of stock tickers mentioned (most discussed first) |
| `investmentView` | Ace's personal investment stance — positions, sizing, entry prices, outlook |
| `trades` | Array of specific trading actions (see schema below) |
| `independentAnalysis` | AI-generated independent research (see schema below) |

### Trades Schema

Extract every specific buy/sell/hold/options action Ace mentions:

```json
{
  "trades": [
    {
      "ticker": "IRDM",
      "action": "hold",
      "positionSize": "3%",
      "entryPrice": "$29",
      "reasoning": "稳健成长股，卫星通信唯一盈利公司，但已重仓RKLB和SATS不会加仓"
    }
  ]
}
```

Possible `action` values: `buy`, `sell`, `hold`, `add`, `trim`, `call`, `put`, `covered-call`, `watching`

### Independent Analysis Schema

For each major stock discussed, generate independent research:

```json
{
  "independentAnalysis": {
    "summary": "对Ace本期提到的主要标的进行独立分析...",
    "stocks": [
      {
        "ticker": "IRDM",
        "fundamentals": "P/E 31.6x, 营收增长12% YoY, 净利润率13.1%, 自由现金流稳定",
        "bullCase": "卫星通信唯一盈利公司，3GPP Release 19纳入L频段标准化，政府合同稳定",
        "bearCase": "Starlink竞争压力持续，用户增长可能见顶，频谱资源不如VSAT",
        "risksNotMentioned": "第二代星座卫星2030年后需要更换，资本开支压力；与高通直连手机合作已失败",
        "consensus": "分析师中位目标价$35，12个买入/5个持有/1个卖出"
      }
    ]
  }
}
```

### Community Posts Schema

```json
{
  "communityPosts": [
    {
      "date": "2026-04-11（4天前）",
      "content": "Post content summarized in Chinese...",
      "likes": 87,
      "comments": 4
    }
  ]
}
```

## Step 3: Write JSON Data

1. Write daily JSON to `/Users/liquan/code/money-or-life-daily/data/YYYY-MM-DD.json`
2. Update `/Users/liquan/code/money-or-life-daily/data/index.json` — add the new date to the array
3. **JSON safety**: Use `「」` for Chinese quotes inside strings, never `""`  which breaks JSON parsing

## Step 4: Git Commit & Push

```bash
cd /Users/liquan/code/money-or-life-daily
git add data/
git commit -m "add daily summary for YYYY-MM-DD"
git push origin main
```

This triggers GitHub Pages deployment automatically (legacy mode, deploys from main branch).

## Step 5: Verify

After push, wait ~60 seconds, then verify the site loads correctly:
- Navigate to https://linusquan.github.io/money-or-life-daily/
- Check that the new date appears in the dropdown
- Check that content renders properly

## Output JSON Template

Full template for a day's data file:

```json
{
  "videos": [
    {
      "title": "视频标题",
      "url": "https://www.youtube.com/watch?v=...",
      "date": "YYYY-MM-DD",
      "duration": "MM:SS",
      "views": "16k",
      "likes": "509",
      "comments": "49",
      "summary": "2-3句中文摘要...",
      "keyPoints": [
        "要点1...",
        "要点2..."
      ],
      "table": {
        "title": "对比表标题",
        "headers": ["列1", "列2", "列3"],
        "rows": [
          ["数据1", "数据2", "数据3"]
        ]
      },
      "stocks": ["TICKER1", "TICKER2"],
      "investmentView": "Ace的投资观点...",
      "trades": [
        {
          "ticker": "TICKER",
          "action": "buy|sell|hold|add|trim|call|put|covered-call|watching",
          "positionSize": "3%",
          "entryPrice": "$29",
          "reasoning": "买卖原因..."
        }
      ],
      "independentAnalysis": {
        "summary": "独立分析总结...",
        "stocks": [
          {
            "ticker": "TICKER",
            "fundamentals": "基本面数据...",
            "bullCase": "看多理由...",
            "bearCase": "看空理由...",
            "risksNotMentioned": "Ace未提及的风险...",
            "consensus": "分析师共识..."
          }
        ]
      }
    }
  ],
  "communityPosts": [
    {
      "date": "YYYY-MM-DD（X天前）",
      "content": "帖子内容...",
      "likes": 0,
      "comments": 0
    }
  ]
}
```

## Notes

- If no new content was published that day, skip — don't create an empty file
- Multiple videos in one day: include all in the `videos` array
- Community posts: include posts from the last few days that haven't been captured yet
- The independent analysis should be genuinely useful — check current market data, not just restate what Ace said
