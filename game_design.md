# Clickjacking - Alice's Cupcakes (game design)

Based on the SEED Labs *Clickjacking* lab (CC BY-NC-SA 4.0, see `ATTRIBUTION.txt`).

**Target audience:** system-security students (about 30 per run). **Duration:** about 90 minutes.
**Scoring:** 200 platform points = 20/20 (x10, because the platform only accepts whole numbers). Hint penalties 5-20 points = -0.5 to -2 / 20.

## Learning outcomes
* Explain clickjacking (UI redress) and build an attack with an iframe and a transparent overlay button.
* Implement a JavaScript frame-buster and understand why front-end defenses are bypassable (iframe `sandbox`).
* Configure `X-Frame-Options` and `Content-Security-Policy: frame-ancestors` on Apache.

## Levels
| Level | Max score | Flag variable | Hints (penalty) | Lab questions covered |
|---|---|---|---|---|
| Task 0 - Start environment | 10 | `flag_t0_setup` | 5, 5 | - |
| Task 1 - iframe | 20 | `flag_t1_iframe` | 5, 10, 15 | Q1 |
| Task 2 - invisible button | 30 | `flag_t2_clickjack` | 5, 10, 20 | Q2-Q4 |
| Task 3 - frame busting | 30 | `flag_t3_framebuster` | 5, 10, 20 | Q5-Q6 |
| Task 4 - sandbox | 30 | `flag_t4_sandbox` | 5, 10, 15 | Q7-Q9 |
| Task 5 - HTTP headers | 40 | `flag_t5_headers` | 5, 10, 20 | Q10-Q12 |
| Final quiz (8 x 5) | 40 | - | - | Q1-Q12 concepts |

Questions Q1-Q12 are asked in the lab report (screenshots + explanations); the quiz re-checks the key concepts automatically.

## Anti-cheating design
* Different random flag per level **and** per sandbox (APG variables).
* The flag is released by `cjlab-verify` only when the student's actual files / live headers are correct.
* The student has no general sudo, so the root-only flag file is not readable.
* Up to 10 wrong answers per level.

## Reference solutions
See the `solution` field of each level in `training.json` (shown to the student only at the price of the whole level score).

### Task 1 - `attacker.html` / `attacker.css`
```html
<iframe src="http://www.cjlab.com"></iframe>
```
```css
iframe { position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none; }
```
### Task 2 - `attacker.css` (button rule)
```css
color: transparent; background-color: transparent; margin-left: 45px; margin-top: 345px;
```
### Task 3 - `defender/index.html`
```javascript
function makeThisFrameOnTop() {
    if (window.top !== window.self) { window.top.location = window.self.location; }
}
```
### Task 4 - `attacker.html`
```html
<iframe src="http://www.cjlab.com" sandbox></iframe>
```
### Task 5 - `image_defender/apache_defender.conf` (then `dcdown; dcbuild; dcup`)
```apache
Header set X-Frame-Options "DENY"
Header set Content-Security-Policy "frame-ancestors 'none';"
```
