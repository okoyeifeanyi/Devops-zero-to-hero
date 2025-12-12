# Mini Project - Collaborative Website Development with Git and GitHub

### This repository is part of my DevOps learning journey.  
The goal is to practice real-world Git workflows by building a simple collaborative website and managing the code with Git and GitHub.  

I’ll be simulating a small team (“Tom and Jerry”) working together: creating branches, making changes, opening pull requests, reviewing code, and merging back into the main branch.

## Part 1: Setup and Initial Configuration

### Prerequisites

I already have:

- Git installed on my machine  
- A GitHub account

### 1. Create a New GitHub Repository

1. Log in to my GitHub account.
2. Click the **“+”** icon in the top-right corner and choose **“New repository”**.

![alt text](<img/1. Create New Repo.png>) 

3. Fill in the repository details:
   - **Repository name:** `ecom-startup-website`
   - **Description:** Short note about this being a Git/GitHub practice project
   - Choose **Public** or **Private**
   - Optionally initialize with a **README.md**
4. Click **“Create repository”**.

![alt text](<img/2. Create New Repo.png>)

### 2. Clone the Repository to My Local Machine
On my repository page on Github, click the `Code` button and copy the HTTPS URL.

![alt text](<img/3. Cloning a git repo.png>)

Open my gitbash, navigate to the folder you want to save my project, create a new folder `devops-projects`

![alt text](<img/4. create a new folder for devops project.png>)

I created another folder called `git-project` inside the devops-projects directory and changede directory into the `git-project`.

![alt text](<img/5. create a new folder for git project.png>)

I cloned the repository from Github using `git clone [URL copied from Github]`

![alt text](<img/6. git clone.png>)

### 3. CD into our repo directory and Configure Git (If Needed)

If not already configured globally:

![alt text](<img/7. configure git.png>)

### 4. Verify Everything Is Connected
git remote -v
git status

![alt text](<img/8. Verify everything is connected.png>)

At this point, my local project is connected to the GitHub repository and ready for the next steps: creating branches, making changes to the website, and practicing collaborative workflows.

## Part 2: Create the First Page and Initial Commit

After cloning the repository, my default branch is `main` and changing directory into the project directory

### 1. Create the main HTML file
I created an empty index.html file that will act as entry point for this simple website.
To create this, I ran the `vim` command. 

![alt text](<img/9. Create an empty indexhtml file.png>)

This command creates a file if it doesn't exist already and also opens up the text editor if you want to add content to the file.
I added the code below inside the index.html file:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>DevOps Git Practice</title>
  </head>
  <body>
    <h1>Welcome to my DevOps Git practice project</h1>
    <p>This is the initial index.html file created on the main branch.</p>
  </body>
</html>
```
![alt text](<img/10. indexhtml content.png>)

### 2. Check the Git status
Check that changes has not been staged using `git status`.

![alt text](<img/11. git status.png>)

At this point, index.html appears as an untracked file.

### 3. Stage the changes and confirm changes have been staged for commit

I staged the changes using `git add [file name]` and confirm changes have been staged for commit using `git status`.

![alt text](<img/12. Git Add.png>)

### 4. Create the first commit

I made the first commit using the `git commit -m "1st commit statement"`

![alt text](<img/13. Git commit.png>)

This takes the staged changes and records them in the repository's history with a message describing what was done. This commit is a milestone, marking a specific point in the project's development.

Now the repository has its first tracked file and commit on the main branch, and I’m ready to start branching and simulating collaborative work.

### 5. Push main branch to Github

To push the main branch to Github. I use the `git push origin main`. This command sends the commits from your local main branch on my laptop to the main branch on GitHub (the remote repository).

![alt text](<img/14. git push.png>)


## Part 3: Simulating Tom and Jerry's Work

To pretend two developers (Tom and Jerry) are sharing one laptop, we’ll use two branches and switch between them, making changes as each person.

## 1. Tom’s work

### 1a. Move into the project directory you just cloned:

`cd "name of directory"`

![alt text](<img/15. Simulating TJ work.png>)

This takes you into the folder that contains the cloned GitHub repository on your machine. it's like stepping into the project's workspace.

- Check the current branch

I can check the current branch using `git branch`. 

![alt text](<img/16. Git branch.png>)

Git will list all the branches in your local repository and put an asterisk * next to the one you’re currently on. At this point you should only see the default branch, main, because no other branches exist yet.

### 1b. Create a new branch for Tom’s work

Now I’ll create a separate branch just for Tom’s changes.

````bash
git checkout -b update-navigation
````
![alt text](<img/17. Git checkout.png>)

This command:

- Creates a new branch called update-navigation (you can choose any name).

- Automatically switches you from main to this new branch.

- Copies everything from main into update-navigation, so you can work on Tom’s changes without touching the main branch.

### 1c. Confirm you’re on the new branch

I confirmed I am on the new branch using the `git branch` command.

![alt text](<img/18. git branch.png>)

Git will list my local branches and put an asterisk * next to the active one.
I should now see:

- main

- \* update-navigation

The * update-navigation shows you’re currently working on Tom’s branch.

Remember: the index.html file you created on main is also available on this branch.
Next, you’ll edit index.html here to simulate Tom’s navigation updates.

### 1d. Make Tom’s change in `index.html`

With the `update-navigation` branch checked out, open `index.html` in your editor and add the line:

```text
This is Tom adding Navigation to the Ecom-startup-website
````
![alt text](<img/19. vi inot index file.png>)

