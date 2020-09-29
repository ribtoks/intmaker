---
title: How to return to the flow faster
date: 2019-08-22T21:34:29-08:00
author: "Taras Kushnir"
image: agenda-concept-development-7376.jpg
categories:
  - Programming
keywords:
  - gtd
  - time management
aliases:
  - /2019/how-to-return-to-flow
---

Hobby projects are very important for me. They give me the freedom to do what I want, how I want it and also they help me to develop quite solid expertise in various fields. Some smaller, some bigger, they always require unattended focus and precious time to work on them. It's always hard to find that time. During the weekend I try to get outside, my evenings are busy with sports or family and during the night I prefer sleeping to coding. **The only time left is 1-2 hours in the early morning before I go to work.**

I've always struggled to use this time as efficiently as possible. **It's hard to open the laptop and continue where you stopped yesterday or before the weekend right away.** It takes time to remember how to proceed from the last stop. I tried various ways and in this post I'd like to share the setup I'm using right now that helps me minimize this catch-up time significantly.

<!--more-->

First of all I tried to answer the question what exactly information I need to continue from where I stopped. While I might remember some bits and pieces of what I did last night, on Monday morning it's almost impossible to remember what I was doing on Friday. So after all, **I found these 3 questions to be the most important**:

- what global problem I was trying to solve?
- what was already done in order to solve this problem?
- what should be done next in order to solve this problem?

As I write it they seem to be quite obvious, however, it was an important step to ask the right questions, because they helped me to find the solution.

<br />

## What global problem I was trying to solve

This one was the easiest. Having typical kanban board in GitHub with "TODO", "In Progress" and "Done" and keeping it up to date pretty much resolves this bullet item. Since I'm a single developer, there wouldn't be much stuff "In Progress" so even a quick glance will remind me what in general I was doing.

![GitHub Project](/img/github-kanban.png)
*GitHub "Project" kanban board from one of my projects*

Also the issue I'm working on now can have some details in the description or in comments that are also 1 click away from this screen.

<br />

## What was already done in order to solve this problem

Typically this question should be easily answered by source control system. Just typing `git status` and `git log`, however, is not enough. I might be interested in changes in some particular files done recently so a more sophisticated dashboard will be handy.

What first comes to my mind is of course to use `gitk` - built-in, that provides you with history, diffs and changed files.

![gitk UI](/img/gitk.png)
*gitk run from my project's root*

I wouldn't say that `gitk` is the quintessence of GUI evolution of 21st century, but it certainly solves my problem. However, usually I work from terminal more than from the UI and I was hoping to find a command line tool. 

