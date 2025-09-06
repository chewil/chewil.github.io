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
