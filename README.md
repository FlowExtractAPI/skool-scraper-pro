# 📚 Skool Scraper Pro

Extract structured data from any **Skool community**  course lessons, videos, resources, the community post feed, comment threads, and even discover new communities by category or keyword  all from one actor, organized and ready to use.

Perfect for **educators**, **community owners**, **researchers**, and **growth/marketing teams** who need to archive course content, monitor community activity, analyze engagement, or find new communities to join or study.

---

## 🧩 Four Independent Sections

The actor is organized into four sections. Fill in only the one(s) relevant to what you want this run to do  you don't need to touch the others.

| Section | What it does |
|---|---|
| 📚 **Courses & Classroom** | Extract a community's courses, lessons, videos, and resources  or its full post feed |
| 💬 **Comments** | Pull a specific post's full content plus its comment thread |
| 🔍 **Discover Communities** | Browse or search `skool.com/discovery` to find communities by category, keyword, or filtered URL |
| 🔐 **Authentication** | Optional email+password or browser cookies, shared by every section above that needs it |

A typical workflow spans separate runs: use **🔍 Discover Communities** to find a community → **📚 Courses & Classroom** to extract its content → copy a post's URL from the results → **💬 Comments** to pull that post's comment thread.

---

## 🚀 Key Benefits & Use Cases

### **📚 For Educators & Community Owners**
- Archive your full course library  lessons, descriptions, videos, and resources
- Download every video hosted on Skool as an MP4 file, always at the best quality available
- Back up post content, attachments, and PDF handouts before they disappear
- Extract your community's entire post feed (the "wall"), not just classroom content

### **💼 For Researchers & Analysts**
- Pull a post's full comment thread, including nested replies, upvotes, and pinned comments
- Extract poll results and engagement data (upvotes, comment counts) from posts
- Build structured databases from classroom content or community activity

### **📈 For Growth & Marketing Teams**
- Discover new Skool communities by category, keyword, or price/language filters
- Monitor a competitor's or partner's community feed for new posts and activity
- Research community size, pricing, and engagement before reaching out or joining

### **🎯 For Content Teams**
- Bulk-export lesson descriptions and resource links for repurposing
- Download all post attachments (images, PDFs) in one automated run
- Get direct download links for every video  click to save the MP4 instantly

---

## ⚙️ Section 1  📚 Courses & Classroom

### Input: `communityUrls`
Accepts a bare community URL or its `/classroom` page, plus anything nested under classroom. What gets extracted is detected automatically from the URL shape:

| URL Format | What it does |
|---|---|
| `skool.com/community` (bare) | Full classroom  or the **feed** instead, if `extractCommunityFeed` is on |
| `skool.com/community/classroom` | Full classroom (all accessible courses) |
| `skool.com/community/classroom/courseId` | One specific course |
| `skool.com/community/classroom/courseId?md=lessonId` | One specific lesson |

You can mix multiple URLs of different types (and different communities) in a single run.

```json
{
  "communityUrls": [
    { "url": "https://www.skool.com/my-community/classroom" },
    { "url": "https://www.skool.com/my-community" }
  ],
  "extractCommunityFeed": true
}
```

### 🧵 Feed mode
A **bare** community URL with `extractCommunityFeed` on pulls the community's post feed (the "wall") instead of courses/lessons. Feed mode respects the exact sort/filter/page you paste in the URL itself  e.g. `?c=&s=newest&fl=unr&p=5` starts from page 5, sorted newest, filtered to unread. A plain bare URL with no query string pulls Skool's default feed order.

### `maxItemsPerSection` (Integer, default `10`)
One field, two meanings depending on mode:
- **Course/Classroom mode**: maximum lessons to pull from **each** course  every course gets its own budget, so a classroom with several courses returns this number **times the course count**, not this number in total.
- **Feed mode**: maximum **total** posts to pull from the community wall  a plain bare URL with no sort override can mean tens of thousands of posts, so this is what keeps a quick preview run from turning into an hours-long pull.

