# Capstone Project: Enhancing a Community Library Website

## Background Scenario

In this capstone project I am part of a small development team working on the website for the **Greenwood Community Library**.  
The existing site already has some core pages:

- **Home**
- **About Us**
- **Events**
- **Contact Us**

The goal of the project is to make the site more engaging and informative for visitors. As a team, we decided to:

- add a new **“Book Reviews”** section, and  
- improve the **“Events”** page so it highlights upcoming community events.

To simulate a collaborative workflow, I use two contributor roles:

- **Morgan** – responsible for implementing the **Book Reviews** section.  
- **Jamie** – responsible for updating the **Events** page with new community events.

All work is coordinated using Git and GitHub.

---

## Objectives

- Practise **cloning a repository** and working with **branches** in Git.  
- Gain experience **staging, committing, and pushing** changes from multiple contributors.  
- Use **pull requests** to review changes and merge them into the `main` branch after resolving any conflicts.  
- Document the full workflow in a way that demonstrates real-world Git / DevOps collaboration.

---

## Setup

1. **Created a Repository on GitHub**

   - Repository name: `greenwood-library-website`.
   - Initialised the repo with a basic project structure and a `README.md`.

 ![alt text](<img/1. create new repo.png>)

 ![alt text](<img/2. new repo created.png>)  

2. **Cloned the Repository Locally**

   ```bash
   git clone https://github.com/<username>/greenwood-library-website.git
   ```
   ```
   cd greenwood-library-website
   ```

![alt text](<img/3. Clone the gitrepo to my local machine.png>)

![alt text](<img/4. git clone.png>)   


## Initial Tasks on the `main` Branch

Before introducing any new features, I prepared the basic pages for the **Greenwood Community Library** website on the `main` branch.

### Creating the Core Pages

Using git, I made sure the following files existed in the project:

- `home.html`
- `about_us.html`
- `events.html`
- `contact_us.html`

Each file received some initial placeholder content so the pages could render correctly in a browser.

Example:

```html
<!-- home.html -->
<h1>Welcome to Greenwood Community Library</h1>
<p>This is the home page of the library website.</p>
````
![alt text](<img/5. creating and updating the website files.png>)

### Committing the Initial Version

After creating and editing the files, I staged and committed the initial version of the website directly to the `main` branch:

```
git add .
git commit -m "Add initial library website pages"
git push origin main

```
![alt text](<img/6. stage, commit and push to main.png>)

This commit represents the team’s existing code base for the library website before any new features (Book Reviews or Events updates) were added.

## Morgan’s Work – Adding the Book Reviews Section

To simulate Morgan’s contribution, I created a dedicated feature branch and implemented a new **Book Reviews** page.

### 1. Creating Morgan’s Branch

From the `main` branch, I created and switched to a new branch called `add-book-reviews`:

```bash
git checkout -b add-book-reviews
```
This branch isolates Morgan’s work from the rest of the code base until it is reviewed and merged.

### 2. Adding book_reviews.html

On the add-book-reviews branch, I added a new file named `book_reviews.html` to represent the Book Reviews section of the site.
For this exercise I added simple placeholder content:
```
<!-- book_reviews.html -->
<h1>Book Reviews</h1>
<p>This page will showcase reviews of books available at Greenwood Community Library.</p>
```
![alt text](<img/7. Git checkout and new file.png>)

### 3. Staging, Committing, and Pushing Morgan’s Changes

After saving the new file, I staged and committed Morgan’s work with a clear message:
```
git add book_reviews.html
git commit -m "Add book reviews section"
```
Then I pushed the `add-book-reviews` branch to Github
```
git push origin add-book-reviews
```
![alt text](<img/8. Stagin commiting and pushing Morgans changes.png>)

At this point, Morgan’s feature branch was available on the remote repository and ready for review.

### 4. Raising a Pull Request for Morgan’s Work

On GitHub, I opened a pull request to merge Morgan’s changes into main:

Navigated to the repository on GitHub.

Clicked on the Compare & pull request banner for the add-book-reviews branch.

![alt text](<img/9. Compare and PR.png>)

Confirmed the branches:

- base: main

- compare: add-book-reviews

Entered a title such as: Add book reviews section.

Added a short description explaining that this PR introduces the new book_reviews.html page.

![alt text](<img/10. Create PR.png>)

This PR represents Morgan’s proposed changes to the project.

### 5. Reviewing and Merging Morgan’s Changes into main

After reviewing the diff and confirming that the new page looked correct, I completed Morgan’s workflow by:

1. ***Clicking Merge pull request.***

2. ***Confirming the merge in GitHub.***

This merged the add-book-reviews branch into main, so the Book Reviews section became part of the main Greenwood Library website code base.

![alt text](<img/11. PR for Morgan confirmed.png>)

![alt text](<img/12. confirm merge.png>)

![alt text](<img/13. Merge PR completed for Morgan.png>)

### Outcome

Morgan’s work demonstrated:

- Creating and working on a feature branch (add-book-reviews).

- Adding a new page to the project (book_reviews.html).

- Pushing changes to a remote branch.

- Using a pull request to review and safely merge the feature into the main branch.


## Jamie’s Work – Updating the Events Page

After Morgan’s Book Reviews feature was merged into `main`, I simulated **Jamie’s** contribution by updating the **Events** page on a separate branch.

### 1. Creating Jamie’s Branch

Starting from the `main` branch (which now includes Morgan’s changes), I created Jamie’s feature branch:

```
git checkout main
git pull origin main     # make sure local main is up to date

