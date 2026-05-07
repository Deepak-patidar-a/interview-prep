# 📸 HLD — Photo Sharing App (Instagram)

> **Series**: Frontend System Design — HLD Case Studies  
> **System**: Photo Sharing App (Instagram-like)  
> **Framework used**: [HLD Introduction](./hld-introduction.md)

---

## Table of Contents

1. [Functional Requirements](#1-functional-requirements)
2. [Non-Functional Requirements](#2-non-functional-requirements)
3. [Architectural Design](#3-architectural-design)
4. [Component Architecture](#4-component-architecture)
5. [Data Models](#5-data-models)
6. [API Design](#6-api-design)
7. [Optimization Strategies](#7-optimization-strategies)
8. [Implementation Details](#8-implementation-details)

---

## 1. Functional Requirements

These are the **features** the system must support, organised by module.

```
Photo Sharing App — Feature Tree

Feed Management
  List (infinite scroll of posts)
  Create Post
    Upload (single / multi image, video)
    Edit (crop, resize, filters)
    Caption, Tags, Location

Reels
  List (vertical scroll, auto-play)
  Create Reel (record / upload video)

Stories
  List (top bar — avatars, auto-advance)
  Create Story (image / video, 24hr expiry)

Comments & Likes
  Like / unlike a post
  Comment on a post
  Reply to a comment

Browse / Explore
  Search (users, hashtags, places)
  Trending / recommended content

Messages (DMs)
  Conversation list
  Message thread
  Share post / reel via DM

Account Management
  Register / Login / Logout
  Auth (OAuth, email/password)
  Settings (privacy, notifications)

Profile Management
  Profile page (bio, photo grid)
  Edit profile
  Follow / Unfollow
```

### Scoping (Prioritisation for this HLD)

For this HLD we focus deeply on the **Feed** module — the most technically complex:
- Feed listing (rendering performance, infinite scroll, SSR)
- Post creation (upload, editing, filters)
- Reactions (likes, comments — optimistic updates)

Other modules (Reels, Stories, DMs) follow similar patterns.

---

## 2. Non-Functional Requirements

| Category | Requirement | Notes |
|----------|-------------|-------|
| **Device Support** | iOS, Android, Desktop web | Responsive layout, touch events |
| **Security** | HTTPS only, CSP headers | XSS, CSRF protection |
| **Auth** | JWT / OAuth 2.0 | Token refresh, session management |
| **SEO** | Public profiles, explore pages | SSR for crawlable content |
| **Performance** | FCP < 1.8s, LCP < 2.5s, TTI < 3.8s | Feed must feel instant |
| **Accessibility** | WCAG 2.1 AA | Alt text on images, keyboard nav |
| **Offline Support** | View cached feed | Service Worker + IndexedDB |
| **Testing** | Unit, integration, E2E | Jest, Testing Library, Playwright |
| **i18n / l10n** | Multi-language, RTL support | react-intl or i18next |
| **Deployment** | CI/CD pipeline, CDN | Vercel / CloudFront + S3 |

---

## 3. Architectural Design

The architecture follows a **layered MVC-like pattern** on the client side, with an API Gateway as the single entry point to server services.

```
CLIENT SIDE
───────────────────────────────────────────────────────────────────────
  VIEW                  CONTROLLER              STORAGE
  ────────────────      ──────────────────      ──────────────────────
  Feed Listing          Filters                 Redux (UI state)
  Post Creation         Editing                 Apollo Cache (GraphQL)
  Stories UI            Upload logic            LocalStorage (tokens)
  Reels Player          Post listing            IndexedDB (offline feed)
                        Post creation

                  SERVICE LAYER
                  ──────────────────────────────
                  Abstracts all API calls
                  - Upload service
                  - Post API service
                  - Feed API service
───────────────────────────────────────────────────────────────────────
                         │ HTTP / WebSocket
                         ▼
SERVER SIDE
───────────────────────────────────────────────────────────────────────
                    API GATEWAY
                    ──────────────────────────────
                    Single entry point. Routes and
                    authenticates all requests.

                    Routes:
                    /auth      → Auth service
                    /upload    → Media service
                    /posts     → Post service
                    /feed      → Feed service
───────────────────────────────────────────────────────────────────────
```

### Storage Layer — When to Use What

| Storage | Use Case | Persistence |
|---------|----------|-------------|
| **Redux** | UI state, feed data in memory, modal state | Session only |
| **Apollo Cache** | GraphQL query results, normalized entity store | Session only |
| **LocalStorage** | Auth tokens, user preferences, theme | Permanent (5MB limit) |
| **IndexedDB** | Cached feed posts for offline, large media blobs | Permanent (large capacity) |

---

## 4. Component Architecture

### Feed List — Component Hierarchy

```
FeedPage                             Page-level, handles data fetching
  └── FeedList                       Manages infinite scroll, virtualization
        └── FeedItem                 One post (repeated N times)
              ├── UserDetails        Avatar + username + follow button
              │    ├── Avatar
              │    └── Username
              │
              ├── PostBody           The image(s) or video
              │    ├── MediaCarousel (multi-image posts)
              │    └── VideoPlayer   (video posts — lazy loaded)
              │
              ├── Caption            Post text, hashtags, "see more"
              │
              └── Reaction           Like, comment, share, bookmark
                   ├── LikeButton        (optimistic update)
                   ├── CommentButton
                   ├── ShareButton
                   └── BookmarkButton
```

### Post/Feed Creation — Component Hierarchy

```
CreatePostPage
  ├── MediaUploader                  Drag & drop / file picker
  │    ├── DropZone
  │    └── FileInput (multiple)
  │
  ├── MediaEditor                    Shows after upload
  │    ├── CropTool                  (Canvas API — crop + resize)
  │    ├── FilterPanel               ← REUSABLE in Stories & Reels
  │    │    └── FilterPreview x N    (CSS filters previewed on canvas)
  │    └── AdjustmentPanel           (brightness, contrast, saturation)
  │
  ├── PostMetadata
  │    ├── CaptionInput              (hashtag + @mention parsing)
  │    ├── TagPeople                 ← REUSABLE in Stories & Reels
  │    ├── AddLocation
  │    └── AccessibilityAltText
  │
  └── UploadProgress                 Chunked upload progress bar
```

### Reusable Components (cross-feature)

| Component | Used in |
|-----------|---------|
| `FilterPanel` | Post creation, Story creation, Reel creation |
| `TagPeople` | Post creation, Story creation |
| `CaptionInput` | Post creation, Story creation, Reel creation |
| `Avatar` | Feed, Stories bar, Comments, DMs, Profile |
| `MediaCarousel` | Feed, Profile grid, Explore |

---

## 5. Data Models

```typescript
// Post — a single image/video post
interface Post {
  id: string;
  caption: string;
  createdAt: string;         // ISO 8601
  updatedAt: string;
  userId: string;            // reference to User
  mediaIds: string[];        // ordered — reference to Media[]
  location?: string;
  taggedUserIds?: string[];
  likesCount: number;
  commentsCount: number;
  isLikedByMe: boolean;      // personalised field for optimistic updates
}

// FeedList — paginated list of posts
interface FeedList {
  postIds: string[];
  totalPosts: number;
  currentPageNumber: number;
  currentPageSize: number;
  nextCursor?: string;       // cursor-based pagination (better than offset)
}

// User — minimal info needed for rendering feed items
interface User {
  id: string;
  username: string;
  fullName: string;
  profilePhotoUrl: string;
  isFollowedByMe: boolean;
}

// Media — image or video asset
interface Media {
  id: string;
  type: 'image' | 'video';
  url: string;
  thumbnailUrl?: string;     // for video posts — poster frame
  width: number;
  height: number;
  aspectRatio: number;       // precomputed — prevents CLS (layout shift)
  altText?: string;          // accessibility
  duration?: number;         // video length in seconds
}

// Comment
interface Comment {
  id: string;
  postId: string;
  userId: string;
  text: string;
  createdAt: string;
  likesCount: number;
  replies?: Comment[];       // nested, one level deep
}
```

> **Why `aspectRatio` on Media?**  
> If the browser doesn't know image dimensions before loading, it collapses the space → layout shifts → high CLS score. Sending `width`, `height`, and `aspectRatio` from the API lets the client reserve exact space before the image arrives.

> **Why cursor-based pagination for feed?**  
> With offset pagination (`page=2`), if a new post is inserted while you're scrolling, you'll see duplicates or skip posts. Cursor-based (`after=postId_xyz`) always picks up exactly where you left off, even as new content arrives.

---

## 6. API Design

### Feed List

```
GET /api/feed?cursor=<postId>&pageSize=20

Headers:
  Authorization: Bearer <jwt_token>

Response 200:
{
  "data": {
    "feeds": [
      {
        "id": "post_abc123",
        "caption": "Golden hour in Mumbai",
        "createdAt": "2026-05-01T18:30:00Z",
        "userId": "user_xyz",
        "mediaIds": ["media_001", "media_002"],
        "likesCount": 1243,
        "commentsCount": 87,
        "isLikedByMe": false
      }
    ],
    "totalPosts": 2840,
    "currentPageSize": 20,
    "nextCursor": "post_def456"
  },
  "error": null
}

Response 401: { "error": { "code": "UNAUTHORIZED", "message": "Token expired" } }
Response 500: { "error": { "code": "SERVER_ERROR", "message": "..." } }
```

### Create Post — Two Strategies

#### Strategy 1 — Single API (simpler, not recommended for production)

```
POST /api/posts
Content-Type: multipart/form-data

Body:
  caption       (string)
  location      (string, optional)
  media[]       (File[], max 10 files, 50MB each)
  taggedUsers[] (string[], optional)
```

**Problem**: Large file + post data in one request. No progress tracking. Full retry on any failure.

#### Strategy 2 — Separate APIs (recommended)

**Step 1: Upload media**
```
POST /api/media/upload
Content-Type: multipart/form-data

Body: file, type ('image' | 'video')

Response 200:
{
  "data": {
    "mediaId": "media_abc123",
    "uploadUrl": "https://cdn.example.com/media/abc123.webp"
  }
}
```

**Step 2: Create post with media IDs**
```
POST /api/posts
Content-Type: application/json

Body:
{
  "caption": "Golden hour in Mumbai",
  "mediaIds": ["media_abc123", "media_def456"],
  "location": "Mumbai, India",
  "taggedUserIds": ["user_001"]
}

Response 201: { "data": { "post": { ...Post } }, "error": null }
```

**Why separate?** Upload shows progress independently. Post creation can retry without re-uploading. Supports resumable chunked uploads for large videos.

### Like / Unlike

```
POST   /api/posts/:postId/like    ← like
DELETE /api/posts/:postId/like    ← unlike

Response 200:
{
  "data": {
    "postId": "post_abc123",
    "likesCount": 1244,
    "isLikedByMe": true
  }
}
```

---

## 7. Optimization Strategies

### Asset Optimization — Images

```
1. Modern formats (WebP by default, AVIF where supported)
   <picture>
     <source srcset="post.avif" type="image/avif" />
     <source srcset="post.webp" type="image/webp" />
     <img src="post.jpg" alt="Sunset photo" />
   </picture>

2. srcset — serve the right resolution per screen size
   <img
     srcset="post-400.webp 400w, post-800.webp 800w, post-1200.webp 1200w"
     sizes="(max-width: 600px) 100vw, 600px"
     src="post-800.webp"
   />

3. Device Pixel Ratio (DPR)
   Retina screens (DPR=2) need 2x pixels for sharp images.
   The browser picks the right srcset entry automatically.

4. Network-aware quality
   if (navigator.connection?.effectiveType === '2g') {
     // Request lower-quality image URLs from API
   }

5. Prefetch next images
   When user is on post 5, prefetch post 6/7 images in the background.
```

### Feed Optimization — Rendering Pipeline

```
Step 1 — SSR for Above The Fold (first 2-3 posts)
  Server renders initial posts → user sees content immediately
  No white screen → better LCP

Step 2 — Lazy load below-fold images
  <img loading="lazy" src="post.webp" />
  Native browser API, zero JS needed

Step 3 — Infinite scroll via Intersection Observer
  Observe a sentinel <div> at bottom of list
  When it enters viewport → fetchNextPage()
  rootMargin: '200px'  ← start loading before user hits bottom

Step 4 — Virtualization (windowing)
  Render only posts in/near viewport
  Off-screen posts removed from DOM, scroll position maintained
  Prevents DOM from growing to thousands of nodes
  Libraries: react-virtual, react-window

Step 5 — Code splitting
  Heavy components loaded on demand only:
  - CropTool, FilterPanel → lazy loaded when user clicks "Create"
  - VideoPlayer → lazy loaded only for video posts

Step 6 — Loading shimmers
  While posts load → animated grey placeholder cards
  User perceives faster load time

Step 7 — Scroll position preservation
  User opens a post → scrollY saved to sessionStorage
  User presses back → feed restores exact scroll position

Step 8 — Web Workers
  Heavy tasks off the main thread:
  - Image compression before upload
  - IndexedDB reads/writes for offline cache

Step 9 — Optimistic updates
  User taps like icon:
  1. Immediately update UI (isLikedByMe=true, likesCount++)
  2. Fire API call in background
  3. If API fails → revert UI to previous state
  Result: zero perceived latency
```

### Summary Table

| Technique | Problem solved | Applied where |
|-----------|---------------|---------------|
| WebP / AVIF | Oversized image files | All images |
| srcset + sizes | Serving wrong resolution | All images |
| SSR for ATF | White screen on initial load | First 2-3 feed posts |
| Native lazy loading | Loading off-screen images early | Below-fold posts |
| Intersection Observer | Polling scroll events (CPU heavy) | Infinite scroll |
| Virtualization | DOM growing unbounded | Feed list |
| Shimmers | Blank loading states | All async content |
| Code splitting | Large initial JS bundle | Editors, video player |
| Scroll preservation | Feed resets on back navigation | FeedPage ↔ PostPage |
| Web Workers | Main thread jank during heavy tasks | Image compression |
| Optimistic updates | API latency felt by user | Like, bookmark, follow |

---

## 8. Implementation Details

### Image Editing — Canvas API

```javascript
// Crop using the Canvas API (no library needed)
function cropImage(imageUrl, cropParams) {
  return new Promise((resolve) => {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    const img = new Image();

    img.onload = () => {
      const { x, y, width, height } = cropParams;

      canvas.width = width;
      canvas.height = height;

      // Draw only the cropped region onto the canvas
      ctx.drawImage(img, x, y, width, height, 0, 0, width, height);

      // Export as WebP blob (quality 0.85)
      canvas.toBlob(resolve, 'image/webp', 0.85);
    };

    img.src = imageUrl;
  });
}
```

### CSS Filters — Instagram-style

```javascript
const FILTERS = {
  'Clarendon': 'contrast(1.2) saturate(1.35)',
  'Gingham':   'brightness(1.05) hue-rotate(350deg)',
  'Moon':      'grayscale(1) contrast(1.1) brightness(1.1)',
  'Lark':      'contrast(0.9) brightness(1.1) saturate(1.2)',
  'Reyes':     'sepia(0.22) brightness(1.1) contrast(0.85) saturate(0.75)',
  'Juno':      'sepia(0.35) contrast(1.15) brightness(1.15) saturate(1.8)',
  'Slumber':   'saturate(0.66) brightness(1.05)',
  'Original':  'none',
};

// Apply to img element for preview
function applyFilter(imgElement, filterName) {
  imgElement.style.filter = FILTERS[filterName] || 'none';
}

// Bake filter into canvas before upload (so server stores the filtered version)
function applyFilterToCanvas(sourceCanvas, filterName) {
  const output = document.createElement('canvas');
  output.width = sourceCanvas.width;
  output.height = sourceCanvas.height;
  const ctx = output.getContext('2d');
  ctx.filter = FILTERS[filterName] || 'none';
  ctx.drawImage(sourceCanvas, 0, 0);
  return output;
}
```

### File Upload Strategies

```javascript
// Strategy 1: Simple multipart/form-data (for images < 5MB)
async function uploadSimple(file) {
  const formData = new FormData();
  formData.append('file', file);
  // Don't set Content-Type — browser sets it with the boundary automatically
  const res = await fetch('/api/media/upload', { method: 'POST', body: formData });
  return res.json();
}

// Strategy 2: Chunked / Resumable upload (for large videos)
async function uploadChunked(file, onProgress) {
  const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB
  const totalChunks = Math.ceil(file.size / CHUNK_SIZE);

  // Step 1: Initiate upload session
  const { uploadId } = await fetch('/api/media/upload/initiate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ filename: file.name, size: file.size }),
  }).then(r => r.json());

  // Step 2: Upload each chunk sequentially
  for (let i = 0; i < totalChunks; i++) {
    const chunk = file.slice(i * CHUNK_SIZE, (i + 1) * CHUNK_SIZE);
    const data = new FormData();
    data.append('chunk', chunk);
    data.append('uploadId', uploadId);
    data.append('chunkIndex', String(i));
    data.append('totalChunks', String(totalChunks));

    await fetch('/api/media/upload/chunk', { method: 'POST', body: data });
    onProgress(Math.round(((i + 1) / totalChunks) * 100));
  }

  // Step 3: Finalise — server assembles chunks
  return fetch('/api/media/upload/complete', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ uploadId }),
  }).then(r => r.json()); // returns { mediaId, url }
}
```

**Strategy comparison:**

| Strategy | Best For | Pros | Cons |
|----------|----------|------|------|
| Simple POST | Images < 5MB | Simple | No progress, full retry on failure |
| Base64 | Tiny icons only | Works in JSON | 33% larger payload |
| Chunked / Resumable | Large videos | Progress bar, resume on failure | More complex server |

### Optimistic Update — Like Button

```javascript
function useLikePost(postId) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (isLiked) =>
      isLiked
        ? fetch(`/api/posts/${postId}/like`, { method: 'DELETE' })
        : fetch(`/api/posts/${postId}/like`, { method: 'POST' }),

    // 1. Update cache immediately before API responds
    onMutate: async (isCurrentlyLiked) => {
      await queryClient.cancelQueries(['feed']);
      const previousFeed = queryClient.getQueryData(['feed']);

      queryClient.setQueryData(['feed'], (old) => ({
        ...old,
        posts: old.posts.map(p =>
          p.id !== postId ? p : {
            ...p,
            isLikedByMe: !isCurrentlyLiked,
            likesCount: isCurrentlyLiked ? p.likesCount - 1 : p.likesCount + 1,
          }
        ),
      }));

      return { previousFeed };
    },

    // 2. Revert if API fails
    onError: (err, variables, context) => {
      queryClient.setQueryData(['feed'], context.previousFeed);
    },
  });
}
```

### Infinite Scroll — Intersection Observer

```javascript
function useInfiniteScroll(fetchNextPage, hasNextPage) {
  const sentinelRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasNextPage) {
          fetchNextPage();
        }
      },
      { rootMargin: '200px' } // start fetching 200px before bottom
    );

    if (sentinelRef.current) observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [fetchNextPage, hasNextPage]);

  return sentinelRef;
}

// In the feed list component:
function FeedList() {
  const { data, fetchNextPage, hasNextPage } = useInfiniteQuery(/* ... */);
  const sentinelRef = useInfiniteScroll(fetchNextPage, hasNextPage);

  return (
    <div>
      {data.pages.flatMap(page => page.feeds).map(post => (
        <FeedItem key={post.id} post={post} />
      ))}
      <div ref={sentinelRef} style={{ height: 1 }} />  {/* sentinel */}
    </div>
  );
}
```

---

## Key Design Decisions — Quick Reference

| Decision | Choice | Reason |
|----------|--------|--------|
| Rendering | SSR for ATF + CSR hydration | Feed SEO + rich interactivity |
| Pagination | Cursor-based | Live feed — offset causes duplicates |
| Like update | Optimistic | Instant feedback, revert on failure |
| Image format | WebP + AVIF + JPG fallback | Best compression, wide support |
| Upload strategy | Separate `/upload` then `/createPost` | Resumable, progress, retry-safe |
| Filters | CSS `filter` property | GPU-accelerated, no extra library |
| Crop / resize | Canvas API | Native browser, no server round-trip |
| Offline | Service Worker + IndexedDB | Cache feed for offline viewing |
| State management | Redux (UI) + React Query (server) | Separate server/client concerns |
| Scroll | Intersection Observer | Low CPU vs scroll event listeners |
| Virtualization | react-virtual | DOM stays constant at any scroll depth |

---

*Next HLD: Real-time Chat Application*  
*Back to: [HLD Introduction](./hld-introduction.md)*
