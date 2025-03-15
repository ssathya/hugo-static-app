+++
date = '2025-03-15T03:24:39-04:00'
draft = false
title = 'Add New Blog'
+++

# Blog Post Creation Workflow

This document outlines the process for creating new blog posts. It will be updated regularly as new techniques and workflow improvements are implemented.

## Prerequisites

* **Git Bash:** Used as the primary shell.
* **Git Extensions:** Preferred tool for managing Git operations.
* **Hugo:** Static site generator.

## Procedure

1. **Navigate to the Repository:**
   
   * Open Git Bash.
   * Navigate to the blog's repository directory: `~/source/Blog/static-app`.

2. **Create a Branch:**
   
   * Using Git Extensions, create a new branch for the blog post. This isolates changes and facilitates easier review.

3. **Create a New Blog Post:**
   
   * Execute the following command to create a new Markdown file for the blog post:
     
     ```bash
     hugo new posts/{blog-post-file-name}.md
     ```
     
     * This command creates an empty blog post file in the `C:\Users\sridh\source\Blog\static-app\content\posts\` directory.
     * Replace `{blog-post-file-name}` with the desired file name for the blog post.

4. **Edit the Blog Post:**
   
   * Open the newly created Markdown file and add the content of your blog post.

5. **Test the Draft:**
   
   * Preview the blog post in draft mode using the following command:
     
     ```bash
     hugo server -D
     ```
     
     * The `-D` flag enables draft content.
   
   * Review the changes in your local browser.

6. **Publish the Blog Post:**
   
   * Once satisfied with the draft, open the Markdown file and change the `draft` status:
     * Replace `draft = true` with `draft = false`.

7. **Final Test:**
   
   * Test the published version of the blog post using the following command:
     
     ```bash
     hugo server
     ```
   
   * Review the changes in your local browser.

8. **Commit and Deploy:**
   
   * Using Git Extensions, commit the changes to the branch.
   * Push the branch to the remote repository.
   * Merge the branch to main branch.
   * The changes will be automatically deployed to the Azure static website.
