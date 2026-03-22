---
layout: post
title: "How to create github io blog"
date: 2025-03-22
---

Here's the full walkthrough, terminal commands and all.

**Step 1 — Create the repo on GitHub**

Go to github.com → New Repository → name it `yourusername.github.io` (use your actual GitHub username). Make it public. Don't add a README yet. Click Create.

**Step 2 — Set up your local folder**

Open Terminal and run:

```bash
mkdir ~/yourusername.github.io
cd ~/yourusername.github.io
git init
```

**Step 3 — Add the config file**

Create `_config.yml`:

```yaml
title: My Blog
description: Some words, occasionally.
theme: minima
```

**Step 4 — Create the index page**

Create `index.md` in the root:

```markdown
---
layout: home
title: Posts
---
```

That's it — the `minima` theme's `home` layout automatically generates a list of all your posts with dates.

**Step 5 — Add your first post**

Posts must live in a `_posts/` folder and follow the naming convention `YYYY-MM-DD-title.md`:

```bash
mkdir _posts
```

Then move/rename your existing markdown file to match the pattern, e.g.:

```bash
cp ~/path/to/your-file.md _posts/2025-03-22-my-first-post.md
```

Now open that file and add **front matter** to the very top (above your existing content):

```markdown
---
layout: post
title: "My First Post"
date: 2025-03-22
---

Your existing markdown content goes here...
```

**Step 6 — Push it**

```bash
git add .
git commit -m "initial blog setup"
git remote add origin https://github.com/yourusername/yourusername.github.io.git
git branch -M main
git push -u origin main
```

**Step 7 — Enable GitHub Pages**

Go to your repo on GitHub → Settings → Pages → under "Source" select **Deploy from a branch**, pick `main` and `/ (root)`. Hit Save.

Give it a minute or two, then visit `https://yourusername.github.io`. You'll see your index page with a link to your post.

**Every time you want to publish a new post going forward:**

```bash
# write your post
vim _posts/2025-06-15-some-new-thoughts.md

# push it
git add .
git commit -m "new post"
git push
```

The index page updates automatically — no manual linking needed. Jekyll sees anything in `_posts/` with the right filename pattern and front matter, and the `home` layout lists them all reverse-chronologically.