# Contributing

Thanks for considering a contribution. This list is curated, not crowd-sourced , every entry has been personally verified before merging. That's the whole point: a reader should be able to trust every link.

## What's in scope

- **ROS 1 / ROS 2 / DDS security** , attacks, defences, tooling, CVEs.
- **Robotics security more broadly** , industrial robots, drones, quadrupeds, humanoids, sensor/perception attacks.
- **Resources that make a researcher more effective** , books, courses, papers, conference talks, simulators, hardware platforms, vendor disclosures.

## What's out of scope

- Resources without primary-source evidence (no "I heard about" entries).
- Generic IoT/embedded content with no robotics tie-in , add those to [V33RU/awesome-connected-things-sec](https://github.com/V33RU/awesome-connected-things-sec) instead.
- Vendor marketing or product pages without security relevance.

## How to add an entry

1. Fork the repo.
2. Add your entry to the most appropriate section, alphabetically within that section unless the section is chronological (e.g. Conference Talks).
3. Verify the link works in an incognito window.
4. For CVEs: include the NVD link and confirm the vendor + description on NVD match your entry.
5. For papers: cite arXiv ID / DOI / publisher link, confirm authors and year.
6. For tools / conferences / books: cite the primary source (project README, conference archive, publisher page).
7. Open a PR with a one-line description in the body.

## PR checklist

- [ ] Every URL I added returns 200 OK (or is a known bot-blocked but real page like NVD/IEEE).
- [ ] Every claim is traceable to the linked primary source.
- [ ] No em-dashes (`—`), no fabricated content, no AI-generated summaries that I haven't fact-checked.
- [ ] Entry is in the correct section.
- [ ] Markdown renders cleanly on GitHub preview.

## Reporting wrong entries

If you find anything in the list that's inaccurate or out of date, open an issue with the **"Wrong / outdated entry"** template. We treat these as P0.

## Code of conduct

Be kind. This is a learning resource for the robotics security community , a community small enough that everyone eventually knows everyone.

## Maintainers

- [@V33RU](https://github.com/V33RU) (IOTSRG)
