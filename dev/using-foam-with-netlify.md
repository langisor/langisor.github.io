# Using Foam with GitHub and netlify

<!-- toc -->

- [Using Foam with GitHub and netlify](#using-foam-with-github-and-netlify)
  - [What is Foam?](#what-is-foam)
  - [Needed Knowledge](#needed-knowledge)
  - [Steps to Create Your Foam PKM Website](#steps-to-create-your-foam-pkm-website)
    - [Step 1: Set up your Foam Workspace (Local)](#step-1-set-up-your-foam-workspace-local)
    - [Step 2: Push Your Foam Notes to GitHub](#step-2-push-your-foam-notes-to-github)
    - [Step 3: Publish Your Foam Site with Netlify](#step-3-publish-your-foam-site-with-netlify)
  - [Continuous Deployment](#continuous-deployment)
  - [Recap of Steps](#recap-of-steps)

<!-- tocstop -->

## What is Foam?

**Foam** is a personal knowledge management and sharing system built on Visual Studio Code and GitHub. It's inspired by Roam Research and leverages Markdown files with "wikilinks" (like `[[this]]`) to connect your notes, allowing you to explore relationships between your thoughts and build a rich knowledge graph.

**Key features of Foam:**

- **Markdown-based:** Your notes are plain text Markdown files, making them portable and future-proof.
- **Wikilinks:** Easily link notes together for a connected knowledge base.
- **Graph visualization:** See how your notes are interconnected.
- **Backlinks:** Discover which notes reference the current note.
- **Tags:** Organize your content with tags.
- **Daily notes:** Create a journal of your thoughts.
- **Extensible:** Works with VS Code extensions to customize your workflow.

## Needed Knowledge

Before diving in, it's helpful to have a basic understanding of these concepts:

1. **Markdown:** This is a lightweight markup language for creating formatted text using a plain-text editor. You'll be writing all your notes in Markdown. It's easy to learn the basics (headings, lists, bold, italics, links).
2. **Git:** A distributed version control system that tracks versions of files. It's essential for managing changes to your Foam notes.
3. **GitHub:** A web-based platform for version control and collaboration, built on Git. You'll use it to store your Foam notes as a public repository and for Netlify to deploy your site.
4. **VS Code (Visual Studio Code):** Foam is a VS Code extension, so familiarity with this code editor is crucial. You'll be using it to write and manage your Foam notes.
5. **Netlify:** A remote-first cloud computing company that offers a development platform for web applications and dynamic websites. You'll use it to publish your Foam site to the internet.

## Steps to Create Your Foam PKM Website

### Step 1: Set up your Foam Workspace (Local)

1. **Install VS Code:** If you don't have it already, download and install Visual Studio Code.

2. **Install Foam Extension:**

      - Open VS Code.
      - Go to the Extensions view (Ctrl+Shift+X or Cmd+Shift+X).
      - Search for "Foam" and install the official "Foam for VSCode" extension.

3. **Create a New Foam Workspace from Template:**

      - The easiest way to start is to use the official Foam template. Go to the `foambubble/foam-template` repository on GitHub.
      - Click the green "Use this template" button. This will create a new repository in your GitHub account based on the Foam template.
      - **Important:** Choose "Public" for your repository visibility if you want to publish it openly.
      - Clone this newly created repository to your local machine using Git:

        ```bash
        git clone https://github.com/YourGitHubUsername/YourFoamRepoName.git
        cd YourFoamRepoName
        ```

      - Open the cloned folder in VS Code (`File > Open Folder...`).
      - VS Code will likely prompt you to install recommended extensions for Foam. Click "Install all."

4. **Start Writing Notes:**

      - Explore the `getting-started.md` file in your new Foam workspace. It provides a good introduction to Foam's features.
      - Start creating new Markdown files (`.md`) for your notes. Use `[[wikilinks]]` to connect them. Foam provides autocompletion for these links.
      - Experiment with Foam's commands (accessible via `Ctrl+Shift+P` or `Cmd+Shift+P` in VS Code), such as:
          - `Foam: Create New Note`
          - `Foam: Open Daily Note`
          - `Foam: Show Graph` (to visualize your knowledge graph)
          - `Foam: Show Backlinks`

### Step 2: Push Your Foam Notes to GitHub

1. **Initialize Git (if not already done by cloning the template):** The Foam template already provides a `.git` folder, so this step is likely already done. If you started from scratch, you would do:

    ```bash
    git init
    git add .
    git commit -m "Initial Foam commit"
    git branch -M main
    git remote add origin https://github.com/YourGitHubUsername/YourFoamRepoName.git
    git push -u origin main
    ```

2. **Regularly Commit and Push:** As you create and modify notes, you'll need to commit your changes and push them to your GitHub repository.
      - In VS Code, use the Source Control view (Ctrl+Shift+G or Cmd+Shift+G) to stage your changes, write a commit message, and commit.
      - Then, click the "Sync Changes" button or use `git push` in your terminal to push your changes to GitHub.
    <!-- end list -->
    ```bash
    git add .
    git commit -m "Added new note about [topic]"
    git push origin main
    ```

### Step 3: Publish Your Foam Site with Netlify

Foam notes are essentially Markdown files. To publish them as a website, you need a static site generator that can process these Markdown files (especially the wikilinks) into HTML. While Foam itself doesn't directly *generate* a full website, it provides the foundation.

Many Foam users leverage existing static site generators (like Eleventy, Gatsby, Jekyll, or MkDocs) to render their Foam notes into a navigable website. The Foam template often comes pre-configured for GitHub Pages (which uses Jekyll), but for Netlify, you might consider Eleventy as it's often recommended with Foam.

Let's assume you'll use a simple setup that doesn't require a complex build process, allowing Netlify to directly serve your Markdown files (if rendered by client-side JavaScript, or if GitHub Pages Jekyll is enabled). However, for robust site generation with navigation and proper linking, a static site generator is ideal.

**Option A (Simpler - relying on client-side rendering or GitHub Pages defaults):**

1. **Sign up for Netlify:** Go to [netlify.com](https://www.netlify.com/) and sign up, preferably by connecting your GitHub account.
2. **Import a new project:**
      - From your Netlify dashboard, click "Add new site" -\> "Import an existing project."
      - Connect to GitHub.
      - Authorize Netlify to access your repositories.
      - Select the GitHub repository where your Foam notes are stored.
3. **Configure deploy settings:**
      - **Branch to deploy:** `main` (or whatever your main branch is called).
      - **Build command:** If you're not using a specific static site generator that requires a build step, you might leave this empty or enter `echo "No build command needed"`. If your Foam template has a `package.json` with a build script, Netlify might detect it automatically (e.g., `npm run build`).
      - **Publish directory:** This is the folder Netlify should serve. For a basic setup, this might be the root of your repository (`.`). If you're using a static site generator, it would be the output folder (e.g., `_site`, `public`, `dist`).
4. **Deploy:** Click "Deploy site." Netlify will connect to your GitHub repository, pull the files, and deploy your site.

**Option B (More Powerful - Using a Static Site Generator like Eleventy for a better experience):**

This approach involves adding a static site generator to your Foam repository. This will give you more control over the website's appearance, navigation, and how your wikilinks are rendered into proper HTML links.

1. **Add a Static Site Generator:**
      - **Choose a Generator:** Eleventy is a good choice for Foam.
      - **Integrate Eleventy into your Foam repo:** This typically involves:
          - Creating a `package.json` file in your repository if you don't have one.
          - Installing Eleventy as a development dependency: `npm install @11ty/eleventy --save-dev`
          - Creating an `.eleventy.js` configuration file to tell Eleventy how to process your Markdown files (especially wikilinks and how to handle Foam's specific syntax). This might involve custom plugins or shortcodes to convert `[[wikilinks]]` to `/notes/your-note.html` style links.
          - Structuring your Foam notes within a specific directory (e.g., `src/notes`).
          - Creating templates (e.g., Nunjucks, Liquid) for your pages.
          - Adding a `build` script to your `package.json` (e.g., `"build": "eleventy"`).
2. **Commit and Push Eleventy Changes:** Push all your new Eleventy files and configurations to your GitHub repository.
3. **Deploy to Netlify (as in Option A, but with specific build settings):**
      - **Build command:** `npm run build` (or `eleventy` if you prefer).
      - **Publish directory:** `_site` (or whatever your Eleventy output directory is configured to be).
      - Netlify will then run your `npm run build` command, generate the static HTML files, and publish them.

## Continuous Deployment

One of the greatest advantages of this setup is **continuous deployment**. Once Netlify is connected to your GitHub repository, every time you push changes to your `main` branch (or the branch you configured for deployment), Netlify will automatically detect those changes, rebuild your site, and deploy the updated version. This means you just focus on writing notes in VS Code and pushing to GitHub, and your website is automatically kept up-to-date.

## Recap of Steps

1. **Install VS Code** and the **Foam extension**.
2. **Create a new public GitHub repository** from the `foambubble/foam-template`.
3. **Clone** your GitHub repository to your local machine.
4. **Open the repository in VS Code** and install recommended Foam extensions.
5. **Write your notes** using Markdown and Foam's `[[wikilinks]]`.
6. **Commit and push your changes** to your GitHub repository regularly.
7. **Sign up for Netlify** and connect it to your GitHub account.
8. **Import your Foam repository** into Netlify.
9. **Configure Netlify's build settings** (especially if using a static site generator like Eleventy for a richer website experience).
10. **Deploy your site** on Netlify.

This setup provides a robust and flexible way to manage your personal knowledge and share it with the world\!
