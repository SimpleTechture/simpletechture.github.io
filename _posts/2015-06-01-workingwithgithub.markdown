---
layout: post
title: Working with Git(Hub)
tags:
- source control
- git
- github
---

This article describes two possible development workflows called Git-Flow and GitHub-Flow. Git-Flow is a more strict development workflow based on the concept of releases, GitHub-Flow is a lightweight development workflow especially suited for continuous deployment. At this moment Git-Flow aligns best with how we develop projects and perform maintenance for clients. Therefore,  we standardized on using Git-Flow for projects.

We are moving from TFS and SVN as our CVS to Git(Hub) for source control management. As developers of a project we noticed several differences between Git and our previous version control systems we were used to, such as SVN or TFS. The most noticeable difference is that Git is a distributed source control system (DVCS). What does that mean exactly? This means that in the Git model there is no central hub that contains all the source code unlike Subversion of TFS. Instead, each copy of a Git repository contains a complete history of all the commits. So if you pull a repository of a project, you download the entire repository including its history to your local system. 

![Git model without central hub](../../../img/Git-Model-Simple.png)

However, that there is no central hub in the Git model does not mean that we cannot create or use a central hub. Especially, when working in a development team, having a central hub provides benefits to having direct Git communication between team members. This is what GitHub is, a central Git repository with a web front end. GitHub also adds some additional functionality on top of Git.

![GitHub as central Git hub](../../../img/Git-Model-Shared-Repository.png)

Besides being distributed, Git also uses different terminology. Below the most important ones are described.

|**master**|the repository's main branch. Depending on the work flow it is the one people work on or the one where the integration happens|
|**clone**|copies an existing git repository, normally from some remote location to your local environment|
|**commit**|submitting files to the repository (the local one); in other VCS it is often referred to as "check-in"
|**fetch or pull**|is like "update" or "get latest" in other VCS. The difference between fetch and pull is that pull combines both, fetching the latest code from a remote repo as well as performs the merging
|**push**|used to submit the code to a remote repository
|**remote**|these are "remote" locations of your repository, normally on some central server.
|**SHA**|every commit or node in the Git tree is identified by a unique SHA key. You can use them in various commands in order to manipulate a specific node.
|**head**|is a reference to the node to which our working space of the repository currently points.
|**branch**|is just like in other VCS with the difference that a branch in Git is actually nothing more special than a particular label on a given node. It is not a physical copy of the files as in other popular VCS.

It is important to define a standard way of working when using with Git in a development team. This Git workflow describes when to create branches, when to merge and how to handle releases.

##Git Development Workflow

There are two well-known ways of working with Git called **Git-Flow** and **GitHub-Flow**. I will describe both briefly together with the pros and cons of each. At last I will describe which one we currently use for projects and maintenance at my company.

###Git flow

The first development workflow is called Git-Flow. Git-Flow is a development model described by [Vincent Driessen](http://nvie.com/posts/a-successful-git-branching-model/). It describes a strict [branching model](http://summitsystemsinc.net/web1/locked/Summit/SCMBranchingModels1.pdf) which is designed around releases. The image below vizualizes the full development model.

![Git-Flow workflow](../../../img/gitflow-model.png)

I always advocate a lean process, using only the things that are really necessary and removing all complexity. So maybe you could imagine that this model looked a bit too complex to me when I first saw it. Later on when reading the process and the rationale behind it made much more sense. 
Basically, before you implement anything, you start with creating a branch of one of the two branches that are always available and have an infinite lifetime (develop and master). 
The *master branch* is the main branch in which the current state of the source code is always production-ready. The *develop branch* is the main branch in which the current state of the source code contains the latest delivered changes for the next release. 
Besides main and developer, there are the following 3 other branches type that are created for a specific purpose.  

* Feature branches
* Release branches
* Hotfix branches

####Feature branches 
A developer creates a new feature branch from the develop branch just before starting implementing a new feature. The feature branch should be named in such a way that it identifies the feature and includes a reference to the jira ticket which describes the feature. The branche is merged back into the develop branch when the implementation is finished. After the merge the feature branch gets deleted. 

####Release branches
A release branch is created from the develop branch when development for the next release is done. On a release branch some last-minute testing and fixes are implemented to make the branch production ready. The name of the release branch should include the version number of 
the upcoming release, for example release-v1.2. When all testing is done the release branch is merged into master, the master branch is tagged with the version number and the actual deployment is done from the master branch. After the release is also merged back to the develop branch it gets deleted.

####Hotfix branches
A Hotfix branche is created to quickly fix a problem that is currently in production. When a critical bug in a production version must be resolved immediately, a hotfix branch may be branched off from the corresponding tag on the master branch that marks the production version. The name of the hotfix branch should include the new version number of the release, for example hotfix-1.2.1. When the test of the hotfix is finished it gets merged back into master and deployment. After the hotfix is also merged back into the develop branch is can be deleted. 

For more information about Gitflow see the [original](http://nvie.com/posts/a-successful-git-branching-model/) article. 

###GitHub-Flow
GitHub-Flow is a lightweight workflow that supports teams and projects were deploymenta are done regularly. GitHub Flow is designed by the team at GitHub and is also (just like GitFlow) based on branches. 

In GitHub-Flow there are only two types of branches, first there is the master branch which should be always in a deployable state and secondly there are feature branches that developers create of master for implementing a feature or a fix. The branch should get a descriptive name that describes the feature or fix.

![GitHub-Flow](../../../img/github-flow.png)

When you need feedback or help, or you think the branch is ready for merging, you need to open a pull request. A pull request is a request to review the changes the developer made so that the changes can be merged back into the master branch. One or more other developers can review the pull request and give feedback. When all agree, the changes can be merged in master and automatically deployed.

A Pull requests is a function that is built into GitHub. You basically ask the main branch to pull (merge) your branch into the main branch. Many open source projects on Github use pull requests to manage changes from contributors as they are useful in providing a way to notify project maintainers about changes one has made and in initiating code review and general discussion about a set of changes before being merged into the main branch.

The GitHub flow is a much simpler proces than Git flow. But means a much more mature deployment process in which we automatically and continuously deploy every checkin on the master branch. 

####Git-Flow or GitHub-Flow?
So what did we decide to use and standardize for development? For now it seemed that Git-Flow best aligns with how we currently develop projects and maintain solutions for our clients. Although we would like to strive to continuously deliver features to our clients this is not a reality just yet. Currently our projects and maintenance is already centered around releases and therefore aligns best with GitFlow. So for now, Gitflow will be our development workflow for projects and maintenance. But this does not mean that this can't change in the future.


