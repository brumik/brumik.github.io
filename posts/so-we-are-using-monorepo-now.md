---
title: So, we are using monorepo now
date: 2024-12-03
permalink: /blog/2024/12/03/so-we-are-using-monorepo-now
---

For more than a year now we switched to a monorepo. This came after the fact that we had 2 projects in one repository. These two projects had around 12 developers working on it in 4 different teams.

Since the original project was created in a startup, it had little structure to it, since it evolved in a fast-paced environment. Now that we are a big corporate (and everything takes 10x more) we want to split up projects better. So let's "clean up" the previous monorepo in a new monorepo.

## Creating monorepo issues

The decision was to use [Nx](https://nx.dev/). So we set up App no.1 in the monorepo, as well as our design system (DS later). Two projects, 4-ish developers from 2 teams, everything goes well. You have to fight the system a bit to get the DS released as a NPM package, but it works, great.

Now, we figure, we need a bit more than that, so we add another API library and a Logger library. We need to release both of these as NPM package too. No problem we have a pattern. But wait, why did our DS also generated a new release. Well, turns out modifying the root folder will release everything. So now you need to add our first monorepo specific process and rule.

Fast-forward half a year, we have around 6 monorepo specific rules, that we would not have if each of these projects were in their single repository.

## Beta packages

Also, we just started to get to a point where we need a new major release of our DS. We create a beta branch, create a new release from beta, the NPM package looks good. New problem, all the other projects inside the monorepo link the libraries by path, so now you need to find a way to pull some of the packages from the same repo but different branch. 

After more than a year, this repo looks really similar to the original one. Constant 15+ pull requests left to age, a bunch of processes specifically to not screw up the monorepo, quirks that would not exist if they would be separate, and 12 people working on it from 4 different teams that has nothing much in common. 

## Summary

I understand the promise of the monorepo, but in a React world where over-abstraction and over DRY-ing every single line of code is becoming and issue, the code sharing could be better solved. We have libraries that we need to maintain either way. Projects can be developed independently, and most developers are not cross-functional anyway.


I have seen working monorepos, like the PatternFly Design system. They hosted all the 3 major versions of the code in one monorepo. The same team was working on it, and made pack porting fixes a breeze (relatively). I have seen backend and frontend monorepos, that mad sense, since both backend and frontend engineers were part of the same team. Now I can say, that monorepo for *multiple teams* and especially *frontend only* is not something I would ever recommend. 
