# Cybersecurity game "Clickjacking - Alice's Cupcakes"

Linear game with **per-student flags** for the [CyberRangeCZ Platform](https://docs.platform.cyberrange.cz/),
built from the structure of [`library-junior-hacker`](https://github.com/cyberrangecz/library-junior-hacker)
and the content of the SEED Labs *Clickjacking (Alice's Cupcakes)* lab (`Web_Clickjacking_Cupcakes.pdf`).
Scored **out of 20** (the platform displays it x10 = 200 points), designed for a class of ~30 students.

## Game levels summary
| # | Level | Type | Points (x10) | = /20 | Flag obtained by |
|---|---|---|---|---|---|
| 0 | Welcome and grading rules | info | - | - | - |
| 1 | Get Access | access | - | - | passkey `start` |
| 2 | Task 0 - Start the lab environment (`docker-compose`) | training | 10 | 1 | `sudo cjlab-verify 0` |
| 3 | Task 1 - Copy that site (`<iframe>`) | training | 20 | 2 | `sudo cjlab-verify 1` |
| 4 | Task 2 - Let's get clickjacking (invisible button) | training | 30 | 3 | `sudo cjlab-verify 2` |
| 5 | Task 3 - Bust that frame (frame-busting JavaScript) | training | 30 | 3 | `sudo cjlab-verify 3` |
| 6 | Task 4 - Bust the buster (iframe `sandbox`) | training | 30 | 3 | `sudo cjlab-verify 4` |
| 7 | Task 5 - The ultimate bust (`X-Frame-Options`, CSP `frame-ancestors`) | training | 40 | 4 | `sudo cjlab-verify 5` |
| 8 | Final quiz (8 questions x 5 pts) | assessment (TEST) | 40 | 4 | - |
| 9 | Debriefing and lab report | info | - | - | - |
| | **Total** | | **200** | **20** | |

**Hint penalties:** every hint costs 5, 10, 15 or 20 platform points = **-0.5, -1, -1.5 or -2 / 20**
(why x10: CyberRangeCZ levels/hints only accept whole numbers, so 0.5 pt is represented as 5 points).
Showing the full solution gives 0 for the level (`solution_penalized: true`).
The student may still divide the platform score by 10 to get the mark out of 20.

## Topology
| Host | Image | Flavor | Role |
|---|---|---|---|
| `seed` | `seed-ubuntu-20.04` (**adapt**) | `m1.medium` | student workstation (replaces Kali); runs the two lab containers (defender 10.9.0.5, attacker 10.9.0.105) with Docker |
| `router` | `debian-12-x86_64` | `m1.small` (**adapt**) | gives the sandbox Internet access (Docker Hub, CDNs) |

Network `seed-switch` 10.1.27.0/24 (`seed` = 10.1.27.10, router = 10.1.27.1) + WAN `internet-connection`.

### What you MUST adapt in `topology.yml`
1. `image:` of the `seed` host - name of your SEED VM image in OpenStack/Glance.
2. `mgmt_user:` - the account CyberRangeCZ/Ansible uses to connect to that image (key-based SSH + passwordless sudo).
3. router `flavor:` if `m1.small` does not exist in your cloud.

### Requirements for the SEED image
* Docker + `docker-compose` (v1 CLI) installed - the SEED Ubuntu 20.04 VM already has them.
* A desktop reachable through the platform's *Open GUI* (xrdp/VNC) with Firefox; user `seed`.
* Internet access through the router (pull `handsonsecurity/seed-server:apache`, Bootstrap/Google-Fonts CDNs used by the pages).

## How each student gets different flags (anti-cheating)
1. `variables.yml` declares 6 variables (`flag_t0_setup` ... `flag_t5_headers`) - the platform generates **new random
   values for every sandbox** (APG / variant answers) and keeps the right answers in its answers storage.
2. `training.json` has `variant_sandboxes: true` and each training level uses
   `variant_answers: true` + `answer_variable_name: <variable>` - the student's answer is compared to *their* sandbox value.
3. During provisioning (role `seed`) the values are written to `/opt/cjlab/secrets.json` (root only) together with the verifier `/usr/local/bin/cjlab-verify`.
4. The student runs `sudo cjlab-verify <n>`. The tool **inspects the real work** (their `attacker.html/.css`, `defender/index.html`, the live HTTP headers
   of `www.cjlab.com`, the running containers) and prints the personal flag **only if everything is correct**; otherwise it lists the `[FAIL]` criteria.
   A student can neither copy a flag from a classmate nor submit it without doing the task.
5. The student is removed from the `sudo` group (only `cjlab-verify` is allowed through `sudoers.d`), so the flag file stays unreadable.

### Verification criteria (`cjlab-verify`)
| Task | Checks |
|---|---|
| 0 | containers `defender-10.9.0.5` and `attacker-10.9.0.105` running; both sites answer HTTP 200 |
| 1 | an `<iframe>` (not commented out) with `src` = `www.cjlab.com`, placed before the button; no border; width and height cover the page |
| 2 | Task 1 criteria + button still goes to `hacked.html`; invisible (`opacity:0` or transparent `color` **and** `background-color`) but clickable (no `display:none`/`visibility:hidden`/...); moved (`margin-*`/`left`/`top`) |
| 3 | `makeThisFrameOnTop()` compares `top`/`self`, assigns `top.location`, and is still called from `window.onload` |
| 4 | iframe has `sandbox` without `allow-top-navigation` / `allow-scripts`; Tasks 2 and 3 still in place |
| 5 | live response of `www.cjlab.com` has `X-Frame-Options: DENY` **and** `Content-Security-Policy` with `frame-ancestors 'none'` |

*Limits:* checks are static (regex-based) except Task 0 and 5, and the **exact position** of the invisible button over *Explore Menu* cannot be verified automatically -
the lab report screenshots (Q1-Q12) remain the proof for that part.

## Import into CyberRangeCZ and run it for 30 students
1. **Push this repository to GitHub** (see below).
2. *Sandbox definitions* -> **Create** -> Git URL of the repo (branch `main`, revision = `main` or a tag). The platform reads `topology.yml`, `variables.yml`, `provisioning/`.
3. *Training definitions* -> **Upload** `training.json` (state `UNRELEASED`, "variant sandboxes" is already enabled) -> release it when ready.
4. *Pools* -> create a pool from the sandbox definition, size **30 (+2-3 spare)** and allocate. Docker image pulls happen during provisioning: allocate in
   batches if you hit Docker Hub rate limits (the base image is pulled once per sandbox; the pull retries 5 times).
5. *Training instances* -> create an instance from the training definition + the pool, hand the access token to the students.
6. The *Results / Scoreboard* shows scores x10; divide by 10 for the mark out of 20.

## Repository content
```
topology.yml                 sandbox topology (seed VM + router)
variables.yml                APG variables = the 6 per-student flags
training.json                training definition (levels, hints, quiz, scoring)
game_design.md               level-by-level design and solutions
provisioning/
  playbook.yml               Ansible entry point
  requirements.yml           external roles (user-access, chrony, sandbox-logging, ...)
  host_vars/seed.yml
  roles/seed/                lab deployment: hosts, Docker, Labsetup, flags, verifier, sudo rules
    files/Labsetup/          SEED Labs Clickjacking lab files (defender + attacker websites)
    files/cjlab-verify       the verifier tool
```

## Push to GitHub
```bash
cd library-clickjacking-cupcakes
git remote add origin https://github.com/<your-account>/library-clickjacking-cupcakes.git
git push -u origin main
```

## Troubleshooting
* **GUI login fails** - the provisioning sets the password of user `seed` to `dees`; check that the `user-access` role ran (sandbox provisioning log).
* **`permission denied` for docker** - log out/in of the desktop session (group `docker` is applied at login).
* **`cjlab-verify: command not found`** - open a new terminal (alias in `~/.bashrc`) or run `sudo /usr/local/bin/cjlab-verify <n>`.
* **Students still have `sudo`** - some SEED images grant `seed` NOPASSWD sudo through `/etc/sudoers.d/`. Remove that rule in the image, otherwise
  a student could read `/opt/cjlab/secrets.json` (flags would still be unique per sandbox, but a student could skip the work).
* **Hint/flag characters** - the `password` APG type produces 16 random characters; students must paste the flag exactly.

## License
* Code (Ansible, verifier, scripts, JSON structure): MIT (see `LICENSE`).
* Lab content and game design (`provisioning/roles/seed/files/Labsetup`, texts): **CC BY-NC-SA 4.0** (SEED Labs, see `ATTRIBUTION.txt`) - non-commercial teaching use.
