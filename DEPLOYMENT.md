# Deploying AI Templates Documentation

## Quick Deploy to GitHub Pages

### Step 1: Verify Git Remote

Check that your remote is correctly configured:

```bash
cd /path/to/your/repo

# Check current remote
git remote -v

# Should show:
# origin  git@github.com:redhat-data-and-ai/website.git
```

### Step 2: Commit Your Changes

```bash
# Stage all changes
git add .

# Commit with a descriptive message
git commit -m "docs: update documentation content"

# Push to main
git push origin main
```

### Step 3: Enable GitHub Pages

1. Go to your GitHub repo settings → Pages
   - URL: https://github.com/redhat-data-and-ai/website/settings/pages
2. Under "Build and deployment":
   - Source: **GitHub Actions**
   - Custom domain: (optional, configure if you have one)
   - Enforce HTTPS: Check this option
3. Save

The GitHub Actions workflow (`.github/workflows/deploy.yml`) will automatically:
- Build the Hugo site
- Deploy to GitHub Pages
- Make it available at your configured URL

### Step 4: Wait for Deployment

- Go to **Actions** tab in GitHub
- Watch the "Deploy Hugo site to GitHub Pages" workflow
- Takes about 2-3 minutes
- When green checkmark appears, site is live!

## Manual Build (Testing)

To test the production build locally:

```bash
cd /path/to/your/repo

# Build static site
hugo --cleanDestinationDir --minify

# Output is in public/ directory
# You can preview with:
python3 -m http.server 8000 -d public
# Then visit http://localhost:8000
```

## Troubleshooting

**Build fails on GitHub Actions:**
- Check the Actions log for specific errors
- Verify all files are committed
- Ensure .gitignore doesn't exclude necessary files

**Site shows but looks broken:**
- Check baseURL in config.yaml matches your GitHub Pages URL
- Verify all static assets are included
- Check browser console for 404 errors

**Changes don't appear:**
- Wait 1-2 minutes for GitHub Pages to update
- Hard refresh browser (Cmd+Shift+R)
- Check Actions tab to ensure deploy succeeded

## Site is Live!

Once deployed, your site will be at:

**GitHub Pages:** https://redhat-data-and-ai.github.io/website/

If you configured a custom domain, it will be available there as well.

Share it with the world!

