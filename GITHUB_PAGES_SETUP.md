# 🚀 GitHub Pages Setup Guide

## Step-by-Step Instructions to Enable GitHub Pages

### 1. **Push Your Code to GitHub**

```bash
# Navigate to your devops-learning directory
cd /Users/ijazahammadshaik/workspace/riyaz/linux/devops-learning

# Initialize git (if not already done)
git init

# Add all files
git add .

# Commit
git commit -m "Add DevOps learning courses with GitHub Pages"

# Add your GitHub repository as remote
git remote add origin https://github.com/YOUR_USERNAME/devops-learning.git

# Push to GitHub
git push -u origin main
```

### 2. **Enable GitHub Pages**

1. Go to your repository on GitHub: `https://github.com/YOUR_USERNAME/devops-learning`

2. Click on **Settings** (top right)

3. In the left sidebar, click **Pages**

4. Under **Source**, select:
   - Branch: `main` (or `master`)
   - Folder: `/ (root)`

5. Click **Save**

6. Wait a few minutes for the site to build

7. Your site will be available at: `https://YOUR_USERNAME.github.io/devops-learning/`

### 3. **Update GitHub Links in index.html**

⚠️ **IMPORTANT**: Replace `reachharsha` with your actual GitHub username in index.html

Find and replace all instances of:
```
https://github.com/reachharsha/devops-learning/blob/main/
```

With:
```
https://github.com/YOUR_USERNAME/devops-learning/blob/main/
```

### 4. **Optional: Use Custom Domain**

If you have a custom domain:

1. In repository **Settings** → **Pages**
2. Under **Custom domain**, enter your domain (e.g., `learn.yourdomain.com`)
3. Create a `CNAME` file in your repository root with your domain name
4. Add DNS records at your domain provider:
   - Type: `CNAME`
   - Name: `learn` (or `www`)
   - Value: `YOUR_USERNAME.github.io`

### 5. **Verify Your Site**

Once GitHub Pages is enabled, visit:
```
https://YOUR_USERNAME.github.io/devops-learning/
```

You should see your beautiful DevOps learning portal! 🎉

### 6. **Update and Deploy**

Whenever you make changes:

```bash
# Make your changes
git add .
git commit -m "Update courses"
git push

# GitHub Pages will automatically rebuild (takes 1-2 minutes)
```

---

## 📂 Repository Structure

Your repository should look like this:

```
devops-learning/
├── index.html                    # Main landing page (GitHub Pages homepage)
├── README.md                     # Repository description
├── GITHUB_PAGES_SETUP.md        # This file
├── linux-fundamentals/          # Linux course files
│   ├── 00-START-HERE.md
│   ├── 01-what-is-linux.md
│   └── ... (all 18 lessons)
├── networking-essentials/       # Networking course files
│   ├── 00-START-HERE.md
│   ├── 01-what-is-networking.md
│   └── ... (all 17 lessons)
└── shell-scripting/             # Shell scripting course files
    ├── 00-START-HERE.md
    ├── 01-what-is-shell-scripting.md
    └── ... (all 31 lessons)
```

---

## 🎨 Customization Options

### Change Colors

Edit `index.html` to customize colors:

```css
/* Primary color (links, headings) */
color: #0366d6;  /* Change this to your preferred color */

/* Background */
background-color: #f8f9fa;
```

### Add Google Analytics

Add before `</head>` in index.html:

```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

### Add Social Media Links

Add in the footer section:

```html
<footer>
  <div style="margin-bottom: 1rem;">
    <a href="https://github.com/YOUR_USERNAME" style="margin: 0 10px;">GitHub</a>
    <a href="https://linkedin.com/in/YOUR_PROFILE" style="margin: 0 10px;">LinkedIn</a>
    <a href="https://twitter.com/YOUR_HANDLE" style="margin: 0 10px;">Twitter</a>
  </div>
  &copy; 2026 DevOps Learning. Built with ❤️ using GitHub Pages.
</footer>
```

---

## 🔧 Troubleshooting

### Site Not Loading?

1. Check repository is public (Settings → Danger Zone)
2. Verify GitHub Pages is enabled (Settings → Pages)
3. Wait 2-5 minutes after pushing changes
4. Check build status: Settings → Pages → "Your site is live at..."

### Links Not Working?

1. Ensure all markdown files are pushed to GitHub
2. Verify GitHub username in URLs is correct
3. Check file names match exactly (case-sensitive)

### 404 Error?

1. Ensure `index.html` is in the root of your repository
2. Clear browser cache (Ctrl+Shift+R or Cmd+Shift+R)
3. Check GitHub Actions for build errors (Actions tab)

---

## 📱 Mobile Friendly

Your site is already mobile-responsive! Test it on your phone by visiting the URL.

---

## 🎓 Sharing Your Knowledge

Once your site is live, share it:

- Add the link to your GitHub profile README
- Share on LinkedIn/Twitter
- Add to your resume/portfolio
- Submit to DevOps learning resource lists

Example README badge:
```markdown
[![DevOps Learning](https://img.shields.io/badge/DevOps-Learning-blue)](https://YOUR_USERNAME.github.io/devops-learning/)
```

---

## 🚀 Next Steps

1. ✅ Push code to GitHub
2. ✅ Enable GitHub Pages
3. ✅ Update GitHub username in links
4. ✅ Visit your live site
5. ✅ Share with the community!

**Your DevOps learning portal is ready to help others learn! 🎉**
