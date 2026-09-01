# Manulife Asia · AI Champions Network Kickoff

Interactive, passphrase-protected playbook for the two-day Asia AI Champions
Network Kickoff — Hong Kong, 8–9 September 2026.

| | |
|---|---|
| **Day 1 — Champion Summit** | 9 sections: welcome, opening pulse, RACE + EQ, personalization, chaining across apps, Cowork skills lab, adoption dashboard, 30-day plan, close |
| **Day 2 — Train the Champion** | 8 sections: welcome, facilitation framework, hard questions & resistance, teach-back, idea to business value, should this become an agent, build your agent, commit & close |

## How it is protected

The entire application (markup + behaviour) is encrypted at rest with
**AES-GCM-256**, keyed by **PBKDF2-SHA256 at 250,000 iterations** over the
session passphrase. Nothing readable ships in the HTML source — view-source
shows only the ciphertext blob. The passphrase is never stored in the file.

Participants enter the passphrase once; it is held in `sessionStorage` for
that browser session only.

## Participant answers

61 fields across the two days save to `localStorage` as the participant
types — nothing is transmitted anywhere. At the end of Day 2 they can export
everything they wrote as a single Markdown file.

## Publish on GitHub Pages

1. Push the contents of this folder to the repo root.
2. **Settings → Pages → Source: Deploy from a branch**, branch `main`, folder `/ (root)`.
3. Wait ~60 seconds. The playbook is at `https://<owner>.github.io/<repo>/`.

`.nojekyll` is included so files are served as-is.

## Rebuilding

The page is generated, not hand-edited. Content lives in `content.py`;
`gen.py` assembles, encrypts and writes `index.html`:

`python gen.py "<passphrase>"`
