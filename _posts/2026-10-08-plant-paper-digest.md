---
layout: post
title: "A weekly plant biology paper digest you can run on your own computer"
date: 2026-10-08
---

<style>
.post img { max-width: 100%; height: auto; border: 1px solid #ddd; }
.post pre { background: #f5f5f2; padding: 10px 12px; overflow-x: auto; font-size: 14px; }
.post code { font-size: 0.92em; }
.post table { border-collapse: collapse; margin: 12px 0; }
.post th, .post td { border: 1px solid #ddd; padding: 4px 10px; text-align: left; }
</style>

For a while now I have had the feeling that I am losing track of the plant biology literature. I follow a lot of people on LinkedIn and Bluesky, and I search Google for things like "calcium Arabidopsis 2026" more often than I would like to admit. Still, every few weeks I find a paper that came out two or three months ago and somehow never crossed my screen.

The reason is fairly simple. Social media shows you what someone decided to post, and search engines show you what is popular. Neither gives you a complete list of what was published last week. So I built a small script that goes to the indexing databases directly, filters by keywords, and gives me one page per week that I can curate in about twenty minutes.

Here is how it works, and how to get your own.

## What it does

Once a week the script asks three sources for everything new:

- **Europe PMC**, which covers PubMed plus preprints. It searches by the date a paper was first *indexed*, which matters: papers published a while ago but indexed late (the ones I kept missing) still show up.
- **bioRxiv**, for new preprints.
- **Crossref**, checked directly for a list of journals. This catches papers when they get a DOI, often before they reach PubMed.

It then keeps only plant papers, scores each one against your keywords, merges preprint and journal versions of the same paper, skips anything it already showed you in an earlier week, and writes an HTML page sorted into topic sections.

On my first 30-day test run, the three sources returned about 18,000 unique records (11,602 from Europe PMC alone). After filtering and scoring, 791 were left, which works out to roughly 185 a week. The top of my calcium and ROS section was exactly the kind of thing I had been missing.

![The calcium and ROS section of my first digest](/docs/assets/plant-paper-digest-screenshot.png)

You can see my own digest, updated every Monday, [here](/my-paper-digest/). On the page you can tick papers, filter by any word, hide preprints or reviews, copy the ticked papers as references, or download them as a RIS file and drag that into Zotero.

## Get your own

The code is free on GitHub: **[plant-paper-digest](https://github.com/milanzupunski/plant-paper-digest)**. You copy it to your own GitHub account with one click and put in your keywords. GitHub then runs it for you every Monday and publishes your digest as a web page you can open on any device, phone included. There is nothing to install and it costs nothing.

Setup takes about five minutes. The README walks through it step by step, including how to write and tune your keywords. If you would rather run it on your own computer, that works too, with Python and a single command.

## Limits

This is a keyword filter. It will not understand that a paper about GLR channels is relevant to you if the abstract never uses any of your terms, so it is worth spending ten minutes on synonyms and gene names. bioRxiv's server also returns errors now and then. The script retries, and Europe PMC indexes bioRxiv preprints a few days later anyway, so a missed day usually turns up the following week.

If you set it up for your own field and find a useful tweak, I would be glad to hear about it.

*I should be clear about who did what. The idea and the testing were mine. The code and the first draft of this post were written with the help of [Claude](https://claude.ai) by [Anthropic](https://www.anthropic.com).*
