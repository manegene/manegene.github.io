# manegene.github.io redesign

Copy these into the root of your manegene.github.io repo, replacing the old ones, then push.

| File | What it is |
|---|---|
| index.html | The new site (replaces the old one) |
| 404.html | GitHub Pages shows this for broken links |
| robots.txt | Lets search engines crawl everything except the /Builds zip files, and points them to the sitemap |
| sitemap.xml | Lists the page for Google and Bing |
| images/robert-manegene.jpg | Profile photo, cropped and resized (53 KB instead of 650 KB) |
| images/og-card.png | The picture shown when the link is shared on LinkedIn, WhatsApp, etc. |

Leave favicon.ico, Builds/ and images/preview.jpg where they are (preview.jpg is no longer used; delete it if you like).

## After you push
1. Google Search Console → add property `https://manegene.github.io/` → verify with the HTML tag method (add the meta tag to index.html, near the other meta tags).
2. Sitemaps → submit `sitemap.xml`. URL Inspection → `https://manegene.github.io/` → Request indexing.
3. Do the same in Bing Webmaster Tools (it can import from Search Console).
4. Each time you change the site, update `<lastmod>` in sitemap.xml.
5. Check the share preview at https://www.linkedin.com/post-inspector/
