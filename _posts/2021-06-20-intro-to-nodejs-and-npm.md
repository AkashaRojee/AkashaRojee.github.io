---
layout: post
title: "Intro to Node.js and npm"
date: 2021-06-20
---

This article is the first of a series about npm for beginners, and the introductory guide that I would have loved when I was just starting out.

I recently started using npm packages for linting and testing. It took me a while to figure my way around the commands. Yet, I didn't fully understand npm as a tool and how it relates to Node.js.

Following several hours of nerding through documentation and some articles, I think I have the basics mostly laid out.

Let's dive in!

## What is Node.js?

Node.js is an open-source cross-platform run-time environment that allows developers to create server-side tools and applications, as well as command-line tools, using JavaScript, and for use outside of a browser (e.g. on a computer or server).

#### **Why Node.js?**

Node.js is particularly great for web server development.

* Great performance, especially in real-time web applications
* Uses JavaScript for both client-side and server-side
* All the advantages that JavaScript comes with (e.g. constant improvements in language design, and lots of popular languages compile/convert to JavaScript)
* NPM! (More on that below)
* Portable over various operating systems
* Many web hosting providers even have specific infrastructure and documentation for hosting Node.js websites
* Active ecosystem and developer community

#### **Any limitations?**

Node.js on itself, however, cannot do everything required for web development tasks, such as:
* handling different HTTP request methods and at different URLs
* serving static files
* creating HTTPS responses dynamically

For this reason, web frameworks are used alongside Node.js, for e.g. Express, which is the most popular Node.js web framework.

## NPM?

npm is 2 things.

1. A repository for open-source Node.js packages.
2. A command-line utility, Node package manager, for interacting with said repository

#### **What does all of this mean?**

npm started off as a way to allow JavaScript developers to easily share their packaged modules of code.

<a href="https://www.npmjs.com/" target="_blank">Today, the npm repository gives access to over 1.5 million reusable packages</a>

![Image of npmjs.com website](/assets/images/npmjs.com.png)

#### **But wait... What is a package?**

A package is basically a folder that contains all the code to use a feature in your project. This feature can be a module, or a library of scripts.

Some examples of popular packages:

* lodash - utility library to work with arrays, numbers, objects and strings
* react - to create user interfaces
* chalk - to style text in the terminal

#### **What about the command-line utility?**

You can use npm commands to:
- install packages from the npm repository
- publish packages to the npm repository
- manage your project dependencies

## Node.js and npm - How does it all link up?

Projects using Node.js packages have a package.json file. The package.json file is like the manifest of your project.

#### **What does package.json contain?**

It contains properties about your project, in JSON file format: metadata like the project's name, version, description and author, as well as information about the packages your project uses (in other words, the packages your project depends on, a.k.a. your project's dependencies).

Those properties help to identify your project and serve as a starting point of reference for users and contributors to get information about the project.

Sample of package.json file:

