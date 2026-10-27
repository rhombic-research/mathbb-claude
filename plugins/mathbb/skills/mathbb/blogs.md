# Blogs

Each MathBB user can have one **blog**: posts written in MathBB's markdown +
LaTeX (the page syntax of the core guide), an About page, comments and an Atom
feed, at `/blogs/<username>/<slug>`. The user runs it in the app at `/app/blog`.

A blog is stored as a notebook: its posts are the notebook's pages and its
images are the notebook's resources. So the blog tools handle what a notebook
lacks (the blog's settings, each post's settings, publishing), and a post's
text, title and images are changed with the ordinary notebook tools on the
blog's `notebook_id`, which `get_blog` gives.

## The tools

- `get_blog`: the blog as JSON (settings, whether it is published, stats,
  `notebook_id`, `about_page_id`, and every post with its `page_id`, `slug`,
  status, date and address). Read it first.
- `create_blog`: start the blog (title, address `slug`, tagline). It starts
  **not published**.
- `update_blog`: its settings (title, slug, tagline, description, author_name,
  listed, comments_enabled, comment_moderation, posts_per_page, feed_enabled).
- `set_blog_published`: publish the blog, or take it offline again. Only when
  the user asks: a published blog is public.
- `create_blog_post`: a new **draft** with its title, full text and settings.
- `get_blog_post`, `update_blog_post`: one post's settings (slug, excerpt,
  tags, cover_image, comments_enabled, pinned, published_at).
- `publish_blog_post`, `unpublish_blog_post`.
- The notebook tools, with the blog's `notebook_id` and a post's `page_id`:
  `read_page`, `edit_page` (the text), `rename_page` (the title),
  `upload_resource` (an image), `create_page` (also makes a draft post).
- **Browser only**: deleting the blog, moderating comments (approve, hide,
  delete), and connecting the blog to a domain of the user's own (Blog
  settings, Custom domain). Tell the user rather than working around it.

## Publishing

- A post is a **draft** until `publish_blog_post`; only the user sees drafts.
- Publishing takes a snapshot of the post's title and text. Edits made after
  that (with `edit_page` or in the editor) stay private until the post is
  published again: `has_unpublished_changes` says when that is the case.
- `published_at` is the post's date. A future date **schedules** the post; a
  past one backdates it. Without one, the post keeps the date it has, or gets
  today's.
- Readers see a post only while the **blog** is published too. A user can write
  and publish posts before publishing the blog; they all go live with it.
- The **About page** (`about_page_id`) is published like a post but has no
  date, tags or comments, and can't be deleted.
- Publish, schedule or unpublish only when the user asks.

## Images

Upload with `upload_resource` on the blog's `notebook_id`, then put the
filename the upload reports in the post: `![A caption](filename)`. An image is
public only once a live post (or the blog's description) uses it. A post's
`cover_image` is a filename from the same media.

## Post addresses and links between posts

A post's address is `/blogs/<username>/<blog slug>/<post slug>` (and
`/<post slug>` on the blog's custom domain, when it has one): `slug` and
`address` in `get_blog` and `get_blog_post`.

- A post gets its `slug` from its title when it is **first published**
  (lowercase words joined by hyphens), unless one was set before. After that
  it stays: retitling the post doesn't move it. A draft that has none yet
  shows the one it would get as `suggested_slug`.
- Set or change it with the `slug` setting of `create_blog_post` or
  `update_blog_post`: lowercase letters, digits and hyphens, up to 200
  characters, not only digits and not a reserved word (`about`, `feed`,
  `media`, …). Changing a published post's slug keeps the old address
  redirecting to the new one.
- Link from one post to another with the path `/blogs/<username>/<blog
  slug>/<post slug>`, even when the blog has a custom domain: it redirects
  there, and keeps working if the domain is disconnected.

## Importing a blog from elsewhere

When the user asks to bring posts over from another platform:

1. Get each post's text as markdown, its title, original date, tags and images.
   Convert the platform's math and embeds to MathBB's syntax (core guide).
2. `create_blog_post` for each post, in date order, with its title, text, tags,
   `published_at` set to the original date, and `slug` set to the post's slug
   on the old platform (the last part of its old address, e.g.
   `the-feedback-loop-of-mathematics-research`), so that old links keep working
   once the user points their domain at MathBB. Upload the images first and
   use their MathBB filenames.
3. Once every post exists, rewrite links between posts to their MathBB
   addresses (`edit_page`), then `publish_blog_post` each one with its original
   `published_at`, and `updated_at` if the original shows when it was last
   revised.
4. Check a few posts in the browser before the user publishes the blog.

`update_blog_post` also takes `view_count`, for importing a post's view count
from the old platform, but only MathBB administrators may set it: it is
refused for everyone else, and shouldn't be offered.
