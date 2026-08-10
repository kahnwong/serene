This is the detailed guide on how to use zola-theme-serene. You should also check zola's [documentation](https://www.getzola.org/documentation/getting-started/overview/).

Serene requires zola `0.23.2` or later.

## Installation

Create a zola site and add serene theme (assuming your site is called `myblog`):

```sh
zola init myblog
cd myblog
git init
git submodule add -b latest https://github.com/isunjn/serene.git themes/serene
```

Copy the content of `myblog/themes/serene/zola.toml.example` to `myblog/zola.toml`.

## Sections and Pages

There is a `nav` config option in your `zola.toml`, which enumerates the navigation entries of the home page. An entry whose `path` starts with `/` links to a section of your site, any other path (e.g. an `https://` URL) is treated as an external link and opens in a new tab. You should have at least one `blog` section.

The name and path can be changed, be noticed that if you changed the blog section path (e.g. from `/posts` to `/blog`), then you should also change `blog_section_path` option.

For home page, create `myblog/content/_index.md`:

```
+++
template = 'home.html'

[extra]
lang = 'en'

# Show footer in home page
footer = false

# Show a few recent posts in home page
recent = false
recent_max = 15
recent_more_text = "more »"
+++

Hi, I'm ...
```

For blog section, create `myblog/content/posts/_index.md`:

```
+++
title = "My Blog"
description = "My blog site."
sort_by = "date"
template = "posts.html"
page_template = "post.html"
insert_anchor_links = "right"
generate_feeds = true

[extra]
lang = "en"

title = "Posts"
subtitle = "I write about ...."

categorized = false # posts can be categorized
back_to_top = true # show back-to-top button
+++
```

