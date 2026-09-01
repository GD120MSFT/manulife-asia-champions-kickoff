# Collecting champion answers — Power Automate setup

The playbook keeps everything a participant writes in their own browser. Pressing
**Send my answers** posts it once to a Power Automate flow, which writes it into a
SharePoint list the programme team can read.

Until the flow exists, the Send button says *"Sharing is not switched on for this
build"* and participants can still use **Download instead** — so the page is
safe to hand out before this is finished.

---

## 1 · Create the SharePoint list

On the Manulife delivery site (`CT-53698`), create a list called
**`Asia Champion Kickoff Feedback`** with these columns:

| Column | Type | Holds |
|---|---|---|
| Title | Single line | the section heading |
| SubmissionId | Single line | groups every row from one person's submission |
| SubmittedAt | Date and time | when they pressed Send |
| ParticipantName | Single line | `(anonymous)` when they leave it blank |
| ParticipantRole | Single line | Regional / Market / Local Champion, or activation lead |
| FunctionName | Single line | e.g. Finance |
| Market | Single line | e.g. Hong Kong |
| Section | Single line | e.g. `Day 2 · From AI idea to measurable business value` |
| Question | Multi-line | the field label they answered |
| Answer | Multi-line | what they wrote |
| AnswerCount | Number | how many fields they completed in total |

---

## 2 · Build the flow (about five minutes)

**make.powerautomate.com** → **Create** → **Instant cloud flow** → **Skip** the
trigger picker → search **"When an HTTP request is received"** → add it.

### Generate the schema
In the trigger click **Use sample payload to generate schema** and paste the whole
contents of `feedback-sample.json` (next to this file). Click **Done**.

Set **Who can trigger the flow?** → **Anyone**.