I even attempted to write my own tool using [gocui](https://github.com/jroimartin/gocui) library that will be an interactive dashboard, when I found [lazygit](https://github.com/jesseduffield/lazygit). Probably author had totally different vision how this tool should be used, but for me it makes just perfect command line git dashboard!

![lazygit UI](/img/lazygit.png)
*lazygit UI*

You get everything from `gitk` but, in my opinion, simpler and organized better. You have a single large area that shows `git log` tree when you go through branches or file diff when you go through files changed in a commit. Also it shows you `git status` and lists latest branches you worked on. This is exactly what I needed.

<br />

## What should be done next in order to solve this problem

This one was the hardest. I tried various approaches before I found the one.

### OneNote pages

This was the system I used initially. For everything which was significant to mention, but not significant enough to become a GitHub Issue, I was writing down in OneNote. When the list grew over 1 page so I needed to scroll, I started another page.

![OneNote list](/img/onenote-todo.JPG)
*First primitive but working wolution*

With time I was migrating most of them to GitHub Issues.

### GitHub subtasks

Theoretically you can have subtasks in main GitHub issue, but that scales only up to a certain size of the issue.

![github subtasks](/img/github-issue-subtasks.png)
*Subtasks grow quite significantly if your issue is a huge feature*

Of course, you can and should split big issues into smaller ones, but that only scales up to some level. Issue is also an incapsulation of an atomical task that at some point cannot be divided further. But the list of subtasks can still be huge if you keep track of some imporant implementation details of your task. **It is, however, fully manual work to go and update subtasks** each time you have an idea or finish one.

### Puzzle-driven development

There was an [interesting concept](https://www.yegor256.com/2010/03/04/pdd.html) from Yegor Bugaenko. The gist is that all issues in the repository are supposed to be very small (like 30 minutes up to an 1 hour for implementation) and when you cannot achieve that with some huge task, you create a special crafted "@todo" comment in the source code with explanation what should be done next. Then, when you merge your branch to master, GitHub bot will parse your source code, extract all new "@todo"s and create new issues based on that. Then, next person working on tasks will pick it up and either implement it or leave another "@todo" comment and so on while the original issue will not get fixed.

![pdd example](/img/todo-pdd.png)
*An example of @todo comment as used in PDD that features issue, estimate and author "role"*

Initially I thought that this would be a perfect process for an indie developer like me that has only very limited chunks of time available. However, the devil is in the ~~implementation~~ detais. Main problem is that PDD (puzzle-driven development) was used in "outsourcing company" / "freelance market" that Yegor runs and used specifically for freelancers to bill the customer atomically. They forced it to work only in _master_ branch and only on "@todo" comments with specific syntax, because only work that makes it to master gets billed.

Soon I realized this wouldn't work for me. I wanted to have my list of puzzles available in any branch I work (if it's a long-living feature branch), not only in master. Also I wanted to leverage existing "normal" style `TODO`/`BUG`/`FIXME` comments that were already present in the code that were not supported by `pdd`. Even besides that, the [ruby gem they provide](https://github.com/yegor256/pdd/) is [very slow](https://github.com/yegor256/pdd/issues/139). This renders this approach quite impractical for my own use.

### TODO-driven development

Although PDD is quite impractical for me, it showed me a right direction. I could still have my subtasks stated in the source code in `TODO` comments, however, I needed some tools to support me.

First of all I tried to check the market if there's anything available. Soon I found project [imdone](https://imdone.io/) that looked very promising. It parses your source code and extracts all `TODO`/`BUG`/`FIXME` comments and organizes them in the kanban board. However, when I installed trial version, I found it very raw. Layout was quite cheap and you couldn't specify which files to include or exclude. Also it didn't support special tags with values like 'estimate' or 'category' which I find useful. However the tool helped me to realize what I wanted and this was priceless.

**So now I knew I wanted a kanban board with all extracted `TODO` comments from the source code**. There was only 1 step left: to create this product. Of course, I didn't want to implement a kanban board from scratch. After quick search I found a beautifully simple project: [kanboard](https://kanboard.org/). After another hour I had it running on my old Raspberry Pi 2.

![kanboard example](/img/kanboard.png)
*Screenshot of kanboard from the official website*

I chose kanboard not only because it was minimalistic, but also because it was easily extensible with plugins or API. So I spent 1 weekend at home and created 2 small projects: a command line tool [tdg](https://github.com/ribtoks/tdg) to fetch all comments from source code and a plugin for kanboard [tdg import](https://github.com/ribtoks/kanboard-tdg-import) to import the output from first tool. 

[tdg](https://github.com/ribtoks/tdg) creates a json out of all comments in the source code:

```js
{
  "root": "/Users/ribtoks/go/src/github.com/ribtoks/tdg",
  "branch": "master",
  "author": "Taras Kushnir",
  "project": "tdg",
  "comments": [
    {
      "type": "TODO",
      "title": "This is title of the issue to create",
      "body": "This is a multiline description of the issue\nthat will be in the \"Body\" property of the comment",
      "file": "README.md",
      "line": 19,
      "issue": 123,
      "category": "SomeCategory",
      "estimate": 0.5
    }
  ]
}
```

After the import with the [plugin](https://github.com/ribtoks/kanboard-tdg-import), it looks like this:

![Xpiks kanboard](/img/kanboard-xpiks.png)
*My local kanboard after import using tdg*

Import plugin for kanboard adds tags for each comment: issue that I'm currently working on and a branch when the task was created on. Also it adds a reference to the file and line of code where the `TODO` was found and time estimate if any left by developer. When I implement the subtask and remove `TODO` comment from code, it get's automatically moved to "Done" column in kanboard so there's **no manual work required**.

Then I created a simple `post-commit` git hook in one of my projects and now after I run `git commit` all `TODO`/`FIXME`/`BUG` comments are automatically synchronized with local kanboard.

```bash
#!/bin/bash
# add real token and host to .bashrc with `export KANBOARD_API_TOKEN='your-token-here'`
API_TOKEN="${KANBOARD_API_TOKEN}"
API_ENDPOINT="http://${KANBOARD_HOST}/kanboard/jsonrpc.php"

JSON_FILE=$(tdg -root "./src" -include "\.(cpp|h)$") || exit $?
PAYLOAD="{\"jsonrpc\": \"2.0\", \"id\": 123456789, \"method\": \"importTodoComments\", \"params\": ${JSON_FILE}}"

CURL_OUT=$(curl -s -u "jsonrpc:${API_TOKEN}" -d "${PAYLOAD}" "${API_ENDPOINT}")
if [[ $CURL_OUT == *'"success":true'* ]]; then
    echo "Sync was successful"
else
    echo "Could not finish sync"
    echo "curl output: ${CURL_OUT}"
    exit 1
fi
```

If you would like to use this setup as `pdd` to have each `TODO` to create a GitHub issue, kanboard project has you covered: there're multiple connectors to platforms as GitHub and GitLab.

## Profit

Now when I start my indie 2-hour workday, I need only a quick glance on `lazygit` and Kanboard to see what I was or will be doing. Usually after I run `git commit`, I go to the Kanboard and drag a task or two to "In Progress" column in order to see next time what should I start with. Eventually before merging back to `master` my kanban board will have all `TODO`s in the "Done" column. Also you can setup automatic action in Kanboard to close all tasks in "Done" column after X days. This way I find it to be nice and clean.