![alt text](<img/20. vi editor.png>)

This line simulates Tom's contribution to the project. This is Tom adding a code to the already existing code to add navigation bar to the website.

### 1e. Check that the change isn’t staged yet

Run:

````
git status
````
Git will show something like:

- On branch update-navigation

- index.html listed under Untracked files or Changes not staged for commit (and shown in red).

This means Git can see that index.html has changed, but the change has not been added to the staging area yet.

![alt text](<img/21. git status.png>)

### 1f. Stage Tom’s changes
To stage Tom's changes, run:
````
git add index.html
````
![alt text](<img/22. git add index.png>)

This tracks the file and moves Tom’s change into the staging area, ready to be committed – you’re basically saying, “include this in the next snapshot. If you run git status again, index.html will now appear in green which means it’s staged and ready to be committed.

![alt text](<img/23. git status to show index in green.png>)

### 1g. Commit Tom’s changes

Now record Tom’s work in the repository history by running:

`git commit -m "Update navigation bar"`

![alt text](<img/24. git commit update nav bar.png>)

This takes the staged changes and records them in the repository's history with a message describing what was done. this commit is a milestone, making a specific point in the project's development.

### 1h. Push Tom’s branch to GitHub

Send Tom’s `update-navigation` branch to the remote repository:

```bash
git push origin update-navigation
```
![alt text](<img/25. Git push origin update-navigation.png>)

This pushes the commits from your local update-navigation branch up to GitHub.
You can think of it as publishing Tom’s work so other collaborators (in this case, “Jerry”) can see and use it.

After completing Tom’s workflow, you’ll now simulate Jerry’s contribution.  
To do this, you will:

- switch back to the `main` branch,
- pull in Tom’s latest changes,
- create a new branch for Jerry,
- make Jerry’s changes, then
- stage, commit, and push those changes to GitHub.

---
### 2. Jerry's Work

### 2a. Switch back to the `main` branch

```bash
git checkout main
```
This command moves you from update-navigation back to the main branch so Jerry starts from the main project line.

![alt text](<img/26. Git checkout main.png>)

Notice that the * is now on main branch meaning we are currently working on the main branch.

