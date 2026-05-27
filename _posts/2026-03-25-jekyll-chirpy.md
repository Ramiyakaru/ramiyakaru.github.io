---
title: "Start Your Own Blog for Free using Jekyll, Chirpy & GitHub Pages (Windows Guide)"
date: 26-03-30 22:30:00 +1030
categories: [Jekyll, Blogging]
tags: [jekyll, ruby, github, chirpy, beginner]
---

## Want to start your own blog for FREE?

If you’ve ever wanted to create your own blog to document your learning journey, projects, or tutorials. This guide will help you set one up **completely free** using:

- Ruby  
- Jekyll  
- Chirpy Theme  
- GitHub

By the end of this guide, you’ll have a **fully working blog running on your local machine**, ready to be deployed online.

---

## What are these tools?

### What is Ruby?

Ruby is a programming language that Jekyll uses to generate static websites.

Think of it as the **engine** that runs Jekyll.

---

### What is Jekyll?

Jekyll is a **static site generator**.

It takes your content (Markdown files) and converts it into a full website.

No databases, no backend, just fast, simple websites.

---

### What is Chirpy?

Chirpy is a modern and powerful **Jekyll theme** designed for technical blogs.

It provides:

- clean UI  
- categories & tags  
- dark mode  
- responsive design  

---

### What is GitHub Pages?

GitHub Pages lets you **host your website for free** directly from your GitHub repository.