```
{
  "name": "portfolio",
  "version": "1.0.0.",
  "description": "Portfolio of projects built",
  "main": "index.js"
  "license": "MIT",
  "devDependencies": {
    "lighthouse": "^8.0.0",
    "mocha": "~6.2.3",
    "stylelint": "~12.0.1",
    "webhint": "~6.0.4"
  },
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

*For more information about each property, <a href="https://docs.npmjs.com/cli/v6/configuring-npm/package-json" target="_blank">view the npm Docs about package.json properties</a>.*

The properties with which we're mostly concerned here are the dependencies.

#### **The big word: dependencies**

Dependencies are packages that your project uses.

Dependency objects come in various forms: `devDependencies`, `dependencies`, `peerDependencies`, `bundledDependencies` and `optionalDependecies`.

The first 2 are the only ones we're interested with here.

1. `devDependencies`: dependencies used during development/testing 
2. `dependencies`: dependencies used during production

Using npm, you can run an install command inside your project: `npm install`

* This first creates a `node_modules` folder inside your project.
* Then, inside this folder, it installs all dependencies, in the version specified for each, as listed in your project's package.json.

Dependencies can then be used in your project through the `require()` function.

Thanks to the nifty tool that npm is, packages that your project uses thus don't need to be bundled with the project itself, making it more lightweight.

There is also no need to install each dependency individually, saving you time. This facilitates installing a Node.js project from a Git repo. You would simply have to enter the following commands, and you're good to go!

```
git clone https://github.com/username/repo-name
cd repo-name
npm install
```

#### **dependencies vs. devDependencies**

The separation of `dependencies` and `devDependencies` are handy for when the project is run in either a production or a development environment. The npm install command has flags that caters for this.

In a production environment, you wouldn't need packages such as page audit tools (e.g. Lighthouse), linters (e.g. Webhint and Stylelint) and testing frameworks (e.g. Mocha). These are used only in a development environment.

To install production-only packages, you would use: `npm install --production`

Examples of packages that would be used in a production environment are your web framework.

On the other hand, in both a production and a development environment, you would need both `dependencies` (e.g. web framework), as well as `devDependencies` (e.g. code and testing utilities).

#### **What else about npm?**

One thing that I particularly love with npm is that it helps to automate various development tasks. More on that later!

As compared to other package managers, npm is also known to have the best dependency resolution (a.k.a. when having to manage dependencies required in your project but which are missing/incorrect/incompatible).

Now that we have a clearer picture of Node.js and npm, let's proceed with their installation.

## Installation

Node.js includes the npm command-line interface.

To install them, it is recommended to use Node version manager. This allows you to install and switch between multiple versions of Node.js and npm.

Why would you want to have multiple versions, you ask?

Just so that you can test your app on multiple versions to ensure that your app works for users on different versions.

I used nvm on Ubuntu.

*For other version managers or other operating systems, <a href="https://docs.npmjs.com/downloading-and-installing-node-js-and-npm" target="_blank">view the npm Docs for downloading and installing Node.js and npm</a>.*

#### **To install nvm**

1. Enter either of the following commands:

    ```
    cURL -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
    ```

    ```
    wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
    ```

    This downloads a bash script and runs it. The script installs the nvm repository to `~/.nvm`.

2. Restart your terminal.

3. Check that nvm is successfully installed by entering the following command:

    ```
    nvm --version
    ```

    which should output the nvm version.

*For more information about nvm, [view the nvm repo](https://github.com/nvm-sh/nvm).*

#### **To install Node.js and npm**

There is a new major release of Node.js every 6 months, with new features added and old ones removed.

To ensure that your Node.js project works stably over a longer period, without requiring constant updates with each new Node.js release, it is recommended to use Long Term Support (LTS) releases.

1. To install the LTS version:

    ```
    nvm install --lts
    ```

2. To check that Node.js and npm are successfully installed, enter the following commands:

    ```
    node --version
    ```

    which should output the Node.js version.

    ```
    npm --version
    ```

    which should output the npm version.

**The real benefit of nvm becomes obvious when you have to use different versions of Node.js, as explained earlier.**

* To install the latest version:

    ```
    nvm install node
    ```

* To list all versions available for installation:

    ```
    nvm ls-remote
    ```

* To install a specific version:

    ```
    nvm install VERSION-NUMBER
    ```

**Switching to a different version of Node.js installed on your machine:**

* To list all versions installed on your machine and see which one is currently used:

    ```
    nvm ls
    ```

* To use a specific version:

    ```
    nvm use VERSION-NUMBER
    ```

**Setting a default version of Node.js:**

* To set a specific version as default:

    ```
    nvm alias default VERSION-NUMBER
    ```

## That's it for now!

For those starting out with Node.js and npm, I hope that this article cleared some of the mystery shrouding them.

I am myself learning Node.js and npm, and I would love to receive your feedback.

For those who already use Node.js and npm, what are some of your favourite packages?

And to more experienced developers, what are some advice/tips/best practices that you would recommend to those starting out?

## Up next: more npm goodness!

## References

* <a href="https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction" target="_blank">Express/Node introduction - MDN Web Docs</a>
* <a href="https://nodesource.com/blog/an-absolute-beginners-guide-to-using-npm" target="_blank">An absolute beginner's guide to using npm - The NodeSource Blog</a>
* <a href="https://nodejs.org/en/knowledge/getting-started/npm/what-is-npm" target="_blank">What is npm? - Node.js</a>
* <a href="https://nodejs.dev/learn/an-introduction-to-the-npm-package-manager" target="_blank">An introduction to the npm package manager - Node.js</a>
* <a href="https://docs.npmjs.com/cli/v6/configuring-npm/package-json" target="_blank">Specifics of npm's package.json handling - npm Docs</a>
* <a href="https://docs.npmjs.com/downloading-and-installing-node-js-and-npm" target="_blank">Downloading and installing Node.js and npm - npm Docs</a>
* <a href="https://github.com/nvm-sh/nvm" target="_blank">nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions - nvm</a>
