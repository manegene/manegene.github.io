# manegene.github.io: blog setup

GitHub Pages builds this site with Jekyll on every push. Nothing to install.

## Write a new post
1. Add a file to `_posts/` named `YYYY-MM-DD-short-title.md` (the date must not be in the future, or it won't show).
2. Start it with:
   ```
   ---
   title: "The question people search for, as a title"
   description: "One or two sentences: what the reader will be able to do. Shown on Google and on the blog list."
   tags: [M-Pesa, nopCommerce]
   ---
   ```
3. Write the post in Markdown below that (## headings, lists, `code`, tables, images in /images/).
4. Push. The post appears at /blog/short-title/, on /blog/, on the homepage ("Guides from real projects"),
   in /sitemap.xml and in /blog/feed.xml automatically.
5. Google Search Console → URL Inspection → paste the post's address → Request indexing. Share it once on LinkedIn.

## Files
| File | What it does |
|---|---|
| `_config.yml` | Site name, address, author and the sitemap / feed / SEO plugins |
| `_layouts/post.html` | How a post looks |
| `blog/index.html` | The list of posts |
| `assets/blog.css` | Blog styles (same colours as the homepage) |
| `index.html` | The homepage; its "Guides from real projects" section lists the latest 3 posts |

`sitemap.xml` is now made by Jekyll (delete the old hand-written one). `robots.txt` stays as it is.