Set high (up to `100000`) to extract everything either way.

### `skipLockedCourses` (Boolean, default `true`)
Skip courses that require a membership level or VIP access you don't have. Turn off only if your credentials already unlock that content.

### Download options
| Field | Default | What it does |
|---|---|---|
| `downloadResources` | `false` | Downloads images/documents from each lesson's resources panel (links like YouTube are skipped  only actual files) |
| `downloadVideos` | `false` | Downloads every hosted-on-Skool video as an MP4, always at the best quality available |
| `downloadAttachments`* | `false` | Downloads images/documents attached to posts |

\* `downloadAttachments` is a **shared toggle**  it's configured once (in the Comments section below) and applies to lesson posts here too.

---

## ⚙️ Section 2  💬 Comments

### Input: `postUrls`
Direct post URLs only  either shape Skool itself uses:
- `skool.com/community/postName`
- `skool.com/community/post/postName`

Often copied straight from a Courses & Classroom run's results.

```json
{
  "postUrls": [
    { "url": "https://www.skool.com/my-community/weekly-wins-recap" }
  ],
  "maxCommentsPerPost": 200
}
```

### `maxCommentsPerPost` (Integer, default `100`)
Cap on comments plus replies extracted per post, combined.
- Set to `0` to skip comments entirely  you get just the post itself, no `comment` field.
- Set high (or to the post's total comment count) to get everything.

### Output shape
- **No comments fetched** (cap is `0`, or the post has none): **one row**, the post's own fields, `type: "post"`.
- **Comments fetched**: **one row per comment**  each row carries the post's full fields (repeated) plus that single comment nested under `comment`, `type: "comment"`. This means a post with 500 comments produces 500 rows, each self-contained and independently usable, rather than one giant combined row.

---

## ⚙️ Section 3  🔍 Discover Communities

Three independent ways to search  use any combination in the same run:

| Input | What it does |
|---|---|
| `discoveryUrls` | Paste a full `skool.com/discovery?...` URL copied from your browser after applying filters |
| `categorySearches` | Browse a category (Hobbies, Music, Money, Tech, Health, Sports, Self-improvement, Relationships, etc.) with Price/Visibility/Language/Sort filters |
| `keywordSearches` | Search by keyword, like typing into the Discovery search bar |

```json
{
  "keywordSearches": [
    { "keyword": "automation", "price": "free" }
  ],
  "discoveryMaxResults": 30
}
```

### `discoveryMaxResults` (Integer, default `30`)
Maximum communities returned **per search** (per Discovery URL, per category search, per keyword search  not a combined total). Set to `0` for unlimited  use with caution, as some categories return thousands of results.

### Output
Each discovered community includes its slug, URL, display name, description, member count, membership/pricing info, and logo  ready to feed into **Courses & Classroom** on a later run.

> ℹ️ This section works without providing any credentials below  Discovery normally challenges anonymous requests, but the actor handles that automatically.

---

## ⚙️ Section 4  🔐 Authentication

Needed for private/level-gated content, or if you want Comments/Courses requests to run under your own account rather than anonymously.

| Method | How to use | When to use |
|---|---|---|
| **Email + Password** (recommended) | Enter credentials in input | Easiest and most reliable |
| **Browser Cookies** | Export from your browser session | Use if login fails, or for advanced cases |
| **None** | Leave blank | Public communities only |

Tried in this order automatically: email + password, then cookies, then public access.

### 🔒 Credential Security
`password` and `cookies` are marked as **secret inputs** on the Apify platform  encrypted at rest the moment you save them, decrypted only inside the actor's own execution environment, and never written to logs or shown in the Console. If accessed directly from storage, only ciphertext is visible. Nothing here is specific to this actor  it's the same protection Apify applies to every secret input field, isolated per user and per actor.

### 🍪 How to export your cookies (optional)
1. Install the [Cookie-Editor extension](https://chrome.google.com/webstore/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm) (Chrome) or [equivalent for Firefox](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor/)
2. Log in to `skool.com` and open your community
3. Open the extension → click **Export**
4. Copy the JSON and paste it into the `cookies` field

> ⚠️ Cookies expire over time. If extraction starts failing on content that used to work, export fresh cookies and re-run.

### 🔒 What happens with locked/private content
If a post or community genuinely requires membership you don't have, the actor returns a clear, immediate error explaining that authentication or membership is required  rather than hanging, retrying forever, or silently returning nothing.

---

## ♻️ Reliability

- **Resumable runs**: if a run is interrupted  a platform restart, a timeout, or you stopping it yourself  starting it again picks up close to where it left off instead of starting over from scratch. This applies across every section: classroom/lesson progress, feed pagination, and comment-thread pagination are all checkpointed as the run goes.
- **Automatic handling of temporary blocks**: occasional anonymous-request blocks are retried automatically before any error is surfaced to you.

---

## 📊 Sample Output

All rows share one dataset, distinguished by a `type` field. Use the Output tab's view switcher to see each kind on its own.

### Lesson (Courses & Classroom)
```json
{
  "type": "lesson",
  "extractedAt": "2026-07-05T04:25:30.228Z",
  "community": "my-community",
  "communityUrl": "https://www.skool.com/my-community",
  "course": "✅ Start Here",
  "courseShortId": "7ddee36d",
  "section": "",
  "lesson": "Who am I? 🤔",
  "lessonId": "614723a946d645b6bef3145d391ecf24",
  "lessonUrl": "https://www.skool.com/my-community/classroom/7ddee36d?md=614723a9...",
  "position": 1,
  "descriptionText": "",
  "hostedVideo": {
    "source": "mux",
    "playbackUrl": "https://stream.mux.com/PLAYBACK_ID.m3u8?token=JWT",
    "durationMs": 286668,
    "aspectRatio": "3:2",
    "status": "ready",
    "fileSizeHuman": "81.2 MB",
    "downloadUrl": "https://api.apify.com/v2/key-value-stores/.../records/video-lesson-614723a9.mp4?signature=...",
    "direct_download": "https://api.apify.com/v2/key-value-stores/.../records/video-lesson-614723a9.mp4?signature=...&attachment=true"
  },
  "resources": [],
  "post": {
    "postId": "5b00705d9cf54ae7ae9273a64cffb9b7",
    "title": "Who am I?",
    "author": { "firstName": "John", "lastName": "Doe", "fullName": "John Doe" },
    "content": "...",
    "attachments": [],
    "upvotes": 15,
    "commentsCount": 7
  }
}
```

### Feed post (Courses & Classroom → Feed mode)
```json
{
  "type": "feedPost",
  "postId": "87a8f93ae3ef4cbc985a8c7d9ed03760",
  "postUrl": "https://www.skool.com/my-community/post/weekly-wins-recap",
  "title": "🏆 Weekly Wins Recap",
  "author": { "firstName": "Jane", "lastName": "Smith", "fullName": "Jane Smith" },
  "content": "...",
  "attachments": [],
  "upvotes": 106,
  "commentsCount": 123,
  "createdAt": "2026-07-03T20:40:01.291033Z"
}
```

### Post with a comment thread (Comments)
```json
{
  "type": "comment",
  "postId": "139cd2fa361e409e81b767a0fed3b9ec",
  "postUrl": "https://www.skool.com/my-community/post/please-read-rules",
  "title": "Please Read | Rules and Guidelines",
  "author": { "firstName": "Nate", "lastName": "Herk", "fullName": "Nate Herk" },
  "content": "...",
  "upvotes": 4926,
  "commentsCount": 2806,
  "comment": [
    {
      "commentId": "5141d15868d14cb6b438f83c213b0975",
      "depth": 0,
      "content": "Thank you!",
      "upvotes": 54,
      "pinned": false,
      "author": { "firstName": "Abdelhakim", "lastName": "Abdessemed", "fullName": "Abdelhakim Abdessemed" },
      "createdAt": "2024-10-23T17:22:13.113714Z"
    }
  ]
}
```

### Discovered community (Discover Communities)
```json
{
  "type": "community",
  "source": "keywordSearch",
  "communitySlug": "makerschool",
  "communityUrl": "https://www.skool.com/makerschool",
  "displayName": "Maker School: AI Automation",
  "description": "Get your first client for an AI automation business in 90 days...",
  "totalMembers": 2074,
  "membershipModel": 2,
  "displayPrice": { "currency": "usd", "amount": 5700, "recurringInterval": "month" },
  "logoUrl": "https://assets.skool.com/..."
}
```

---

## 💾 Key-Value Store

When download options are enabled, files are saved to the actor's Key-Value Store organized by type:

| Key prefix | Contents | Trigger |
|---|---|---|
| `image-{id}.{ext}` | JPEG, PNG, GIF, WebP, SVG, BMP, TIFF | `downloadAttachments` or `downloadResources` |
| `doc-{id}.{ext}` | PDF, Word, Excel, CSV, TXT, JSON, XML | `downloadAttachments` or `downloadResources` |
| `video-lesson-{id}.mp4` / `video-post-{id}.mp4` | Hosted video files | `downloadVideos` |

Each downloaded file is referenced by a `kvKey` (or `downloadUrl`/`direct_download` for videos) on its parent row in the dataset, so you can always trace a file back to its source.

---

## 🤝 Support & Resources

- 🌐 **Website**: [flowextractapi.com](https://flowextractapi.com)
- 📧 **Email**: [flowextractapi@outlook.com](mailto:flowextractapi@outlook.com)
- 🙋 **Apify Profile**: [dz_omar](https://apify.com/dz_omar?fpr=smcx63)
- 💬 **GitHub**: [FlowExtractAPI](https://github.com/FlowExtractAPI)

### Social Media
- 💼 **LinkedIn**: [flowextract-api](https://www.linkedin.com/in/flowextract-api/)
- 🐦 **Twitter**: [@FlowExtractAPI](https://x.com/@FlowExtractAPI)
- 📱 **Facebook**: [flowextractapi](https://www.facebook.com/flowextractapi)

---

## 🌟 Related Actors by FlowExtractAPI

### 📚 Education & Community
- **[Skool Scraper Pro](https://apify.com/dz_omar/skool-scraper-pro?fpr=smcx63)**  Lessons, videos, posts, comments, and community discovery from Skool
- **[Skool Map Scraper](https://apify.com/dz_omar/skool-map-scraper?fpr=smcx63)**  Member locations and profiles from Skool community maps

### 🎬 Video & Media
- **[Loom Scraper](https://apify.com/dz_omar/loom-video-scraper?fpr=smcx63)**  Loom video & transcript extraction
- **[YouTube Scraper Pro](https://apify.com/dz_omar/Youtube-Scraper-Pro?fpr=smcx63)**  Complete channel and playlist extraction
- **[Zoom Scraper](https://apify.com/dz_omar/zoom-scraper?fpr=smcx63)**  Download Zoom recordings and transcripts
- **[TikTok Video Downloader](https://apify.com/dz_omar/tiktok-video-downloader?fpr=smcx63)**  TikTok without watermarks

### 🛠️ Developer Tools
- **[Universal Downloader](https://apify.com/dz_omar/universal-downloader?fpr=smcx63)**  Download any file type with proxy support
- **[Ultimate Screenshot](https://apify.com/dz_omar/ultimate-screenshot?fpr=smcx63)**  Advanced website screenshot tool
- **[n8n Workflow Server](https://apify.com/dz_omar/n8n-workflow-server?fpr=smcx63)**  Run n8n automation on Apify

### 📱 Social & Ads
- **[Facebook Ads Scraper Pro](https://apify.com/dz_omar/facebook-ads-scraper-pro?fpr=smcx63)**  Facebook Ad Library extraction
- **[Google Ads Scraper](https://apify.com/dz_omar/google-ads-scraper?fpr=smcx63)**  Google Ads Transparency Center data

---
