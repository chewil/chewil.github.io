---
title: "How to Set Up a Hugo Website with GitHub Pages Using WSL on Windows"
date: 2025-09-06T12:00:00Z
draft: false
tags: ["hugo", "github pages", "wsl", "static site", "windows"]
categories: ["Web Development", "Tutorials"]
summary: "A complete guide to creating and deploying a Hugo site on a Windows machine using WSL and GitHub Pages."
---

If you're using a Windows machine and want to create and deploy a fast, modern static website with [Hugo](https://gohugo.io/), you're in the right place. In this tutorial, we'll walk through setting up a Hugo site using **WSL (Windows Subsystem for Linux)** and publishing it via **GitHub Pages**.

---

## 🧰 Prerequisites

Before we begin, make sure you have:

- Windows 10/11 with **WSL installed**
- A GitHub account
- Basic familiarity with Git

---

## 🔧 Step 1: Install WSL and Set Up Ubuntu

If you haven’t set up WSL yet:

1. Open PowerShell as Administrator:
   ```powershell
   wsl --install

1. Restart your computer when prompted.

1. On reboot, WSL will install Ubuntu by default. Set up your username and password when prompted.

To confirm installation:

```bash
wsl --list --verbose
```

Make sure your WSL version is 2:

```powershell
wsl --set-version Ubuntu 2
```

## 🏗️ Step 2: Install Hugo in WSL (Ubuntu)

1. Open your WSL terminal (Ubuntu).

1. Update and install dependencies:

   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install git curl wget -y
   ```

1. Install Hugo (replace version with latest available):

   ```bash
   wget https://github.com/gohugoio/hugo/releases/download/v0.123.4/hugo_extended_0.123.4_linux-amd64.deb
   sudo dpkg -i hugo_extended_0.123.4_linux-amd64.deb
   ```

   Test installation:

   ```bash
   hugo version
   ```

## 🌱 Step 3: Create a New Hugo Site

1. Create a folder for your projects:

   ```bash
   mkdir ~/hugo-sites && cd ~/hugo-sites
   ```

1. Create your site:

   ```bash
   hugo new site myblog
   cd myblog
   ```

1. Initialize Git:

   ```bash
   git init
   ```

1. Add a theme (e.g., Ananke):

    ```bash
    git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
    echo 'theme = "ananke"' >> hugo.toml
    ```

## ✍️ Step 4: Add Some Content

1. Create your first post:

   ```bash
   hugo new posts/my-first-post.md
   ```

1. Edit the markdown file and set draft = false:

    ```bash
    nano content/posts/my-first-post.md
    ```

🚗 Step 5: Run the Hugo Server Locally

Start the Hugo development server:

```bash
hugo server -D
```

Visit: http://localhost:1313 in your browser.

## 🐱 Step 6: Set Up GitHub Repositories

You'll need two GitHub repos:

Option 1: One repo (deploy to `gh-pages` branch)

- `myblog` (main branch: source code, deploys to `gh-pages`)

Option 2: Two repos (recommended)

- `myblog` – Hugo source code
- `myblog-pages` – contains the built site (`public/` folder)

## ☁️ Step 7: Configure GitHub Deployment

### A. Create Repositories on GitHub
Create two repositories:

- `myblog`
- `myblog-pages`

### B. Link Remotes

From your Hugo project root:

```bash
git remote add origin git@github.com:yourusername/myblog.git
git add .
git commit -m "Initial Hugo site"
git push -u origin main
```

Deploy the public site:

```
cd public
git init
git remote add origin git@github.com:yourusername/myblog-pages.git
git branch -M main
git add .
git commit -m "Deploying site"
git push -u origin main
cd ..
```

## ⚙️ Step 8: Automate Deployment

Create a deploy script:

```bash
nano deploy.sh
```

Paste:

```bash
#!/bin/bash

# Build the site
hugo

# Go to public directory
cd public

# Push to GitHub Pages repo
git add .
git commit -m "Deploying site $(date)"
git push origin main

# Return to root
cd ..
```

Make it executable:

```bash
chmod +x deploy.sh
```

Run it:

```bash
./deploy.sh
```

## 🌍 Step 9: Enable GitHub Pages

1. Go to `myblog-pages` on GitHub.

1. Navigate to **Settings → Pages.**

1. Select `main` branch as source.

1. Save.

Your site will be live at:

```
https://yourusername.github.io/myblog-pages/
```

## ✅ Done!

You now have a working Hugo site running on WSL with automatic publishing to GitHub Pages. You can update your content, run ./deploy.sh, and your site will update online.

---

## 🛠️ Bonus Tips

- Add a custom domain in GitHub Pages settings

- Use GitHub Actions for automatic deploys

- Add a `.gitignore` file to the main repo:

```
/public
/resources
```

---
## 💬 Final Thoughts

Running Hugo with WSL gives you the power of Linux development on Windows. It’s fast, clean, and deploys easily via GitHub Pages — a perfect stack for blogs, docs, and portfolios.

Happy building! 🚀