### 2b. Pull Tom’s latest changes into main
To achieve this, we run the below command:
```
git pull origin update-navigation
```
![alt text](<img/27. Git pull Tom's work.png>)

This ensures that you have the latest updates from the repository, including Tom's merged changes, if any.

### 2c. Create a new branch for Jerry’s work

```bash
git checkout -b add-contact-info
```
![alt text](<img/28. Git checkout add contact info.png>)

This creates a new branch called add-contact-info and switches you onto it.
Jerry will make his changes here, separate from the main branch until they’re ready to be merged.

### 2d. Make Jerry’s change in `index.html`

Open index.html and add some contact details (for example, an email address or phone number).

![alt text](<img/29. vi into indexhtml editor mode.png>)

![alt text](<img/30. vi into indexhtml file.png>)

This represents Jerry’s task of adding contact information to the page.

### 2e. Stage Jerry’s changes
To stage Jerry's changes run:
```
git add index.html
```

This stages the changes in index.html, telling Git to include them in the next commit.

### 2f. Commit Jerry’s changes
To commit Jerry's changes run:
```
git commit -m "Add contact information"
```
![alt text](<img/31. Git add and commit Jerry's changes.png>)

### 2g. Push Jerry’s branch to GitHub
To push Jerry's branch to GitHub, run:
```
git push origin add-contact-info
```
![alt text](<img/32. git push Jerry's work to github.png>)

This uploads the add-contact-info branch and Jerry’s commit to GitHub so it’s available in the remote repository for review and merging into the main project.


## Part 4: Merging Changes

### Overview

In this part of the project, **Tom and Jerry are collaborating on the same repository**.  
Each of them works on a separate branch so their changes can be reviewed before being merged into `main`.

In this section, I documented how **I created a Pull Request (PR) to add Tom’s changes from the `update-navigation` branch into the `main` branch**.

### Context

- **Tom’s branch:** `update-navigation`  
  Tom has been working on updating the navigation of the website on this branch.
- **Target branch:** `main`  
  This is the main project branch that will receive Tom’s changes after review.

Instead of merging directly, I used a **Pull Request** to follow a proper collaboration workflow.


### Steps I Took to Create the Pull Request for Tom

#### 1. Navigated to the GitHub Repository and switch to Tom's branch

I opened my web browser and went to the GitHub page for the project repository.

From the branch dropdown near the top-left of the file list, I selected **Tom’s branch**: `update-navigation`.

This confirmed that I was looking at the changes Tom had pushed.

![alt text](<img/33. merging changes.png>)

Notice that the branch have been switched as seen below:

![alt text](<img/34. switched branch.png>)

### 2. Create a pull request by clicking the contribute icon to show the open pull request from the drop down.
![alt text](<img/35. pull request.png>)

Github will take me to a new page to initiate a pull request. It will automaticcally select the main project's branch as the base and my recently pushed branch as compare branch.

### 3. Review Tom's changes
I will reach out to Tom to ensure he has reviewed his changes to ensure eberythign is correct. As Hithun shows the differences between the base branch and Tom's branch. It is a good opportunity for Tom to double-check his work.

#### 4. Set Base and Compare Branches

I made sure the branches were set as:

- **base:** `main`  
- **compare:** `update-navigation`

GitHub showed a message indicating that the branches were **able to merge**, meaning there were no merge conflicts between `main` and Tom’s `update-navigation` branch.

![alt text](<img/36. create pull request.png>)

### 5. Reviewing and Merging Tom’s Pull Request

After I created the pull request for **Tom’s `update-navigation` branch**, it became visible to the rest of the team on GitHub. Other team members can now:

- review the changes,
- leave comments, and
- request modifications if necessary.

Once all my team were satisfied that Tom’s changes were correct and stable, I (or another collaborator with merge permissions) proceeded to merge the pull request. This action incorporated all of Tom’s updates from the `update-navigation` branch into the `main` branch.

![alt text](<img/37. Pre merge request.png>)

![alt text](<img/38. Tom's work merge completed.png>)


This step demonstrates a typical DevOps collaboration workflow: work is done on a feature branch, reviewed through a pull request, and only then merged into the main project.

### 6. Updating Jerry’s Branch with the Latest Changes

After Tom’s changes were merged into `main`, **Jerry** also needed to prepare his work for merging. Jerry’s feature branch is called `add-contact-info`.

Before merging Jerry’s branch into `main`, it is important to make sure his branch is **up to date** with the latest changes from `main`. This reduces the chances of merge conflicts and ensures that Jerry’s work is compatible with Tom’s updates.

#### a. Steps I Took to Update Jerry’s Branch

1. **Switched to Jerry’s Branch**

   In the terminal, I first checked out Jerry’s branch:

   ```
   git checkout add-contact-info
   ````
  

2. **Brought in the Latest Changes from main**

With Jerry’s branch checked out, I updated it with the latest changes from main:

```
git pull origin main
```
This command fetched the latest changes from the main branch (including Tom’s updates) and merged them into Jerry’s add-contact-info branch. As a result, Jerry’s branch now had both:

- his own changes, and

- the most recent updates from main.



If there had been any merge conflicts at this stage, I would have resolved them, tested the project, and committed the resolved state before moving on to create a pull request for Jerry’s branch.

### 7. Finalising Jerry’s Contribution

After updating **Jerry’s** branch (`add-contact-info`) with the latest changes from `main` and resolving any potential conflicts, his work was ready to be pushed to GitHub and merged into the main project.

#### 1. Pushing the Updated Branch to GitHub

To upload Jerry’s latest changes (which now included both his own work and Tom’s merged updates from `main`), I ran:

```bash
git push origin add-contact-info
```

![alt text](<img/39. git push - Jerry.png>)

This command pushes the local add-contact-info branch to the remote repository named origin on GitHub.
After this step, the remote branch contains:

- Jerry’s feature changes, and

- the most recent updates pulled from main.

origin is the default name Git gives to the remote repository when the project is first cloned or when the remote is added.

#### 2. Creating a Pull Request for Jerry’s Branch

With the updated branch available on GitHub, I then:

1.  Went to the repository on GitHub.

2.  Opened the Pull requests tab and clicked on Compare & pull request.

![alt text](<img/40. Creating a pull request for J branch.png>)

3. Set:

- base: main

- compare: add-contact-info

4. Reviewed the diff to confirm that the contact-info changes and the latest navigation updates were all present.

5. Added a clear title and description explaining that this PR contains Jerry’s contact information feature, updated with the latest changes from main.

![alt text](<img/41. PR for Jerrys branch.png>)

6. Clicked Create pull request.

![alt text](<img/42. PR successful.png>)

#### 3. Merging Jerry’s Changes into main

Once the pull request had been reviewed and approved, I completed the process by:

Clicking Merge pull request, and

Confirming the merge.
![alt text](<img/43. confirm merge.png>)

![alt text](<img/44. Merge successful.png>)

At this point, the main branch contained contributions from both collaborators:

Tom’s navigation updates, and Jerry’s contact information feature.