git checkout -b update-events
```
![alt text](<img/14. creating Jamie's branch.png>)

The `update-events` branch is where Jamie’s changes to the Events page are developed.

### 2. Updating events.html

On the update-events branch, I edited events.html to include new and more detailed community events for the Greenwood Community Library.

placeholder content:

```
<!-- events.html -->
<h1>Upcoming Community Events</h1>
<ul>
  <li>Monthly Book Club – First Thursday of every month</li>
  <li>Children's Story Time – Every Saturday at 10am</li>
  <li>Author Meet & Greet – Last Friday of the month</li>
</ul>
```
![alt text](<img/15. updating events file.png>)

### 3. Staging, Committing, and Pushing Jamie’s Changes

Once the updates were made, I staged and committed Jamie’s work:
```
git add events.html
git commit -m "Update events page with upcoming community events"
git push origin update-events
```
![alt text](<img/16. staging, commiting and pushing Jamie's changes.png>)

At this point, Jamie’s `update-events` branch – with the updated Events page – was available on GitHub.
![alt text](<img/17. updated events page on update-events branch.png>)

### 4. Keeping Jamie’s Branch in Sync with `main`

Before raising a pull request, I made sure that update-events contained the latest changes from main (including Morgan’s Book Reviews work). From the update-events branch I ran:
```
git pull origin main
```
This pulled the latest main branch changes into update-events.
If any conflicts had appeared, I would have resolved them, tested the project, and then committed the resolved changes.

![alt text](<img/18. keeping Jamie's branch in synch with main.png>)

### 5. Creating a Pull Request for Jamie’s Work

On GitHub, I opened a pull request to merge update-events into `main`:

Opened the repository and clicked the Compare & pull request banner for update-events.

Confirmed the branches:

- base: main

- compare: update-events

Used a clear title, such as Update events page with new community events.

Added a description summarising the changes on the Events page.

![alt text](<img/19. PR for Jamie's work.png>)

### 6. Merging Jamie’s Changes into main

After reviewing and confirming that the new events information looked correct, I completed Jamie’s workflow by:

- Clicking Merge pull request, and

- Confirming the merge on GitHub.

![alt text](<img/20. Merge PR.png>)

![alt text](<img/21. Merge request completed.png>)

Now the main branch includes contributions from both collaborators:

- Morgan’s Book Reviews section, and

- Jamie’s updated Events page.

### Outcome

Jamie’s part of the capstone reinforced the core Git collaboration workflow:

- Working on a dedicated feature branch (update-events),

- Pulling the latest changes from main to stay in sync,

- Pushing updates to a remote branch, and

- Using a pull request to safely integrate changes into main.

## Lessons Learned / Reflection

This capstone project gave me a practical, end-to-end experience of using Git and GitHub in a collaborative setting.  
By simulating the roles of **Morgan** and **Jamie**, I was able to see how multiple contributors can safely work on the same code base without breaking the `main` branch.

### Git & Collaboration

- **Branching for isolated work**  
  Creating feature branches (`add-book-reviews`, `update-events`) allowed each contributor to work independently. I learned how this keeps `main` clean and stable while still enabling parallel development.

- **Clear commit messages**  
  Writing messages like `Add book reviews section` and `Update events page with upcoming community events` made the history easy to understand. This is crucial when reviewing changes or debugging later.

- **Pull requests as a review gate**  
  Using pull requests to merge Morgan’s and Jamie’s branches into `main` showed me how PRs act as a checkpoint for discussion, code review, and approval before changes are integrated.

- **Keeping branches up to date**  
  Pulling from `main` into `update-events` before raising Jamie’s PR demonstrated the importance of syncing feature branches with the latest code to reduce merge conflicts.

### DevOps Mindset

- **Single source of truth**  
  Treating the `main` branch as the single reliable version of the project mirrors real DevOps practices, where production (or main) must remain stable.

- **Small, incremental changes**  
  Each branch focused on one clear feature (Book Reviews, Events page updates). This matches the DevOps preference for small, reviewable changes that are easier to test and roll back.

- **Documentation as part of the work**  
  Writing this README alongside the code made me think about *why* I was running each command, not just *how*. It also makes the project easier for others (or future me) to understand.

### Next Steps

To build on this capstone and deepen my Git/DevOps skills, I plan to:

1. **Introduce basic automation**  
   Add a simple GitHub Actions workflow (for example, an HTML/CSS linter or a basic build check) that runs automatically on every pull request.

2. **Experiment with more complex branching strategies**  
   Try scenarios with multiple concurrent features and deliberate conflicts, then practise resolving them and documenting the decisions.

3. **Apply the same workflow to a real application**  
   Use this branching + PR model on a small backend or frontend project, integrating testing and deployment steps as part of a CI/CD pipeline.

Overall, this capstone helped me move from just “knowing Git commands” to understanding how Git, branches, and pull requests fit into a real collaborative workflow that is essential in modern DevOps teams.
