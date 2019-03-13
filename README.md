# searx2

Ideas for a asciimoo/searx like metasearch-engine

I did a leittle bit of work on https://github.com/asciimoo/searx, a great project with the right ideas:
- Open Source (with a copyleft license)
- Privacy by design first
  - proxying all ressources that are automatically loaded from third-party sides
  - Optional (js cleaned) proxying of results via the morty proxy


I still see room for improvements, so this is just my fantasy, of how a project like this could be structured and run:

## Which problems should be solved?
- Provide a single endpoint to access search resulsts (from multiple places)
- Privacy first: No trackers, everything proxied through server, no ads
- Clean & extendable code.

## which problems shouldn't it solve?
- SSL and load balancing. Designed to sit behind a reverse proxy
- Keeping state for clients. only cookies / get / post params are honored.
- Log aggregation. Shall provide methods to access logs via a standard format (socket / file), but does not rotate or analyze them.
  

## The Software

### Features

#### User facing:

- [Core] Classic Web Search:
  - [Core] Articles: Web / News
    - Reader-mode link. If the website is article like, it will be attempted to reduce it to it's pure content
  - Media: Images / Videos / Music / Files
    - Reduced, embedded Media player
    - Download-Option where possible (youtube-dl)
  - Map
    - OpenStreetMap
  - URL-safety indication (WOT, blocklists, ...)
  - deobfuscation, deobfuscation and resolving of urls wherever possible
  - WebArchive, etc. "load from cache" option
  - "load via sanitizing proxy"
  - Image preview of webpage
  - Additional information to results wherever applicable (eg. youtube vid: channel & sub count, likes/dislikes) as icons below / beside result (i18n!)
- [Core] Bangs!
  - Redirections to other search engines (eg. "!wiki" -> Wikipedia)
- Questions?
  - explicit requests for instant answers (eg. "?calc 5+3", "?random 1-10", "?en-it Pizza")
  - displaying results from other search engines without redirection to them (e.g. "?wiki Searchengine")
- Accessibility
  - [Core]i18n translated
  - accessible html UI
  - Core functionality does not rely on js
  - Simple keyboard shortcuts:
    - [Ctrl] + [F] (x1) -> Search bar. [Ctrl] + [F] (x2) -> native browser search
    - [Tab]: Searchbar -> Iterates through results
    - [Up] / [Down] (+ [W] / [S] when searchbar is not in focus) -> Iterate through results
    - [Enter] / [Space] -> Opens result. [Shift] / [Ctrl] + [Enter] / [Space] -> Opens result in new tab

#### Admin's pleasure:

- Deployment methods:
  - AUR package
  - Docker file
  - Heroku / etc. deploy button
  - Update script to migrate configurations.
- Rotating outgoing proxies to avoid engine IP bans
- Apache / nginx examples + config examples
- Configuration via cli args -> ENV vars -> config file (checked and honored in that order)


### Design:

**Principles:**
- Low ressource usage - should be able to run smoothly on a raspberry pi.
- Sane defaults. You should not need to configure much to enjoy it.
- Solid core - Should be able to handle all non-critical errors while running gracefully
- Crash fast and loud - When starting with invalid settings or insufficient ressources, it should crash and report those issues to the admin right away to avoid running into those problems later on.


#### Flow of data:

- Input: Web- / JSON- / RSS-Query
- Parsing:
  - Search query:
    - Querying: Gather information from local and remote sources.
    - Aggregation: Rank and merge results
  - Bang: Redirect to other search
  - Question:
    - Fetch results
    - Display instant answer box
- Display: Output result(s) in requested format


#### Structure:

The search engine should probably consists of
- a library that provides the search functionality
- a web-server programm that accesses the library and provides a frontend and the various APIs
- a web-frontend.

Having those components seperate forces a clean separation of unrelated aspects and incremental development. 
I am still unsure, how I want the adapterrs/backends for other searches. Either as independant sub-modules, developed in micro-repos on their own, or as a part of the core library.


- Modular:
  - Core functionality is started as one program.
  - Sanitizing proxy as seperate program.
  - [headless] browser proxy as seperate program.
- Each search engine / instant answer lives in it's own, isolated file/module.
- The core provides the most commonly used APIs and infos to those modules:
  - HTTP requests (GET/POST with PLAIN/FAKE_BROWSER/REAL_BROWSER)
  - JSON / XML / HTTP parsing libraries
  - Information about the query (query, headers, origin_ip, ...)
- Incoming requests are queued up for parsing, then the actions needed to fulfill the request and after a timeout the results and failures are delivered back to the client.



#### Language considerations:
- Go - simple binaries and light threading. Made with web apps in mind.
- Elixir - known for stability in long running, parallel environments.
- Rust - Might be a bit over the top, but lord, Rust IS sexy.
- Python - used by searx, would simplify writing adapters/backends for other search engines
- NodeJS - made for the web, but no type safety (unless typescript) and also bloaty.




## The Project's Organisation
This covers the development, governance and community interaction

- Developed on Github for maximal exposure.
- Slave-Git server on private infrastructure, in case anything goes wrong.
- User and Instance-Admin docs on github wiki pages.
- Short and simple license.
- Funding: Maybe librepay or another open platform (bountys?)

### Contributers and their roles

- **Admins:** 2 persons with admin privileges. They prepare, bundle up and publish releases. 2fa required.
- **Core Contributors:** Up to 5 persons. May accept PRs to the master branch. 2fa required.
- **Trusted Contributors:** Can push to feature-* / fix-* branches.
- **Moderator:** Maintainance of the issue section. Help users and keep track of issues, planning and labeling. Remind everyone of community guidelines.

All contributors are encouraged to enable 2fa on their accounts.


### Development Workflow

1) Features and fixes are developed in seperate `feature-x` and `fix-x` branches. Those branches are always to be created from master.
2) All pushes to branches are automatically built and tested by CI.
3) Core contributors review the changes. If a PR is accepted, it is merged into `master`.
4) Once in a while, the core contributors branch/merge `master` into `stable`. Important fixes may be merged there at any time. Each update to `stable` get's built and packaged up as a release. A changelog has to be created. Config-updating scripts shall be included into the release. Push a projectname-bin to AUR.

- Easy issues that don't enjoy high priority shall be labeled "good-first-issue" and get a link to the CONTRIBUTING docs and a mentor asigned, who can be asked in case of problems.

