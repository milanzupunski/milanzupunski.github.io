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

Here is how it works and how to set it up for your own field.

## What it does

Once a week the script asks three sources for everything new:

- **Europe PMC**, which covers PubMed plus preprints. It searches by the date a paper was first *indexed*, which matters: papers published a while ago but indexed late (the ones I kept missing) still show up.
- **bioRxiv**, for new preprints.
- **Crossref**, checked directly for a list of journals. This catches papers when they get a DOI, often before they reach PubMed.

It then keeps only plant papers, scores each one against your keywords, merges preprint and journal versions of the same paper, skips anything it already showed you in an earlier week, and writes an HTML page sorted into topic sections.

On my first 30-day test run, the three sources returned about 18,000 unique records (11,602 from Europe PMC alone). After filtering and scoring, 791 were left, which works out to roughly 185 a week. The top of my calcium and ROS section was exactly the kind of thing I had been missing.

![The calcium and ROS section of my first digest](/docs/assets/plant-paper-digest-screenshot.png)

On the page you can tick papers, filter by any word, hide preprints or reviews, copy the ticked papers as references, or download them as a RIS file and drag that into Zotero.

## What you need

- Python 3 with the `requests` package. Anaconda includes it. Otherwise run `pip install requests`.
- The files: **[download plant-paper-digest.zip](/docs/assets/plant-paper-digest.zip)** (unzip it anywhere).
- Nothing else: no accounts, API keys or costs.

## Setup

1. **Download and unzip the folder** and put it somewhere permanent, for example `Documents/paper-digest`.
2. **Edit `profiles.txt`** with your own keywords (see below).
3. **Do a first run** that looks back a month. Open a terminal in that folder and run:

   ```
   python digest.py --days 30
   ```

   On Windows you can use `run_digest.bat --days 30` instead, which also opens the result in your browser. The digest appears in `digests/latest.html`. A month of data can take up to half an hour, mostly because bioRxiv is fetched one day at a time. A weekly run is much quicker.

4. **Schedule it weekly.**
   - On **Windows**, double-click `setup_weekly_task.bat`. It creates a task that runs every Monday at 08:30, or at your next login if the computer was off.
   - On **Mac or Linux**, add a line to your crontab (`crontab -e`):

     ```
     30 8 * * 1  cd /path/to/paper-digest && python3 digest.py >> run_log.txt 2>&1
     ```

     On a Mac, cron is not allowed into your Documents folder by default. Either keep the folder somewhere else (for example `~/paper-digest`) or give `cron` Full Disk Access in System Settings, under Privacy & Security.

That is all the maintenance it needs. If the computer is off for a few weeks, the next run catches up on up to 60 days.

## Your keywords

Everything lives in `profiles.txt`, a plain text file. It has four parts.

**1. The plant gate.** A paper must mention at least one of these to be considered at all. This is what keeps imaging and microfluidics papers about human cells out of the list.

```
[gate]
plant
plants
Arabidopsis
rice
seedling*
chloroplast*
# add your organisms here
```

**2. Topic sections.** Each section becomes a heading in the digest. A paper goes into the section where it scores highest. Put your own topics here:

```
[section: YOUR TOPIC 1]
your term | 2
another term* | 1
ACRONYM | 1.5

[section: YOUR TOPIC 2]
...
```

Some rules for writing terms:

| You write | It matches |
|---|---|
| `root hair` | "root hair" and "root-hair" |
| `stoma*` | stoma, stomata, stomatal |
| `ROS` | only uppercase ROS (all-caps terms are case-sensitive) |
| <code>calcium &#124; 2</code> | weight 2 (the default is 1) |
| `re:PIN\d+` | a regular expression, here PIN1, PIN2 and so on |

A term found in the title counts double. Give broad words like "growth" or "development" a low weight (0.5) and the specific ones you really care about a high weight (2 to 3). In my file, terms like GCaMP, RootChip and extracellular ATP carry the most weight.

**3. Boosts and penalties.** Boosts add to the score without defining a section. Penalties push off-topic papers down:

```
[boost]
Arabidopsis | 0.5

[penalty]
patients | 5
plant extract* | 4
cultivar* | 2
```

**4. Journals.** The list that Crossref checks directly, as `ISSN | name | plant/any`. Mark a journal `plant` if every paper in it counts as a plant paper. Mark it `any` if its papers must pass the gate.

## Tweaks

These are the settings I found myself adjusting:

- **Too many papers?** Raise `min_score` in the `[settings]` block. In my case 6 gave about 185 papers a week, 8 about 125, and 10 about 75.
- **Whole fields you don't want.** Add a `[journal penalty]` block. Applied plant pathology and agronomy journals flooded my first run, so I penalize them:

  ```
  [journal penalty]
  Plant Disease | 4
  Phytopathology | 4
  =Plant Pathology | 3
  ```

  The `=` means exact name, so `=Plant Pathology` does not also penalize *Molecular Plant Pathology*.
- **A paper you expected is missing.** Run `python digest.py --ignore-seen --days 30`. Look at the "matched:" line under similar papers to see which terms fire, then add the term that is missing.
- **Reviews crowding the top.** Reviews mention many keywords, so they score high. Use the "hide reviews" checkbox on the page.
- **Old papers showing up.** Europe PMC sometimes indexes old papers in bulk. `max_age_days` (default 365) drops anything published longer ago than that.
- **Only new preprints, or revisions too.** Change `biorxiv_new_only` in the settings block.
- **Windows opens the page in Acrobat or Word.** That means .html files are set to open with that program. Right-click the file, choose *Open with*, then *Choose another app*, pick your browser and click *Always*.
- **A journal always returns zero.** The run report at the bottom of each digest lists journals with no papers. If one stays at zero for a few weeks, its ISSN is probably wrong.

## Limits

This is a keyword filter. It will not understand that a paper about GLR channels is relevant to you if the abstract never uses any of your terms, so it is worth spending ten minutes on synonyms and gene names. bioRxiv's server also returns errors now and then. The script retries, and Europe PMC indexes bioRxiv preprints a few days later anyway, so a missed day usually turns up the following week.

If you set it up for your own field and find a useful tweak, I would be glad to hear about it.

## Credit

I should be clear about who did what. The idea and the testing were mine. The code and the first draft of this post were written by [Claude](https://claude.ai), the AI assistant made by [Anthropic](https://www.anthropic.com), over a single conversation in which I described what I needed, ran it on my own computer and reported back what looked wrong.