Your blog will be live at: [Github](https://ramiyakaru.github.io)

---

### What is Markdown?

Markdown is a lightweight language used to write blog posts.

Example:

```markdown
## Heading
**Bold text**
- List item
```

I’ll write a separate detailed post about Markdown later.

### 1️⃣ Install Ruby

Download Ruby 3.2.x with DevKit: [Link](https://rubyinstaller.org/downloads/)

![Ruby Install](/assets/img/blog1/1-ruby-download-page.webp)
*Ruby download page*

During installation:
✅ Add Ruby executables to PATH

![Add Ruby executables to PATH](/assets/img/blog1/2-add-RUBY-executables-to-your-PATH.webp)
*Tick "Add Ruby executables to PATH"*

✅ Select MSYS2 development toolchain

![Select MSYS2 development toolchain](/assets/img/blog1/3-Ruby-MSYS2-development-toolchain.webp)
*Tick "MSYS2 development toolchain"*

✅ Tick "Run ridk install"
![Tick "Run ridk install"](/assets/img/blog1/4-run-ridk-install-toolchain.webp)
*Tick "Run 'ridk install .....'" and finish installation setup*

### 2️⃣ Install MSYS2 Toolchain

After installation, a terminal opens.

Select: 3

![Ruby cmd menu](/assets/img/blog1/5-ruby-cmd-menu.webp)
*Ruby CMD menu*
![Installing MSYS2 and MINGW development toolchain](/assets/img/blog1/6-installing-MSYS2-and-MINGW-development-toolchain.webp)
*Installing MSYS2 and MINGW development toolchain*
![MSYS2 and MINGW development toolchain setup finish](/assets/img/blog1/7-MSYS2-and-MINGW-development-toolchain-setup-done.webp)
*Successful installation of MSYS2 and MINGW development toolchain*

### 3️⃣ Verify Ruby Installation

```bash
ruby -v
```

![Check Ruby version](/assets/img/blog1/8-1-check-Ruby-versio.webp)
*Check installed RUBY version*

### 4️⃣ Check Bundler

Bundler manages Ruby dependencies.

```bash
bundle --version
```

![Check bundler version](/assets/img/blog1/8-2-check-bundler-version.webp)
*Check installed bundler version*

### 5️⃣ Install Jekyll & Bundler

```bash
gem install jekyll bundler
```

![Installing Jekyll dependencies](/assets/img/blog1/9-installing-Jekyll-dependencies.webp)
*Installing required Jekyll dependencies*
Verify:

```bash
jekyll -v
```

![Check jekyll version](/assets/img/blog1/10-check-jekyll-version.webp)
*Check installed jekyll version*

### 6️⃣ Get Chirpy Starter Template

Go to:

[Gituhb Chirpy Starter](https://github.com/cotes/chirpy-starter)

Click:

Use this template
Create new repository

Name it:
 yourusername.github.io for example ramiyakaru.github.io

![Chirpy starter repo](/assets/img/blog1/11-chirpy-starter-repo.webp)
*Check Chirpy start repository by cotes*
![Create new repo with chirpy](/assets/img/blog1/12-create-new-repo-with-chirpy.webp)
*Creating a new repository with chirpy*
![created new repo](/assets/img/blog1/13-created-new-repo-.webp)
*Created new chripy repository*

### 7️⃣ Download Repository Locally

Option A: Using Git (recommended)

Download Git:
[Git download link](https://git-scm.com/install)

![Download git](/assets/img/blog1/15-download-GIT.webp)
*Git download page*
Clone repo:

```bash
git clone https://github.com/RamiyaKaru/ramiyakaru.github.io.git
```

![Git clone command](/assets/img/blog1/16-git-clone-command.webp)
*Git repository clone command*
![Successful clone](/assets/img/blog1/17-after-sucessful-clone.webp)
*Successfully cloned repository*
![Cloned repo on local disk](/assets/img/blog1/18-cloned-repo-on-PC.webp)
*Successfully cloned repository on local disk*

Option B: Download ZIP

Download and extract manually.

### 8️⃣ Install Project Dependencies

Use this following command to install required dependencies for this chirpy project.

```bash
bundle install
```

![Install additional dependencies required by chirpy](/assets/img/blog1/19-install-additional-dependencies-needed-for-chirpy-repo.webp)
*Installing addtional dependencies required by Chirpy*

### 9️⃣ Open Project in VS Code

Visual Studio Code (VS Code) is a free, lightweight, and powerful open-source code editor developed by Microsoft for Windows, macOS, and Linux. It acts as an integrated development environment (IDE) for many languages, featuring built-in AI support, debugging, Git control, and customizable extensions. You can download VS Code from following link

Download:
[VS Code link](https://code.visualstudio.com/download)

You can use following bash command to open the project folder on VS Code.

```bash
code .
```

![Config file opened with VS Code](/assets/img/blog1/20-config-file-opened-with-vs-code.webp)
*Chirpy coonfiguration file opened with VS Code*

### 🔟 Run the Website Locally

Once everything is done. You can host your website on localhost with jekyll by using the following command.

```bash
bundle exec jekyll serve
```

Open your browser and go to this link to preview your hosted website.

<http://127.0.0.1:4000/>

![Running chirpy on localhost with jekyll](/assets/img/blog1/21-running-chirpy-on-localhost-with-jekyll.webp)
*Running Chirpy on localhost with Jekyll*
![Running chirpy website on localhost](/assets/img/blog1/22-running-the-website-on-localhost.webp)
*Running Chirpy website on localhost*

### 1️⃣1️⃣ Configure Your Blog

If everything was configured properly, you should see the chirpy base website on your browser. Now we have to edit the config file of this website to match our website. Most of the configurations are saved in the config.yml file. Open the config file from vs code and let's start editing these followings first. Make sure to add your repo link as the URL in config file (this is later used to host our website with github pages).

- title
- timezone
- URL → ramiyakaru.github.io
- social links
- email

![Changing few settings on config file](/assets/img/blog1/23-changing-few-settings-on-config-file.webp)
*Changing initial settings on Chirpy configuration file*

Add Avatar

You can add your own avatar to the sidebar. To do that create a folder inside the assets folder names 'img' and add your avatar image (Try changing image dimensions to fit it nicely)

```yml
avatar: /assets/img/yourimage.webp
```

![Changing sidebar avatar with a local file](/assets/img/blog1/24-changing-sidebar-avatar-with-a-local-file.webp)
*Changing sidebar avatar with a file stored in local disk*
![Output after changing avatar](/assets/img/blog1/25-output-after-changes.webp)
*Final look after changed avatar*

### 1️⃣2️⃣ Create Your First Blog Post

Now that our website is running locally, it’s time to create our first blog post.

Jekyll stores all blog posts inside the `_posts` folder. Each post must follow a specific naming format:

`YYYY-MM-DD-title.md`

This format is important because Jekyll uses the date and filename to organize and display your posts correctly.
Go to:
`_posts`

Create a new file like:
`26-03-24-my-first-post.md`

![Starting first blog post heading](/assets/img/blog1/26-starting-first-blog-post-heading.webp)
*Creating a first blog post heading*

At the top of the file, add the following:

```markdown
title: "Title of your blog post"
date: 26-03-25 :00:00 +1030
categories: [Jekyll]
tags: [blog, beginner]
```

This section is called front matter. It tells Jekyll important information about your post such as:

- The title of your post
- The publish date
- Categories (used for grouping content)
- Tags (used for filtering and search)

Once you add this, your post is ready, you can now start writing your content below it.

---

### **Write Content in Markdown**

```markdown
## 1️⃣3️⃣ Write Content in Markdown

Jekyll uses **Markdown** to write blog content.

Markdown is a lightweight and easy-to-learn formatting language that allows you to write structured content without needing complex HTML.

Instead of writing code-heavy pages, you can simply use symbols and formatting like:

- `#` for headings  
- `**text**` for bold  
- `-` for lists  

This makes writing blog posts much faster and cleaner, especially for technical content.

For example:

## This is a heading

This is **bold text**

- Item 1
- Item 2
```

Jekyll will automatically convert this into a properly formatted webpage.

![Website with first blog post](/assets/img/blog1/27-website-with-first-blog-post.webp)
*Locally hosted website with first blog post*

#### Example Markdown Features

Markdown supports a wide range of features that are very useful for technical blogs:

#### Headings & Text Formatting

![Markdown syntax for headings and text formatting](/assets/img/blog1/28-markdown-syntax-for-headings-and-text-formatting.webp)
*Markdown syntax for headings and text formatting*
![Output of headings and text formatting](/assets/img/blog1/29-output-of-headings-and-text-formatting.webp)
*Output of headings and text formatting*

#### Lists (Ordered & Unordered)

![Markdown syntax for lists](/assets/img/blog1/30-markdown-syntax-for-ordered-list-and-unordered-lists.webp)
*Markdown syntax for list*
![Output of lists](/assets/img/blog1/31-output-of-for-ordered-list-and-unordered-lists.webp)
*Output of lists*

#### Tables, Links & Task Lists

![Markdown syntax for tables, links, and task lists](/assets/img/blog1/32-markdown-syntax-for-table,-task-list-and-links.webp)
*Markdown syntax for tables, links, and task lists*
![Output of tables, links, and task lists](/assets/img/blog1/33-output-of-table,-task-list-and-links.webp)
*Output of tables, links, and task lists*

#### Images

![Markdown syntax for images](/assets/img/blog1/34-markdown-syntax-for-images.webp)
*Markdown syntax for images*
![Output of images](/assets/img/blog1/35-output-of-images.webp)
*Output of images*

#### Code Blocks

![Markdown syntax for code block](/assets/img/blog1/36-markdown-syntax-for-code-block-and-fenced-code-block.webp)
*Markdown syntax for code block*
![Output of code blocks](/assets/img/blog1/37-output-of-code-block-and-fenced-code-block.webp)
*Output of code blocks*

Markdown is simple, but very powerful.

In a future post, I’ll cover Markdown in detail with all features and examples.

---

## Wrapping Up

And that’s it - you now have your very own blog running locally using **Jekyll** and the **Chirpy theme** 🎉

In this guide, we covered everything from:

- Installing Ruby and required dependencies  
- Setting up Jekyll  
- Using the Chirpy starter template  
- Running your site locally  
- Creating your first blog post  

At this point, you have a solid foundation to start building and customizing your blog.

---

## Next Step: Go Live with GitHub Pages

Right now, your site is running locally on:

```bash
http://127.0.0.1:4000/
```

The next step is to publish it online using **GitHub Pages**, so others can access your blog from anywhere.

I’ll cover this in an upcoming post where we’ll:

- Push the project to GitHub  
- Enable GitHub Pages  
- Make your blog publicly accessible  

---

## Keep Building

The best way to improve is to start writing.

Don’t worry about making things perfect, focus on:

- Sharing what you learn  
- Documenting your process  
- Helping others who are starting out  

Even simple posts can be valuable.

---

## Final Thoughts

Starting a blog might feel overwhelming at first, especially with new tools and technologies.  

But once everything is set up, it becomes a powerful platform where you can:

- Showcase your skills  
- Track your progress  
- Build your personal brand  

For me, this blog is more than just a website
it’s my **IT Sandbox**, where I experiment, learn, and grow.

---

## What’s Coming Next?

- A complete guide to **Markdown syntax**  
- More tutorials on **networking and home labs**  
- Real-world troubleshooting and setups  

---

***If you found this helpful, stay tuned for more posts and start building your own sandbox***
