---
layout: post
title:  "On Code Reviews and Git Commit Messages"
date:   2018-05-23 22:04:42 +0200
categories: git
---
Can you tell what the commits in the following screenshot do?

![]({{site.url}}/assets/posts/1.jpg)

They introduce changes to code, but in order to understand them and be able to perform a code review, you’d need to dig deep into the changes themselves, build up the context around them, hunt through files, and try to link everything together. In your head.

You’ll waste precious mental resources trying to re-establish the context and understand the scope of the changes. If the changes in question are non-trivial (and chances are they aren’t), you’ll also try to reason about the motivation behind the changes. And that’s expensive. It’ll take you time and energy to go through the commit, jumping from one end to another, trying to piece together what affects what. You’ll most probably end up opening a few older commits in hopes of grasping the effects and reasons for the changes you’re reviewing.

If you can’t understand the point and purpose of the changes, you can’t easily judge on their merit either.

Of course, there are other factors involved which can, to a limit, safeguard against unwanted changes, such as high quality tests and good test coverage of your code base. But let’s face it – that’s not enough by itself.

The point of code review shouldn’t be about pointing out idiomatic constructs in diffs or making sure the contributor adheres to a particular style guide. There are ways to automate those kinds of checks and perform automatic fixes ([phpcs-fixer](https://github.com/FriendsOfPhp/PHP-CS-Fixer), [phpmd](https://github.com/phpmd/phpmd), build tools like [phing](https://github.com/phingofficial/phing), task runners like [robo](https://robo.li/)).

The point of code review should be to weigh on the benefit a proposed change would bring to the project.
When passing such judgement, reviewer should look to the technical aspects of the changes proposed as well as judge on the benefits these changes would bring to the business side of the story.

Will it remove some of technical debt we incured thusfar?
Will it speed up the execution time by reducing the number of I/O calls?
Or will it help us down the line because it introduces low coupling and high cohesion to the part of the system that is expected to grow in the near future?

Such changes aren’t easy to piece together, and the latter ones are especially difficult to understand and judge if there’s no back story.

This can be easily avoided by writting useful Git Commit messages, and keeping your commits _atomic_ (I'll write about _atomic commits_ in a later post).

By providing a useful commit message you help your fellow developers rebuild the context of the changes you commited more easily and quickly.

You’ve already created the changes, built the context in your head, you understand the motivation and reasons that brought about the change, and you know what parts of code base are affected, in which areas the changes will have side-effects.
You are the one who delved in the business problem. You are the one who understands what reasons drove the changes. You are the one who commited the changes.

![]({{site.url}}/assets/posts/2v2.jpg)


Why force your code reviewers to try to understand in five or ten minutes the things that took you days, if not weeks?

Isn’t it wasteful and inconsiderate not to use your knowledge to describe the work you did?

Help you fellow developers, and even yourselves down the line, and describe the whats and whys of the hows in your commits.

There's a lot more to writing good and useful commit messages, but as a start, try following these 7 simple rules:

- Separate subject from body with a blank line
- Limit the subject line to 50 characters
- Capitalize the subject line
- Do not end the subject line with a period
- Use the imperative mood in the subject line
- Wrap the body at 72 characters
- Use the body to explain what and why vs. how

When it comes to writing the commit subject line, just follow this simple rule:

<p markdown="1"> _If applied, this commit will **your subject line here**._ [^1] </p>



If you want to know more, check out these great articles:

- https://chris.beams.io/posts/git-commit/
- http://who-t.blogspot.hr/2009/12/on-commit-messages.html
- http://www.osnews.com/story/19266/WTFs_m
- https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html

[^1]:https://chris.beams.io/posts/git-commit/
