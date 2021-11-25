# Introduction to Continuous Delivery with GitLab

<center>

![Introduction to Continuous Delivery with GitLab](img/cover400.jpg)

</center>

The tutorial will give you a quick impression of how teamwork is done with GitLab. 
Generally, it's easier to start practicing DevOps/CD with GitLab than with other products because GitLab is an all-in-one solution.

💻 During the tutorial, we will
- set up basic project management using GitLab;
- create continuous delivery pipeline;
- perform a couple of GitLab Flow cycles;
- study CI/CD metrics in GitLab.

📖 It's good but not necessary to know
- Git;
- Node.js;
- React;
- Docker.

I will ask you to do something from time to time. Such actions are marked with  🛠️ emoji. Please perform such actions as you read the text to take the most from the tutorial.

## 🏁 Intro and getting familiar with the project

We will use a slightly modified **create-react-app** project as a guinea pig.

>Why React? First, it's is the most widely adopted JavaScript UI library, and many readers are familiar with it. Second, **create-react-app** gives us sensible build and test stages which are already implemented.

🛠️ Let's now clone the repository with the code we are going to work with.

```
git clone https://github.com/ntaranov/gitlab-cd-react
```
🛠️ Navigate to the local repository folder
```
cd gitlab-cd-react
```

>You can just create new **create-react-app** application instead, but the version in the repo contains some fixes, and I tested the package versions.

🛠️ Install the **npm** packages 
```bash
npm install
```
>Use `npm ci` instead if you encounter an error running `npm install`

🛠️ Try to "build" the project
```bash
npm run build
```
Note that `./build` directory now has relevant files including the minified JavaScript and CSS. 

🛠️ Then run the tests
```bash
npm run test -- --coverage --watchAll=false --forceExit
```

🛠️ Now we need to deploy to finish, but to avoid cluttering the tutorial with web server installation instructions, let's just launch the debug web server to see how our web application looks in browser. 
```bash
npm start
```
Standard **create-react-app** starter page should appear.

If you think about it, all that remains is to replace the last 
instruction with copying the static files from `./build` directory to a web server, and the deployment is done. Basically, this is enough to work by yourself, however, if you want to work as a part of a team, you might also need at least the following:
- centralized repository with
  - control over team members' permissions;
  - code review;
- bug tracker to
  - plan work and
  - track the progress;
- build system with
  - pipelines launched automatically as needed;
  - build status visualization;
  - artifacts storage;
  - automatic notifications about key events;
  - automatic deployment into different environments;
- continuous deployment metrics collection.

