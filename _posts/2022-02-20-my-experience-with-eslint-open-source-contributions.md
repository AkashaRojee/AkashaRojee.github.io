---
layout: post
title: "My Experience with ESLint Open-Source Contributions"
date: 2022-02-20
---

This post shares one of my open-source contribution experiences, with <a href="https://eslint.org" target="_blank">ESLint</a>.

Having regularly worked with ESLint, and as a strong advocate of understanding the tools we work with, I wanted to better understand the configurations. Plus, JavaScript being a language I specialise in, a tool like ESLint is very useful, and I wanted to be able to customise my tooling workflow.

While setting up ESLint from scratch, I nerded through the documentation, and the user guide really helped to make more sense of the basics. This additional understanding could enable me to contribute to ESLint, so I checked some of the issues on the repository, and found some beginner-friendly ones.

I started off by maintaining the documentation blog, then by identifying and fixing website bugs, and today I am creating actual documentation.

## The current issue I'm working on

There was an issue (<a href="https://github.com/eslint/website/issues/899" target="_blank">Add warning for suggested changes to rule docs #899</a>) that I have been eyeing since January, but someone had already asked to work on it since December. However, there was no update for nearly a month. So, I decided to give it a go, and spent a few hours going through the documentation to figure out the solution.

This issue is about adding documentation for suggestions that are manual fixes to problems detected by rules.

## How I Went about Solving it

### Rule Problems

Reading through the documentation about <a href="https://eslint.org/docs/developer-guide/working-with-rules#providing-suggestions" target="_blank">suggestions</a>, I learned about the method `context.report()` that is found in the rule source.

The following code sample is the `context.report()` method for the rule `no-unsafe-negation`, which I started the work with:

<script src="https://emgithub.com/embed.js?target=https%3A%2F%2Fgithub.com%2Feslint%2Feslint%2Fblob%2F345e70d9d6490fb12b18953f56f3cea28fd61d83%2Flib%2Frules%2Fno-unsafe-negation.js%23L99-L123&style=a11y-dark&showLineNumbers=on&showFileMeta=on"></script>

The `context` object contains functionalities that help the rule to do its job. Among its methods, the one in which we're interested here is `report()`.

When your code is incorrect according to the rule, `context.report()` displays a warning/error message.

Usually, ESLint can itself fix the problem.

However, there are some cases where ESLint doesn't do so:

- If the fix can change the code behaviour.
- If there is more than 1 valid way to fix the code.

### Rule Suggestions

In these cases, ESLint provides suggestions for you to manually fix the problem. It does so by having a `suggest` key on `context.report()` (see _line 104_ in the above code sample). This allows other tools (e.g. code editors) to expose helpers so that you can choose which suggestion to apply.

The `suggest` key is an array of suggestion objects. Each suggestion object has a description of the suggestion, and a function that converts your code according to the suggestion.

The description for the suggestion can either be the hard-coded description (as the `desc` key), or an identifier for the description (as the `messageId` key - see _lines 106 & 117_ in the above code sample). The latter is stored in the `messages` key in the rule's `meta` key.

The following code sample is the `messages` key for the rule `no-unsafe-negation`:

<script src="https://emgithub.com/embed.js?target=https%3A%2F%2Fgithub.com%2Feslint%2Feslint%2Fblob%2F345e70d9d6490fb12b18953f56f3cea28fd61d83%2Flib%2Frules%2Fno-unsafe-negation.js%23L77-L81&style=a11y-dark&showLineNumbers=on&showFileMeta=on"></script>

And that's how I understood where to find each rule's suggestions.

### Documenting the Data

To create the documentation for this rule's suggestions, I needed to get more examples of incorrect code and of correct code fixed with those suggestions. Other than from the <a href="https://eslint.org/docs/rules/no-unsafe-negation" target="_blank">rule's documentation page</a>, I found extensive examples from the rule's test source.

The following code sample are examples of invalid code and respective suggestions for the rule `no-unsafe-negation`:

<script src="https://emgithub.com/embed.js?target=https%3A%2F%2Fgithub.com%2Feslint%2Feslint%2Fblob%2F345e70d9d6490fb12b18953f56f3cea28fd61d83%2Ftests%2Flib%2Frules%2Fno-unsafe-negation.js%23L62-L255&style=a11y-dark&showLineNumbers=on&showFileMeta=on"></script>

And there I was, pushing <a href="https://github.com/AkashaRojee/eslint-website/commit/75c150ca09a1b5b934ae50482f485ff5266277ec#diff-6637a8e0a8b96de68eeccb593b63a8015783404d59a9650881405c091d3f0da7" target="_blank">my first draft for a documentation page for one of my favourite tools</a> :smiley:

## What's Next

Getting to make more substantial contributions to ESLint and, in the process, understand more about how the tool works, is a step further.

It was also amazing to see how ESLint automates the generation of their documentation via a <a href="https://github.com/eslint/eslint/blob/main/Makefile.js" target="_blank">Makefile</a>!

After finalising the first draft, deciding on the folder/file structure for that part of the documentation, and clarifying how to proceed with pull requests between the documentation repository and the main repository, the documentation for the other rules' suggestions is next.

Some more ESLint-related blog posts are in the works, as well as a related open-source project.

Eventually, the goal is to be able to contribute to the tool's code itself.

Last but not least, ESLint is one of the most popular JavaScript tools, and the leading open-source JavaScript linting utility. Contributing to something that is used by millions of people is awesome :grin:

I hope this inspires and encourages you to contribute to your favourite open-source tools' repository, whether it's by participating in issues/PRs, or by contributing to blogs, forums, documentation or even code!

What are some of your favourite open-source tools?

Any open-source projects that you are working on?

Share your experience!

## References

* <a href="https://eslint.org/docs/developer-guide/working-with-rules" target="_blank">ESLint - Working with Rules</a>