### Flatten the sections into rows
**+ New step** → **Control** → **Apply to each**
- *Select an output from previous steps*: `sections`
- Inside it, add a **second Apply to each** over `answers` (the current item's answers)
- Inside that, **SharePoint → Create item**:

| Field | Value |
|---|---|
| Site Address | `https://microsoft.sharepoint.com/teams/CT-53698` |
| List Name | `Asia Champion Kickoff Feedback` |
| Title | `section` (from the outer loop) |
| SubmissionId | `submissionId` |
| SubmittedAt | `submitted` |
| ParticipantName | `name` |
| ParticipantRole | `role` |
| FunctionName | `functionName` |
| Market | `market` |
| Section | `section` (outer loop) |
| Question | `question` (inner loop) |
| Answer | `answer` (inner loop) |
| AnswerCount | `answerCount` |

### Optional — a readable copy per person
After the loops, add **Outlook → Send an email (V2)** to yourself and the Manulife
programme lead, with `name`, `functionName`, `market` and `answerCount` in the
subject and the `markdown` field as the body. That gives you one tidy document per
champion without opening the list.

### Save and copy the URL
**Save**, then reopen the trigger — the **HTTP POST URL** is now filled in.

---

## 3 · Second flow — the live room board (optional but recommended)

The playbook has a facilitator view at **`?room=1`** that shows the pulse answers
back to the room as anonymous distributions. It needs a **read** flow.

**make.powerautomate.com** → **Create** → **Instant cloud flow** → **When an HTTP
request is received**. Set **Method: GET** and **Who can trigger: Anyone**.

1. **SharePoint → Get items** on `Asia Champion Kickoff Feedback`, with
   *Filter Query* `EntryType eq 'Pulse'` and *Top Count* `500`.
2. **Data Operations → Select** over `value`, mapping to:

   | Key | Value |
   |---|---|
   | phase | `Phase` |
   | demo | `DemoScore` |
   | peer | `PeerScore` |
   | level | `LevelScore` |
   | need | `Answer` |

3. **Response** action:
   - *Status Code*: `200`
   - *Headers*: `Content-Type: application/json` **and**
     `Access-Control-Allow-Origin: *` — without this second header the browser
     blocks the read.
   - *Body*: `{ "responses": <output of the Select step> }`

Save, copy the GET URL, and rebuild:

```
python gen.py "<passphrase>" "<POST flow URL>" "<GET flow URL>"
```

The pulse rows need four extra columns on the list: **Phase** (single line),
**DemoScore**, **PeerScore** and **LevelScore** (all number). In the POST flow, add a
**Condition** on `kind` — when it equals `pulse`, write one row with
`EntryType = Pulse`, `Phase = phase`, `DemoScore = demo`, `PeerScore = peer`,
`LevelScore = level`, `Answer = need`; otherwise run the section loops from step 2.

The three scores are the readiness questions asked at all three points:
ready to lead a demonstration, ready to help a peer, and ready to move others
to the next level. Asking the same three each time is what makes the movement
visible on the room board.

### Running it in the room
Open `https://<owner>.github.io/<repo>/?room=1` on the projected screen, enter the
passphrase once, and leave it up. It refreshes every 15 seconds and has tabs for
Monday morning, End of Day 1 and End of Day 2.

### If you would rather not build this
A **Microsoft Form** with the same three questions gives you a live results chart
with no build at all — put the Form link on screen, and use the playbook fields
just as the participant's private copy. You lose the single-export-per-champion,
but you get the room view in five minutes.

---

## 5 · Rehearsing before the day

**You do not need to delete anything.** Add `?session=rehearsal` to the URL and
every submission is tagged with that session:

| URL | What it does |
|---|---|
| `…/?session=rehearsal` | Test run. Own local storage, own tag on every submission, amber REHEARSAL bar along the bottom so it cannot be confused with the real thing. |
| `…/?room=1&session=rehearsal` | Room board showing **only** the rehearsal answers. |
| `…/` and `…/?room=1` | The live day. Rehearsal rows are filtered out automatically. |
| `…/?room=1&all=1` | Everything, if you ever need to see it. |

So you can rehearse as many times as you like, from as many devices as you like,
and on the morning of 8 September the live board starts empty without anyone
touching the list. Rows written before sessions existed count as live.

If you would still rather clear the list, delete the rows where
`Session` equals `rehearsal` — but there is no need.

---

## 6 · Facilitator-only controls

The participant URL is plain. These flags reveal the extra controls:

| Flag | Reveals |
|---|---|
| `?facilitator=1` | **Start fresh** in the resource rail — clears the answers on *that device* (two taps to confirm). |
| `?room=1` | The projected room board. Implies facilitator. |
| `?session=…` | Tags the run, as above. |

This is a convenience gate, not a security boundary — a determined participant
could guess the flag. That is fine: the worst they could do is clear their own
answers on their own device. Nobody else's data is reachable from the page.

**Do not** keep two separate HTML files for facilitator and participant. Two
files means two things to keep in sync, two URLs to hand out, and a real chance
of sharing the wrong one on the day. One file plus a flag is the same pattern the
Southwest labs use.

---

## 7 · Rebuild the playbook against the URLs

```
python gen.py "<passphrase>" "<the flow URL>"
```

Then push. Submit one real test from the live page, confirm the rows land, and
delete the test rows.

---

## Two things to know

**The flow URL is a secret.** Anyone holding it can POST to this flow. It sits
inside the encrypted payload, so it is not readable without the passphrase — but a
participant could extract it after unlocking. Worst case is junk rows in this one
list, which is exactly why this flow should only ever write here and nowhere else.
To rotate it, regenerate the trigger URL in Power Automate and rebuild.

**Content type is `text/plain` on purpose.** The page posts with
`Content-Type: text/plain;charset=UTF-8` so the browser treats it as a CORS
"simple request" and never sends a preflight `OPTIONS`, which Power Automate does
not answer. Power Automate still parses the body as JSON because of the schema in
step 2. Do not change it to `application/json` — it will start failing in the
browser.

---

## Privacy note for the room

Say this out loud when you introduce the playbook: everything they write stays on
their own device unless they choose to press Send, the name field is optional, and
what is shared goes to the Manulife and Microsoft programme team so commitments can
be supported and recurring blockers fixed once.
