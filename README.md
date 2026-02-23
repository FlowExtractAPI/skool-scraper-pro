# 📚 Skool Scraper Pro

Extract structured data from any **Skool community classroom** lessons, descriptions, videos, attachments, posts, and polls organized and ready to use.

Perfect for **educators**, **community owners**, and **researchers** who need to archive, analyze, or repurpose their Skool course content without manual effort.

---

## 🚀 Key Benefits & Use Cases

### **📚 For Educators & Community Owners**
- Archive your full course library with descriptions, videos, and lesson resources
- Back up post content, attachments, and PDF handouts before they disappear
- Generate a searchable index of every lesson across all your courses

### **💼 For Researchers & Analysts**
- Extract poll results and engagement data (upvotes, comment counts) from course posts
- Build structured databases from community classroom content
- Analyze content distribution across courses and sections

### **🎯 For Content Teams**
- Bulk-export lesson descriptions and resource links for repurposing
- Download all post attachments (images, PDFs) in one automated run
- Map full course structures with sections, positions, and metadata

---

## 🔍 What Gets Extracted

### 📖 **Lesson Data**
- Title, position, section, course
- Full description (plain text + raw ProseMirror format)
- Created and updated timestamps
- Direct lesson URL

### 🎬 **Video Data**
- Hosted Mux videos: playback URL, duration, thumbnail, aspect ratio
- Signed JWT playback token (ready to use with ffmpeg or a video player)

### 🔗 **Resources Panel**
- All links from the lesson resources panel with platform detection (YouTube, Google Drive, etc.)
- Downloadable files (PDFs, images) identified with `fileType` field
- Optional download to Key-Value Store with `kvKey` reference

### 📝 **Post Data** (linked to each lesson)
- Full post content, author, contributors
- All extracted URLs with platform detection
- Poll results (option text + vote counts)
- Upvotes and comment count

### 📎 **Post Attachments**
- Images and documents (PNG, JPEG, PDF, Word, Excel, etc.)
- File name, size, content type, download URL
- Optional download to Key-Value Store with `kvKey` reference

### 🎥 **Post Videos**
- Mux-hosted videos uploaded directly to posts
- Full playback URL, duration, thumbnail

---

## ⚙️ Input Options

### 🔗 URL Modes — Four ways to scope your extraction

The actor detects what to extract automatically from the URL format:

| URL Format | What it does |
|---|---|
| `skool.com/group/classroom` | Extract **all accessible courses** |
| `skool.com/group/classroom/38fd1c1d` | Extract **one specific course** |
| `skool.com/group/classroom/38fd1c1d?md=lessonId` | Extract **one specific lesson** |
| `skool.com/group/postName` | Extract **one specific post** (post data only) |

You can mix multiple URLs of different types in a single run.

```json
{
  "communityUrl": [
    "https://www.skool.com/my-community/classroom",
    "https://www.skool.com/my-community/01"
  ]
}
```

### 🔐 Authentication

The actor tries authentication in priority order — stopping as soon as one method succeeds:

| Priority | Method | When to use |
|---|---|---|
| 1 | **Email + Password** | Easiest — just enter your Skool credentials |
| 2 | **Browser Cookies** | Fallback if login fails, or if you prefer cookie-based auth |
| 3 | **None** | Public communities only |

#### Method 1 — Email + Password (recommended)

Set `email` and `password` in the actor input. The actor logs into Skool automatically, obtains a fresh session, and proceeds. Your password is stored as a secret and is never logged.

```json
{
  "email": "you@example.com",
  "password": "your-password"
}
```

If login fails (wrong credentials, account issue), the actor automatically falls back to browser cookies if you also provided them.

#### Method 2 — Browser Cookies (fallback)

If you prefer not to share your password, or if automatic login fails, you can export your browser session cookies and paste them in.

**How to get your cookies:**