Display options like `toc` / `code_copy` / `comment` / `date_format` have site-wide defaults in `zola.toml`, you can override them here for this section (e.g. `toc = false`), see [Front Matter](#front-matter) for details.

Blog section is defined by `posts.html` and `post.html`. Serene also has a special template called `prose.html`, it applies the same styles of blog post page. You can use it as a section template for a custom section page, for example if you want a separate `about` page, you can add a `{ name = "about", path = "/about" }` to the `nav` and create a `myblog/content/about/_index.md`:

```
+++
title = "About me"
description = "About page of ..."
template = "prose.html"
insert_anchor_links = "none"

[extra]
lang = 'en'

title = "About"
subtitle = "About this site"
+++

Hi, My name is ....
```

The default date format is "%b %-d, %Y", e.g. "Dec 13, 2025", check [this page](https://docs.rs/jiff/latest/jiff/fmt/strtime/index.html) if you want to customize it, for example change to "%Y-%-m-%-d", e.g. "2025-2-13".

### Multiple list sections

You can have more than one blog-like list section, e.g. a `series` section alongside `posts`. Just create `myblog/content/series/_index.md` the same way as the blog section (with `template = "posts.html"` and `page_template = "post.html"`), and add it to `nav` in `zola.toml`. Each list section has its own options in `[extra]` (`date_format`, `toc`, `categorized`, etc.), and can have its own feed by setting `generate_feeds = true` in its `_index.md` (available at e.g. `/series/feed.xml`).

The `blog_section_path` option in `zola.toml` points to your *main* blog section, it is used by the recent posts list of the home page and by the tags pages (tags are site-wide: posts from all list sections that share a tag will be listed together).

Now the myblog directory may look like this:

```
├── zola.toml
├── content/
│   ├── posts/
│   │   └── _index.md
│   ├── about/
│   │   └── _index.md
│   └── _index.md
├── sass/
├── static/
├── templates/
└── themes/
    └── serene/
```

## Favicon

Create a new directory `img` under `myblog/static`, put favicon related files here, you can use tools like [favicon.io](https://favicon.io/favicon-converter/) to generate those files. If you want to display avatar in home page, also put your avatar picture file `avatar.webp` here, webp format is recommended.

```
...
├── static/
│   └── img/
│       ├── favicon-16x16.png
│       ├── favicon-32x32.png
│       ├── apple-touch-icon.png
│       └── avatar.webp
...
```

## Icon

The default icons are placed in `myblog/themes/serene/static/icon`, the `icon` value in `links` of `zola.toml` is the file name of the svg file.

To customize, find the svg file you want, modify (in case you don't kown, a svg file is just a plian text file) its width and height to `18`, and the color to `currentColor`:

`... width="18" height="18" ... fill="currentColor" ...`

and then put it in `myblog/static/icon`, file in this folder with the same name will override the default one.

The default icons mostly came from [Remix Icon](https://remixicon.com/).

## Theme

The `color_scheme` option in `zola.toml` controls the light/dark mode behavior: `"auto"` (default) follows the visitor's system preference and shows a theme toggle button, while `"light"` / `"dark"` locks the site to a single mode and hides the button.

## RSS

There are two ways to provide feeds:

- **Per-section feeds** (recommended): set `generate_feeds = false` in `zola.toml`, and `generate_feeds = true` in the `_index.md` of your list sections. Each of these sections gets its own feed (e.g. `/posts/feed.xml`, `/series/feed.xml`), using the `title` and `description` of that section. The RSS button in the footer links to the feed of the section the current page belongs to.
- **A single site-wide feed**: set `generate_feeds = true` in `zola.toml`, and `generate_feeds = false` in section `_index.md` files. The feed is located in the root directory (e.g. `/feed.xml`), contains posts from all sections, and uses the `title` and `description` of `zola.toml`. The RSS button in the footer links to it on all pages.

`feed_filenames` can be set to `["feed.xml"]` (serene's own atom template), or `["atom.xml"]` / `["rss.xml"]` (zola's built-in templates), corresponding to different feed standards.

## Open Graph

Each page has [Open Graph](https://ogp.me/) and Twitter Card meta tags (plus a canonical link), so links shared to social media and chat apps can show a rich preview card with title, description and image.

Title and description come from the same sources as the page's `<title>` and meta description. The preview image is resolved as follows:

- Post pages use `og_image` in the `[extra]` section of front matter, if set. It can be a full URL, a path in the `static` folder (starting with `/`), or the filename of a [colocated asset](https://www.getzola.org/documentation/content/overview/#asset-colocation) of the post.
- Otherwise, the site-wide default `og_image` in the `[extra]` section of `zola.toml` is used, if set. It can be a full URL or a path in the `static` folder, e.g. `og_image = "img/og.png"`.
- If neither is set, image related tags are omitted.

An image around 1200x630 is recommended for the best display on most platforms.

## Analytics

To add scripts for analytics tools (such as Google Analytics, Umami, etc.), you can create  `myblog/templates/_head_extend.html`. The content of this file will be added to the html head of each page.

## Custom CSS

Copy `myblog/themes/serene/templates/_custom_css.html` to `myblog/templates/_custom_css.html`, variables in this file are used to control styles, such as the theme color `--primary-color`, modify them as you want.

If you want to customize more, you need to copy that file under the `templates`, `static`, `sass` directory in the corresponding `themes/serene` to the same name directory of `myblog`, and modify it. Be careful not to directly modify the files under the serene directory, because these modifications may cause conflicts if the theme is updated.

If you want to use a custom font, create a new `myblog/templates/_custom_font.html` and put the font link tags (for example, from [google fonts](https://fonts.google.com/)) into it, and then modify `--main-font` or `--code-font` in `myblog/sass/templates/_custom_css.html`. For performance reasons, you may want to self-host font files, but it's optional:

1. Open [google-webfonts-helper](https://gwfh.mranftl.com) and choose your font.
2. Modify `Customize folder prefix` of step 3 to `/font/` and then copy the css.
3. Replace the content of `myblog/templates/_custom_font.html` with a `<style> </style>` tag, with the css you just copied.
4. Download step 4 font files and put them in `myblog/static/font/` folder.

## Front Matter

The content inside the two `+++` at the top of the post markdown file is called front matter, used to provide metadata and config options of that post. Supported options:

```md
+++
title = ""
description = ""
date = 2022-01-01
updated = 2025-01-01
draft = true

[taxonomies]
categories = ["one"]
tags = ["one", "two", "three"]

[extra]
lang = "en"
toc = true
comment = false
code_copy = true
outdated_alert = true
outdated_alert_days = 120
math = false
mermaid = false
featured = false
reaction = false
og_image = "cover.png"
+++

new post about something...
```

Display options follow a unified fallback chain: **post front-matter → the post's section `_index.md` → `[extra]` of `zola.toml`**, the closest one wins. This applies to `toc`, `code_copy`, `comment`, `math`, `mermaid`, `reaction`, `outdated_alert` and `outdated_alert_days` (`date_format` and `outdated_alert_text_before/after` follow a section → config chain). So you can set site-wide defaults in `zola.toml`, override them per section, and override again per post.

If you set `categorized = true`, posts will be sorted alphabetically by default, you can manually set the order by adding a prefix  `__[0-9]{2}__` in front of the category name, for example, `categories = ["__01__CatXXX"]`

## Table of Contents

Set `toc = true` to display of table-of-contents.

## Math & Chart

Set `math = true` to enable formula rendering with KaTeX.

Set `mermaid = true` to enable chart rendering with Mermaid.

## Featured Mark

Set `featured = true` to display an asterisk(*) mark in front of the title.

## Outdated Alert

If one of your posts has strong timeliness, you can display an outdated alert after certain days.

Set `outdated_alert` and `outdated_alert_days` to enable the alert.

Options `outdated_alert_text_before` and `outdated_alert_text_after` are the text content of the alert, they can be set in `[extra]` of `zola.toml`, or per section in its `_index.md`.

## Comment

You can use [giscus](https://giscus.app) as the comment system.

To enable it, you need to create `myblog/templates/_giscus_script.html` and put the script configured on the giscus website into it, then change the value of `data-theme` to `https://<your-domain-name>/giscus_light.css`, replace `<your-domain-name>` with you domain name, same as `base_url` in `zola.toml`, if you set `color_scheme` to `"dark"`, replace `giscus_light.css` with `giscus_dark.css`.

Then set `comment = true` to enable comment.

## Reaction

This theme supports a feature called anonymous emoji reaction, visitors of you site can react to your post with emojis, without the need to log in or register.

You need to setup a backend api endpoint to enable it. Your endpoint should handle both `GET` and `POST` request:

- `GET`

    Request query:

     `slug`: the slug of the post

    Response:

    ```jsonc
    {
      "👍": [123, true], // emoji: [count, reacted]
      "👀": [456, false]
    }
    ```

- `POST`

    Request body:

    ```json
    {
      "slug": "post-slug",
      "target": "👍",
      "reacted": true
    }
    ```

    Response:

    ```json
    {
      "success": true
    }
    ```

For conivence, you can use one template repo to setup your own endpoint:

-  [isunjn/reaction](https://github.com/isunjn/reaction): All you need is a [Cloudflare](https://cloudflare.com) account. The free tier is good enough for a low-traffic personal blog.

- [mildronize/reaction](https://github.com/mildronize/reaction): Specific to [Azure](https://azure.microsoft.com/) platform.
- [sorokya/reaction](https://github.com/sorokya/reaction): A self-contained one, you can run it with docker.

After you setup your endpoint, set `reaction_endpoint = "<your-endpoint>"` and `reaction = true` to enable it.

Giscus also support a reaction feature, but it requires visitors to log in to GitHub, you can disable it in giscus's settings.

## Codeblock

Zola supports some [annotations for code blocks](https://www.getzola.org/documentation/content/syntax-highlighting/#annotations).

## Callouts

Callouts use the [GitHub alert syntax](https://github.com/orgs/community/discussions/16925), there are 5 types: `NOTE` `TIP` `IMPORTANT` `WARNING` `CAUTION`:

```md
> [!NOTE]
> note text
```

Serene styles them with an icon and a title. The title texts default to "Note" / "Tip" / "Important" / "Warning" / "Caution", you can change them (e.g. for a non-English site) by setting css variables in your `_custom_css.html`:

```css
--callout-note-title: "注意";
--callout-tip-title: "提示";
--callout-important-title: "重要";
--callout-warning-title: "警告";
--callout-caution-title: "当心";
```

## Components

Since zola `0.23`, your markdown content is itself a [Tera](https://keats.github.io/tera/) template, and shortcodes were replaced by [Tera components](https://www.getzola.org/documentation/content/overview/#templating-your-content). Serene provides some built-in components.

Note that component arguments other than strings are wrapped in `{...}`, e.g. `autoplay={true}`.

- Use `figure` to add caption or width/height to an image, `alt` `caption` `width` `height` are all optional (`width` and `height` take strings):

  ```md
  {{ <figure src="/path/to/img" alt="alt text" caption="caption text" width="600" height="400" /> }}
  ```

  If `src` is the filename of a [colocated asset](https://www.getzola.org/documentation/content/overview/#asset-colocation), pass `page` (or `section` when used in a section's `_index.md`) so the image URL can be resolved:

  ```md
  {{ <figure src="colocated-img.png" caption="caption text" page /> }}
  ```

  The caption is parsed as markdown so you can use bold / italic / link, for example `caption="[via](https://example.com)"`

  Adding height to an image is always recommended, as this can avoid page layout shift. When you use `[](https://exmaple.com/img.png)`, browser cannot determine the image's dimensions before it loads.

- Use `quote` to display a special quote block, `cite` is optional:

  ```md
  {% <quote cite=""> %}
  // content...
  {% </quote> %}
  ```

- Use `detail` to add an expandable detail block, `default_open` is optional:

  ```md
  {% <detail title="" default_open={false}> %}
  // content...
  {% </detail> %}
  ```

- Use `mermaid` to add a mermaid chart:

  ```md
  {% <mermaid> %}
  flowchart LR
  A[Hard] -->|Text| B(Round)
  B --> C{Decision}
  C -->|One| D[Result 1]
  C -->|Two| E[Result 2]
  {% </mermaid> %}
  ```

- Use `youtube` to embed a youtube video, `autoplay` is optional, default to `false`:

  ```md
  {{ <youtube id="<youtube-video-id>" autoplay={true} /> }}
  ```

Since your markdown content is now a Tera template, if you want to write literal `{{` or `{%` in your content (e.g. in a code block), wrap it with `{% raw %}` and `{% endraw %}`.

Note that component calls must be at the top level of your content — don't nest them inside a list item, as the component's HTML output would break the list's indentation rules and produce broken HTML.

## Collection

This theme has a special component for creating a collection of items. These collections can be used to showcase various types of your list, such as projects, publications, blogroll, bookmarks, etc. Check [this page](http://serene-demo.pages.dev/collections) on demo site to see some examples.

Currently, there are 7 types of collection item:

- `card`

    ```toml
    [[collection]]
    type = "card"
    title = "Title"
    subtitle = "Subtitle" # optional; supports Markdown
    date = "Date" # optional
    link = "https://example.com" # optional
    icon = "https://example.com/image.png" # optional
    content = "Content" # supports Markdown
    tags = ["tag1", "tag2"] # optional
    featured = false  # optional
    ```

- `card_simple`

    ```toml
    [[collection]]
    type = "card_simple"
    title = "Title"
    date = "Date" # optional
    link = "https://example.com" # optional
    icon = "https://example.com/image.png" # optional
    content = "Content" # supports Markdown
    featured = false  # optional
    ```

- `entry`

    ```toml
    [[collection]]
    type = "entry"
    title = "Title" # optional
    subtitle = "Subtitle" # optional
    link = "https://example.com" # optional
    icon = "https://example.com/image.png" # optional
    ```

- `box`

    ```toml
    [[collection]]
    type = "box"
    title = "Title"
    subtitle = "Subtitle" # optional
    link = "https://example.com" # optional
    img = "https://example.com/image.png" # optional
    ```

- `art`

    ```toml
    [[collection]]
    type = "art"
    title = "Title"
    subtitle = "Subtitle" # optional; support Markdown
    link = "https://example.com" # optional
    img = "https://example.com/image.png"
    content = "Content" # optional; supports Markdown
    footer = "Footer" # optional; supports Markdown
    ```

- `art_simple`

    ```toml
    [[collection]]
    type = "art_simple"
    title = "Title"
    subtitle = "Subtitle" # optional; supports Markdown
    link = "https://example.com" # optional
    img = "https://example.com/image.png"
    ```


List your items in a toml file and then use the `collection` component to render them.

For example, to create a "projects" section page:

1. Create `myblog/content/projects/projects.toml`:

    ```toml
    [[collection]]
    type = "card"
    title = "Tokio"
    link = "https://example.com"
    content = "Tokio is an asynchronous runtime for the Rust programming language. It provides the building blocks needed for writing network applications. It gives the flexibility to target a wide range of systems, from large servers with dozens of cores to small embedded devices."
    tags = ["rust", "async", "runtime"]

    [[collection]]
    type = "card"
    title = "Kubernetes"
    link = "https://example.com"
    content = "Kubernetes, also known as K8s, is an open source system for managing containerized applications across multiple hosts. It provides basic mechanisms for the deployment, maintenance, and scaling of applications."
    tags = ["k8s", "golang"]

    [[collection]]
    type = "card"
    title = "Next.js"
    link = "https://example.com"
    content = "Next.js is a React framework for building full-stack web applications. You use React Components to build user interfaces, and Next.js for additional features and optimizations."
    tags = ["typescript", "react", "frontend"]

    ```

2. Create `myblog/content/projects/_index.md`:

    ```
    +++
    title = "My projects"
    description = "Projects page of ..."
    template = "prose.html"

    [extra]
    title = "Projects"
    subtitle = "Some cool projects I made"
    +++

    {{ <collection file="projects.toml" section /> }}
    ```

3. Add projects section in `nav` of `zola.toml`

    ```toml
    nav = [
      # ...
      { name = "projects", path = "/projects" },
    ]
    ```

## Build & Deploy

Local preview:

```sh
zola serve
```

Build the site:

```sh
zola build
```

To deploy a static site, refer to zola's [documentation about deployment](https://www.getzola.org/documentation/deployment/overview/).

## Update

Check the [CHANGELOG.md](https://github.com/isunjn/serene/blob/main/CHANGELOG.md) on github for breaking changes before you update.

If you copied some files from `myblog/themes/serene` to `myblog/` for customization, such as `_custom_css.html` or `main.scss`, then you should record what you have modified before you update, re-copy those files and re-apply your modification after updating. The `zola.toml` should be re-copied too.

You can watch (`watch > custom > releases > apply`) this project on github to be reminded of a new release.

```sh
git submodule update --remote themes/serene
```