All the above, and much more, is supported by Gitlab. [The paid version](https://about.gitlab.com/pricing/) offers more features, but the free version available on [GitLab.com](https://gitlab.com) is enough for what we are after.

>What sets GitLab apart is the absence of restrictions on self-hosted job runners, however some pretty basic "management" and "corporate" GitLab features like mandatory approval for merge requests are considered premium offerings. Open source projects have access to all the GitLab features for free.

## ⚙️ Setting up basic project management on GitLab.com

In this section we will create the GitLab project itself and issues we will work on later, and will set up a Kanban board to visualize the project status.

### 💼 Creating a project in GitLab

We will use GitLab.com to avoid the chores inevitable when installing and setting a self-hosted instance.

🛠️ Please browse to `https://gitlab.com` and register an account if you don't have one yet.

GitLab lets us create a project by just doing `push` to a remote repository.

🛠️ Let's use the following command for this:
```
git push https://gitlab.com/<user name>/gitlab-cd-react
```
`<user name>` here is our GitLab.com username.

This will create a private project named `gitlab-cd-react` in your GitLab.com account.

🛠️ Please navigate to `https://gitlab.com/<user name>/gitlab-cd-react`.

After, we will set up the Kanban board and create several issues to have something to collect data about.

### 🗃️ Creating issues and setting up a board

Let's start with creating an issue.

🛠️ Click  **Issues** in the left side menu, then press one of the **New Issue** buttons in the same area. An issue "create" form will open. Specify "Create issues for the tutorial" as the title.

Specify the following text in description:
```
Create issues with the titles listed below.
- [ ] Create labels
- [ ] Set up board
- [ ] Create continuous delivery pipeline
- [ ] Perform GitLab Flow iterations
- [ ] Study the metrics
```

Yes, these are the things we are going to do. Leave default values for all the other fields. Press **Submit issue** button.

Please note that the list with items starting with square brackets was recognized as a checklist and its items as to-dos. GitLab, as many other platforms, uses **Markdown** for text formatting.

🛠️ Assign `Create issues for the tutorial` issue to yourself. To achieve this click **Edit** in the panel with text **0 Assignees** on the right in the issue "edit" form.

Congrats, now you are the assignee. Actually, the issue could have been assigned to any team member, but as nothing would really be different, we don't spawn the unnecessary extra accounts.

🛠️ Now, let's actually create all the issues listed in the description of the first one. 

Only specify titles, leave default values for all the other fields. You can "check the boxes" in the `Create issues for the tutorial` issue description either as you create the other issues, or all at once when you are done with them.

There are different ways to create issues. When creating the issues from the list, find and use at least 3 different ways to do it. You should have 6 issues total in the end.

🛠️ Close `Create issues for the tutorial` issue. You can do this via **Issues** -> **List** or **Issues** -> **Board**. Click the issue's title and press **Close issue** button in the issue edit form.

### 🎢 Our workflow
Workflow is a sequence of stages a task passes to be considered "done". In Continuous Delivery, Kanban or DevOps, the task moves through some sequence of stages forward or might be returned to one of the previous ones.

>This assumes linear value streams. You can [read about value streams here](https://en.wikipedia.org/wiki/Value-stream_mapping#Purpose_of_value-stream_mapping). Reworking sometimes pretty baroque workflow diagram into a sequence of stages might be a tough management task, solving which is far beyond the scope of the article.  

We will use the simple stages' sequence.

- Open
- Dev
- Dev: done
- QA
- Closed

In the beginning, an issue is in **Open** stage. Then, it's pulled into work by developers and moved to **Dev** stage. After the work is done, the issue is placed into the stage **Dev: done**.

>Why do we need the extra stage? This is because in [Lean](https://en.wikipedia.org/wiki/Lean_manufacturing), which all the methodologies like CD, Kanban, and DevOps are based on, the next work center pulls work items when it's ready as contrasted with the methodologies where work items are handled to the next work center by the previous.
This makes it possible to discover work items waiting to be processed, so that work items accumulating before some stage reveal a bottleneck in our pipeline.

Subsequently, the issue is pulled into **QA** stage where quality assurance and autotests do their job, and **QA** "graduates" are considered so good that they are deployed straight to production.

>"That's not how it's done, you know, that's one-two-production, there have to be other environments like Staging in the process!" you might say. I totally agree, but here we make use of a pipeline as simple as possible both to implement and to understand. Otherwise, we might find ourselves burrowed under complexities originating, say, from Kubernetes.

### 🏷️ Setting up labels

GitLab uses **labels** to organize issues. Labels are like tags and let you search, filter, and display issues in different places based on the existence of one or another label.

🛠️ Open the issue `Create labels` and assign it to yourself. Select **Project information** -> **Labels** in the left menu, then click **New label** in the right part of the main area. Create so the following 3 labels picking colors you prefer.
- Dev
- Dev: done
- QA

The names of the labels here are arbitrary and don't carry any special sense from the GitLab's perspective.

🛠️ When done, assign the status **Closed** to the issue `Create labels`.

Now everything is ready for setting up the Kanban board.

### 📝 Setting up board

GitLab has [Kanban boards](https://docs.gitlab.com/ee/user/project/issue_board.html). 
They help show different issues according to their labels in a meaningful way. Despite the name, the boards can be used not just to implement Kanban, but also Scrum, and other methodologies. We even already have an issue dedicated the boards.

🛠️ Assign the corresponding issue to yourself. 

🛠️ Select **Issues** -> **Boards** in the left menu. You'll see the default board. It already have **Open** and **Closed** list columns. Press **Add list** in the top right part of the main page area. Create one list for each of the labels created earlier.
- Dev
- Dev: done
- QA

If you created them in a different order, you can just drag the lists around by titles to change the order. 

🛠️ On the **Issues** -> **Boards** page, drag the `Set up board` issue from the  **Open** column to the **Dev** column. Select the issue clicking the part of the card, not occupied with any text.

This way we indicate that we started working on the issue. Note that the label **Dev** has just been automatically assigned to the issue.

🛠️ As we in fact have already done all the work for the issue, move the issue to the **Dev: done** column. 

We are closing to the stage **QA**. Try to impersonate a tester, think about how unreliable developers are, and, optionally, take a small appliance apart with a screwdriver.

🛠️ Then move the issue to **QA** stage. Here you can drag the issue from column to column back and forth several times. By the way, such adventures in fact eventually occur with issues at this stage. In the process, the label for a new column will appear and the label for the previous column will disappear for the issue. Finally, leave the issue in the **Closed** column and note that this action has closed it.

Another useful UI is [To-Do List](https://gitlab.com/dashboard/todos)(there is a submenu new the login menu). This is the [list of actions](https://docs.gitlab.com/ee/user/todos.html) expected from you for some reason. We will not dwell on it in detail, but I recommend checking it during the tutorial from time to time to get an idea of how it works.

## 🚀 Continuous delivery pipeline

In this section, we will implement Continuous Delivery using GitLab.

### 🏗️ Implementing a CD pipeline using GitLab

To actually run pipeline code, **GitLab.com** uses shared runners that implement [docker executors](https://docs.gitlab.com/runner/executors/). If you register your own agents, you have much more flexibility in [choosing build environments](https://docs.gitlab.com/runner/executors/).

Let's now build our pipeline. 

🛠️ Assign issue `Create continuous delivery pipeline` to yourself and transfer it into **Dev** stage.

🛠️ Let's author the CI/CD pipeline. For this purpose, we need to create a file called `.gitlab-ci.yaml`. At the time of this writing, it could be done like this.

1. Browse to **gitlab-cd-react** project.
2. Either on **Project overview / Details** or **Repository** page you will see files in your repository.
3. Select **New file** option as shown in the screenshot. You will be redirected to the file creation page.
   ![New file](img/new-file.png)
4. In the **File name** window, enter the name of the file `.gitlab-ci.yml`, the **Template** field to the right will be filled in automatically, another **Apply a template** field will appear. Check out the available options. Finally choose `Bash` - it's just a template with **bash** commands, we'll use that as a starting point.

Let's take a look at the resulting file. The official document on the `.gitlab-ci.yml` file format is [here](https://docs.gitlab.com/ee/ci/yaml/gitlab_ci_yaml.html).

>The format that GitLab uses to define build pipelines is one of the simplest and most intuitive in the industry.


```YAML
image: busybox:latest

before_script:
  - echo "Before script section"
  - echo "For example you might run an update here or install a build dependency"
  - echo "Or perhaps you might print out some debugging details"

after_script:
  - echo "After script section"
  - echo "For example you might do some cleanup here"

build1:
  stage: build
  script:
    - echo "Do your build here"

test1:
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"

test2:
  stage: test
  script:
    - echo "Do another parallel test here"
    - echo "For example run a lint test"

deploy1:
  stage: deploy
  script:
    - echo "Do your deploy here"
```
To get the pipeline we need, we slightly modify this file.
```YAML
image: busybox:latest
```
`image` specifies which docker image will be used to run build jobs. GitLab uses Docker Hub by default, but this is configurable. We need an image with **Node.js** preinstalled.

🛠️ Use
```YAML
image: node:14-alpine
```

`before_script` and` after_script` are executed before and after each job, respectively. In the process of running the jobs, we will use Node.js modules.

🛠️ We organize caching of modules in order not to download modules every time from the Internet.
```YAML
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .npm/

before_script:
  - npm ci --cache .npm --prefer-offline
```
- We first indicate where the jobs will look for the cache, namely - inside the current folder in the `.npm` subfolder.
We use `${CI_COMMIT_REF_SLUG}` as the cache key, which means we keep a separate cache for each Git branch.
- In `before_script`, we install packages according to `package-lock.json`, specifying the `.npm` subfolder as the path for the cache.

Thus, further in the `.gitlab-ci.yml` code, we can assume that the packages are already installed.

We will not need `after_script` in this tutorial.

Let's move on to the `build1` section, initially there should be something like this:
```YAML
build1:
  stage: build
  script:
    - echo "Do your build here"
```
Let's take a look at the `stage: build` chunk. `stage` defines the pipeline stage, the stages are executed one after another. One stage can contain one or more jobs that can be executed in parallel.

🛠️ Let's change the commands to actually build something:
```YAML
build1:
  stage: build
  script:
  - npm run build
  artifacts:
    expire_in: 1 week
    paths:
    - 'build/'
```
- `npm ci` installs the required modules according to the `package-lock.json` file without checking if they already exist in the `node_modules` folder.
- `npm run build` runs actual build of the minified React application.
- `artifacts` specifies which files should be saved for future use. By default, **webpack** with the default **create-react-app** settings copies the files into `build` folder. 
- We use `expires_in` to avoid wasting space on builds that are no longer needed.

Now let's review the piece of code for the stage `stage: test`
```YAML
test1:
  stage: test
  script:
    - echo "Do a test here"
    - echo "For example run a test suite"

test2:
  stage: test
  script:
    - echo "Do another parallel test here"
    - echo "For example run a lint test"
```
🛠️ We do not need the job `test2`, but we will change the job `test1` as follows:
```YAML
test1:
  stage: test
  script:
  - "CI=true npm test" 
  dependencies:
  - build1
```
- `npm test` here runs the unit tests defined in our project.
- `dependencies` indicates that the given job `test1` depends on the results of the job `build1` so that the artifacts of the job are available in this stage.

Let's move on to the part related directly to deployment.
```YAML
deploy1:
  stage: deploy
  script:
    - echo "Do your deploy here"
```
This part is usually tied to the specifics of the environment in which the deployment is carried out and technically complex. GitLab supports deploying to Kubernetes with very little effort. The alternative is to implement the deployment logic by ourselves. We'll leave the technical details of this process for future articles. The focus of this article is on explaining the basics of working with GitLab, so we will resort to a trick: we will use the fact that our React application is technically a static website and deploy it to GitLab Pages, which are available to anyone with an account on GitLab.com.

🛠️ Replace job `deploy1` with this code.
```YAML
pages:
  stage: deploy
  script:
  - mv public _public
  - mv build public
  only:
  - master
  artifacts:
    paths:
    - public
  dependencies:
  - build1
```

We renamed `deploy1` to `pages` because GitLab, by the name of the job, understands that it is required to deploy the files available to this job to GitLab Pages.
Next, we do 2 things.
- `mv public _public` saves the `public` folder generated from the **create-react-app** template. We do this because GitLab will return the contents of the folder named `public` in response to requests to Pages.
- `mv build public` - here we just place the build result where the GitLab Pages web server will look for it.

Let's complete the work on our pipeline.

🛠️ Commit the edited file `.gitlab-ci.yml`, specify "Add CI/CD pipeline" as **Commit message**, and leave `master` as the branch.

🛠️ Click **CI/CD** in the left menu to observe our pipeline execution. You can see the run details and logs.

🛠️ Move the `Create continuous delivery pipeline` issue to the **Dev: done** stage.

## 🎠 A few rounds of work with GitLab Flow

Git is a very flexible version control system and can be used in very different ways. If each team member uses Git in his own way, confusion arises when it is difficult to understand the logic of other people's actions from the history of changes, and nobody understands what to expect from the others. Measuring performance isn't even really possible in a situation like this. In order to avoid such confusion, team member usually agree on uniform rules for working with Git, and such an agreement is called Git workflow.

Let's do the following to study the implementation of GitLab Flow in GitLab:
- discuss what Git workflow is in general;
- see how access rights and protected branches are implemented in GitLab;
- figure out how merge requests work;
- do several iterations of GitLab Flow.

### ➡️ GitLab Flow

In short, [GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html) is:
- every time you need to change something in the code, a branch is created from the main (in Git, the default is `master`);
- then a merge request is opened(called a pull request in other systems) where the changes are discussed;
- there is a separate branch for each of the environments (test, production, etc.);
- one of the "environment branches" from where the unconditional deployment takes place may coincide with the main one;
- deployment occurs by merging into a branch of the corresponding environment;
- merges happen from a feature branch to the main branch, or from a "less production" "environment branch" to a "more production" branch.

>Actually, any Git workflow that supports trunk based development will suffice to implement CD, and we could implement any such workflow with GitLab. However, as GitLab recommends using it, and to keep things simple, we use GitLab Flow.

### 🔐 Access levels in GitLab

There are different [access levels](https://docs.gitlab.com/ee/user/permissions.html), ordered from the highest to the lowest.
- **Instance administrator** - available only for self-hosted installations, can do anything.
- **Owner** - the owner of a group of projects, can do anything except purely technical things such as enabling/disabling features and integrating with other services.
- **Maintainer** - can do anything on the project level and below except some actions related to the entire project, such as changing its name or visibility, as well as destructive actions such as deleting issues.
- **Developer** - can do anything **Maintainer** can, except some administrative and destructive actions inside projects, such as setting the branches protection and editing comments.
- **Reporter** - can edit issues, but cannot make changes to the repository.
- **Guest** - has read-only access to issues except confidential, and can create new issues.

The permission levels are given here for completeness, we will not actively use them in the tutorial.

### 🔀 Merge requests

[Merge requests](https://docs.gitlab.com/ee/user/project/merge_requests/getting_started.html) are the main way to make code changes when using GitLab. Changes are made to the code, then a merge request is created by the author of the changes. The merge request is then discussed, the code review takes place, and, as a result, is accepted, sent for revision or rejected.

🛠️ Go to the issue `Perform GitLab Flow iterations`, assign it to yourself and move it to the **Dev** column.

🛠️ Move the task `Create continuous delivery pipeline` to **QA** stage.

>Yes, this is how we will test our pipeline, but you should approach testing more responsibly on real projects. Do as I say, not as I do 😉

Our application is useless. This is not uncommon as there are many other useless applications on the Internet. To take it to the next level, let's make our application toxic. There are 2 features that are proven to help with this.

🛠️ Open `src/App.js` file for editing (**Repository** -> **Files**, **Edit**) and add this line right at the top of the file.
```jsx
import {useEffect} from 'react';
```
For **Commit message** specify, for example, "Add React imports". Leave the **Target branch** as `master` and hit **Commit changes**.

Oops, we just committed the changes directly to `master`. Some workflows allow this, but we will comply with GitLab Flow. A GitLab feature called **protected branches** will help us to avoid such incidents in the future. Its purpose is not to control access, but to help team members work within the agreements and to avoid accidentally changing or deleting data in the repository.

🛠️ Click in the left menu **Settings** -> **Repository**. In the page that opens, find the **Protected Branches** section.

You will see that only the `master` branch is protected by default, but you can add others. By default
- **force push** is forbidden to all;
- **push** allowed for Maintainers;
- **merge** allowed for Maintainers.

🛠️ Change **Allowed to push** to **No one**.

🛠️ Great, let's go back to editing `src/App.js`. After the line
```jsx
function App() {
```
and before the line
```jsx
  return (
```
insert
```jsx
  useEffect(() => {
    alert('Consent to cookies and everything!');
  }, []); 
```
The corresponding piece of code should now look like this
```jsx
// ...other code
function App() {
  useEffect(() => {
    alert('Consent to cookies and everything!');
  }, []); 

  return (
// ...other code
```
Specify "Add the annoying popup" as the commit message.

You will see that you are prompted to add a commit to a certain branch and a certain branch name has already been automatically generated. We will change this name in the future to something more meaningful and consistent with our Git workflow, but for now, let's do a little experiment.

🛠️ Replace the suggested name with `master` and click **Commit changes**.

You will see the error message `You are not allowed to push into this branch`. Good! Branch protection works. Let's now initiate the procedure for making changes to the code as it is supposed to be done.

🛠️ Now change the branch name from `master` to `feature-cookies-consent`. Leave the **Start a new merge request with these changes** option checked. A branch with the specified name will be created and the changes will be committed to that branch. You will be taken to the merge request creation page. Leave
- the default merge request title.
- `Unassigned` for **Assignee** and **Reviewer**.
- everything else as default.

Click **Submit merge request**

By the way
- **Assignee** is responsible for working on the merge request and merging it into the target branch,
- **Reviewer** examines the proposed changes and can approve them if they look correct.

After you submit the merge request, you will be taken to its page. Let's explore it. In the main area, we see the following.
- Which branch we are merging to which (`feature-cookies-consent` to `master`).
- The pipeline status for the `feature-cookies-consent` branch.
- Ability to approve the merge request on behalf of the current user.
- The **Merge** button to accept the merge request and the merging of the branches, which results in changes in `master`.
- The area where you can see the commits that will be merged; you can change the merge commit message there.
- Ways to take part in the discussion:
  - express your attitude and leave a smile;
  - add a comment.
- The **Close merge request** button, which allows you to close the merge request without accepting changes to the `master` code.

>For free subscriptions there is only the possibility of "optional" approval of a merge request, in paid versions it is possible to make the approval mandatory.

If you switch to the **Changes** tab, you can see what changes are planned to the code. Here you can also create a comment that will refer to a line of code.

Merge requests also support labels that you can use to make them easier to find.

By this time, the pipeline should have finished and... the tests have failed. In our particular case, this is because we used `window.alert`, which is a browser object and our unit tests running in Node.js environment do not have access to it. 

For Continuous Delivery, a good set of automated tests is required because we rely on them for **QA** almost completely, and completely in the case of Continuous Deployment. Keeping such a test suite up to date is the main technical challenge when implementing such methodologies.

🛠️ Let's fix `src/App.js` by adding a check that the code runs in the browser. Let's put the code that uses `window.alert` inside the code block that verifies that we are in the browser. Make sure you use `feature-cookies-consent` when you edit the file. It should look something like this
```jsx
// ...other code
function App() {
  useEffect(() => {
    if (typeof process === 'undefined' || process.release === undefined) {
      alert('Consent to cookies and everything!');
    }
  }, []); 

  return (
// ...remaining code
```
Add "Ensure running in a browser" as the commit message. Target branch should be `feature-cookies-consent`.

🛠️ Let's go back to the merge request `Add the annoying popup` and click the **Merge** button and commit the changes to the code. Leave the **Delete source branch** checkbox checked.

When using GitLab Flow, feature branches are traditionally removed after merge. This allows to avoid cluttering the system with no longer relevant branches and makes it possible create a branch with the same name in case of an issue being sent for revision.

🛠️ Open **Repository** -> **Graph** and notice that `feature-cookies-consent` is now merged into `master`.
Then open the `src/App.js` file in the `master` branch and notice that the code now contains our edits.

🛠️ Go to the issue `Perform GitLab Flow iterations` and click the down arrow next to the **Create merge request** button in the main area. Specify the name of the branch you want to create in order to base a new merge request on it. `feature-notifications-consent` is fine. Leave the source branch as `master`.

When creating a merge request like this, the issue will be closed automatically as soon as we merge the code into the main branch.

Note that the auto-generated title of our merge request starts with `Draft:`. This means that [merge request is marked as draft](https://docs.gitlab.com/ee/user/project/merge_requests/drafts.html). This is useful for authors to indicate that the work is in progress and to avoid merging the changes "as is" as a result of misunderstanding. The same can be achieved with the **Mark as draft** button in the upper right part of the main area.

Let's add push notifications to our website to make it look modern.

🛠️ Let's edit the code slightly differently for a change. On the merge request page, click the **Open in Web IDE** button. An interface that is slightly more convenient for working with code will open. From there, open the `src/App.js`. Add the code
```jsx
      Notification.requestPermission().then(function(result) {
        alert(`You ${result} notifications`);
      });
```
before the line which we added earlier
```jsx
      alert('Consent to cookies and everything!');
```
🛠️ Click the **Commit** button at the bottom left. You will see the commit interface, specify the commit message "Add the notifications users want". Leave everything else as default and hit the **Commit** button.

🛠️ Go to merge request, for example, using the submenu at the top right of the screen next to the login menu. Click the **Mark as ready** button at the top right of the merge request form. Click the **Merge** or **Merge when pipeline succeeds** button depending on whether the pipeline has already completed.

🛠️ Open our website and see if the new "features" work and our page feels like most modern sites on the Internet. Normally, GitLab Pages serve pages from `https://<user name>.gitlab.io/gitlab-cd-react` where `<user name>` is your GitLab.com username.

🛠️ If everything is fine, drag `Create continuous delivery pipeline` issue to the **Closed** stage in the board.

So, we went through several full work cycles, and now we can study the metrics that GitLab collected in the process.

## 📈 CI/CD Metrics in GitLab

During teamwork, it is useful to collect statistics to understand whether the productivity is increasing or decreasing. The way to measure performance can be very different depending on the nature of work and the type of project or product.

>For example, a web studio may have "typical" tasks such as coding a page layout as a part of a standard package of services offered to clients. An agile startup may be at the stage when the product vision is changing with a growing understanding of the customer and the market, and tasks can be difficult to predict and often unique. Still, we can come up with purely technical performance indicators such as time to market(TTM) or just the duration of a particular stage. Tracking the dynamics of such indicators can be very useful: over a  long enough period of time, this allows you to understand how the performance is changing.

The good news is that GitLab has the ability to collect the statistics. Let's check out the functionality.

🛠️ Assign `Study the metrics` issue to yourself and drag it to **Dev** stage.

🛠️ Click on **Analytics**. By default, the [**Value stream** section](https://docs.gitlab.com/ee/user/analytics/value_stream_analytics.html) will open. In Lean in general, and in DevOps in particular, it is believed that tasks ultimately bring some benefit to the customer. And  movement of such useful tasks, from the stage of inception through all stages of work to the moment when the results become available to the customer, is called the value stream.

Main average values ​​by stage:
- **Issue** - the time it takes to "take issue into work" i.e. label or add to a Milestone.
- **Plan** - the time from the last action in the previous stage till the first commit appears in the branch, at least one of the commits of which is associated with the same issue. That is, "the time it takes to start committing".
- **Code** - the lifetime of a branch associated with a particular issue, which passes before the merge request appears.
- **Test** - the time from the start to the end of all pipelines for this project.
- **Review** - the time from the creation of the merge request to its merge or closure.
- **Staging** - the time from accepting a merge request to deploying to a production environment.

If you add up the duration of **Issue**, **Plan**, **Code**, **Review**, and **Staging**, you get roughly time to market (TTM).

At the top of the page, you will also see some aggregate indicators for the project for the selected period of time.

**Analytics** -> **Repository** shows different graphs related to commit languages, code coverage metrics(if configured), distribution of commits over time(month, days of the week, hours).

**Analytics** -> **CI/CD** shows summary data on executed pipelines, including a graph showing the dynamics of changes in the pipelines "successful" executions.

🛠️ Close `Study the metrics` issue.

## 🎉 Congratulations!

You've walked through implementing CD with GitLab starting from issues and boards and finishing with the metrics.

If you noticed any problem with the course or know how to improve it, please do create an issue [in the course starter repository](https://github.com/ntaranov/gitlab-cd-react).

## Links
1. [GitLab product handbook](https://about.gitlab.com/handbook/product/gitlab-the-product)
2. [The DevOps Handbook](https://www.amazon.ca/DevOps-Handbook-World-Class-Reliability-Organizations/dp/1942788002)
3. [Pro Git Book](https://git-scm.com/book/en/v2)