1. **Install the Cookie-Editor extension**
   - [Chrome](https://chrome.google.com/webstore/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
   - [Firefox](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor/)

2. **Log in to Skool**, then navigate to your community

3. **Open Cookie-Editor** → click **Export** → copy the JSON

4. **Paste into the `cookies` field** in actor input:

```json
{
  "cookies": [
    { "name": "auth_token", "value": "your-token-value", "domain": ".skool.com" }
  ]
}
```

> ⚠️ Cookies expire. If extraction fails with 403 errors, export fresh cookies and re-run.

### ⬇️ Download Options

#### `downloadAttachments` (Boolean)
- **Default**: `false`
- Downloads images and document files (PDF, Word, Excel, etc.) attached to lesson posts
- Saved to Key-Value Store with keys like `image-{id}.png` / `doc-{id}.pdf`
- Each attachment in the dataset gets a `kvKey` field pointing to its stored file
- ⚠️ Increases run time — large communities may have many attachments

#### `downloadResources` (Boolean)
- **Default**: `false`
- Downloads image and document files from the lesson resources panel
- Web links (YouTube, Chrome extensions, etc.) are automatically skipped — only actual files are downloaded
- Each resource gets a `kvKey` field when downloaded
- ⚠️ Only useful if the community shares PDF guides or image files in lesson resources

---

## 📊 Sample Output

### Full lesson row (`mode: all`, `mode: course`, `mode: lesson`)
```json
{
  "extractedAt": "2026-02-22T15:30:38.866Z",
  "community": "my-community",
  "communityUrl": "https://www.skool.com/my-community",
  "course": "Course Title",
  "courseShortId": "38fd1c1d",
  "section": "Section Name",
  "lesson": "Lesson 01",
  "lessonId": "2a25cd8ad2664da78894b8290dd25b43",
  "lessonUrl": "https://www.skool.com/my-community/classroom/38fd1c1d?md=2a25cd8...",
  "position": 12,
  "descriptionText": "Lesson 01 - Introduction\nThis lesson covers...",
  "hostedVideo": {
    "source": "mux",
    "playbackUrl": "https://stream.mux.com/PLAYBACK_ID.m3u8?token=JWT",
    "durationMs": 5143300,
    "thumbnailUrl": "https://assets.skool.com/...",
    "aspectRatio": "16:9"
  },
  "resources": [
    {
      "title": "Lesson on YouTube",
      "url": "https://www.youtube.com/watch?v=...",
      "platform": "youtube",
      "fileType": null,
      "kvKey": null
    },
    {
      "title": "PDF Guide",
      "url": "https://assets.skool.com/.../guide.pdf",
      "platform": "other",
      "fileType": "doc",
      "kvKey": "doc-2a25cd8ad266-r1.pdf"
    }
  ],
  "post": {
    "postId": "5b00705d9cf54ae7ae9273a64cffb9b7",
    "postName": "01",
    "postUrl": "https://www.skool.com/my-community/post/01",
    "title": "Lesson 01 - Introduction",
    "author": { "firstName": "John", "lastName": "Doe", "fullName": "John Doe" },
    "contributors": [],
    "content": "Watch the lesson here:\n[https://youtube.com/...](https://youtube.com/...)",
    "links": [{ "url": "https://youtube.com/...", "platform": "youtube" }],
    "attachments": [
      {
        "id": "6fe0d721...",
        "fileName": "handout.png",
        "contentType": "image/png",
        "url": "https://assets.skool.com/...",
        "fileSizeBytes": 6033806,
        "fileType": "image",
        "kvKey": "image-6fe0d721....png"
      }
    ],
    "videos": [
      {
        "source": "mux",
        "playbackUrl": "https://stream.mux.com/....m3u8?token=JWT",
        "durationMs": 1220880,
        "thumbnailUrl": "https://assets.skool.com/..."
      }
    ],
    "poll": {
      "entries": [
        { "text": "Yes, very clear", "count": 10 },
        { "text": "Somewhat clear", "count": 1 }
      ]
    },
    "upvotes": 15,
    "commentsCount": 7
  }
}
```

### Post-only output (`mode: post`)
Returns just the `post` object above — no lesson or course wrapper.

---

## 💾 Key-Value Store

When `downloadAttachments` or `downloadResources` is enabled, downloaded files are saved to the actor's Key-Value Store organized by file type:

| Key prefix | Contents |
|---|---|
| `image-{id}.{ext}` | JPEG, PNG, GIF, WebP, SVG, BMP, TIFF |
| `doc-{id}.{ext}` | PDF, Word, Excel, CSV, TXT, JSON, XML |

Each downloaded file is referenced by `kvKey` on its parent attachment or resource object in the dataset, so you can always trace a file back to its lesson.

> **Note**: Apify Key-Value Store keys are limited to **63 characters**. All keys generated by this actor stay well within this limit.

---

## 🎬 Downloading Videos (Manual / Future Feature)

The actor extracts `hostedVideo.playbackUrl` and `post.videos[].playbackUrl` for every lesson.

> 🔮 **Automatic video download within the actor is planned for a future release.**

---

## 🛠️ Troubleshooting

### No data extracted / empty dataset
- Community may require login — provide your email + password or browser cookies
- URL format may be wrong — check the [URL Modes](#-url-modes--four-ways-to-scope-your-extraction) table
- Community slug is case-sensitive: `my-community` not `My-Community`

### `descriptionText` is empty
- Some lessons genuinely have no description — this is normal
- If you expect a description, verify it's visible on the Skool lesson page when logged in

### Downloads fail with 403
- Assets hosted on external CDNs (e.g. giphy.com) may block direct downloads — these are skipped automatically
- Skool assets (`assets.skool.com`) should always work; if they fail, try providing credentials or refreshing your cookies

### Locked courses skipped
- Provide email + password, or browser cookies from an account with access to those courses
- If using cookies, make sure they haven't expired — export fresh ones and re-run
- Only courses where `hasAccess: 1` are extracted

### Login fails with correct credentials
- Double-check there are no extra spaces in the email or password fields
- Try the browser cookies method as a fallback

### Rate limiting / Cloudflare blocks
- The actor uses a built-in request delay between API calls to avoid triggering rate limits
- If you still see blocks, try providing fresh browser cookies or re-running after a few minutes

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

### 🎬 Video & Media
- **[Loom Scraper](https://apify.com/dz_omar/loom-video-scraper?fpr=smcx63)** Loom video & transcript extraction
- **[YouTube Scraper Pro](https://apify.com/dz_omar/Youtube-Scraper-Pro?fpr=smcx63)** Complete channel and playlist extraction
- **[Zoom Scraper](https://apify.com/dz_omar/zoom-scraper?fpr=smcx63)** Download Zoom recordings and transcripts
- **[TikTok Video Downloader](https://apify.com/dz_omar/tiktok-video-downloader?fpr=smcx63)** TikTok without watermarks

### 🛠️ Developer Tools
- **[Universal Downloader](https://apify.com/dz_omar/universal-downloader?fpr=smcx63)** Download any file type with proxy support
- **[Ultimate Screenshot](https://apify.com/dz_omar/ultimate-screenshot?fpr=smcx63)** Advanced website screenshot tool
- **[n8n Workflow Server](https://apify.com/dz_omar/n8n-workflow-server?fpr=smcx63)** Run n8n automation on Apify

### 📱 Social & Ads
- **[Facebook Ads Scraper Pro](https://apify.com/dz_omar/facebook-ads-scraper-pro?fpr=smcx63)** Facebook Ad Library extraction
- **[Google Ads Scraper](https://apify.com/dz_omar/google-ads-scraper?fpr=smcx63)** Google Ads Transparency Center data

---
