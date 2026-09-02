# Live capture — how it works

**Status: built and connected.** Nothing here is left for you to set up.
This file is the record of what exists, so it can be repaired or rebuilt later.

---

## The pieces

| Piece | Where |
|---|---|
| Playbook | https://gd120msft.github.io/manulife-asia-champions-kickoff/ |
| Passphrase | `Manulife-Champions-HK-2026` |
| SharePoint list | **Asia Champion Kickoff Feedback** on `teams/CT-53698` |
| List GUID | `f7cd4cda-f276-4a0a-8b61-cb25dee6868f` |
| Collector flow | `Manulife Asia Champions - Collect Feedback` · `1bd4b864-51c8-4934-8010-21c2bc2f8c9f` |
| Room feed flow | `Manulife Asia Champions - Room Board Feed` · `71aa901c-f7e6-4674-a1fe-3ae4e2aa5859` |
| Environment | `839eace6-59ab-4243-97ec-a5b8fcc104e4` (default) |

Both flows run under the SharePoint connection owned by `giodepaula@microsoft.com`.

---

## What happens when someone presses a button

**"Share with the room"** on any of the three pulses sends one row:

```
kind=pulse · phase · session · name · functionName · market
ready / demo / peer / level (1-5) · need (free text)
```

**"Send my answers"** sends the whole worksheet, and the collector writes
**one row per answer** so the results are readable in SharePoint without
unpacking JSON.

Both post as `text/plain` on purpose. That keeps it a CORS *simple* request,
so the browser never sends a preflight `OPTIONS` — which Power Automate does
not answer. The collector therefore parses the body itself with
`@json(string(triggerBody()))` rather than relying on a trigger schema.

---

## The room board

Open `?room=1` on the projector. It polls the room feed flow, filters to the
current phase, and renders one bar chart per readiness question plus the
free-text responses.

- `?room=1` alone → the opening pulse (one chart)
- `?room=1&phase=close1` / `close2` → the closing pulses (three charts each)
- `?room=1&all=1` → ignore the session filter and show everything

The feed returns `Access-Control-Allow-Origin: *`, which is what lets the
GitHub Pages page read it.

---

## Rehearsing without polluting the day

Add `?session=rehearsal` to the URL. Every submission is tagged with that
session, and the plain `?room=1` board filters it out. **Nothing needs
deleting afterwards** — test rows simply never appear on the live board.

The facilitator "Start fresh" button is hidden unless you add
`?facilitator=1`, so participants cannot wipe their own work by accident.

---

## List columns

`Title` · `SubmissionId` · `SubmittedAt` · `Session` · `EntryType`
`ParticipantName` · `ParticipantRole` · `FunctionName` · `Market`
`Section` · `Question` · `Answer` · `AnswerCount` · `Phase`
`ReadyScore` · `DemoScore` · `PeerScore` · `LevelScore`

`EntryType` is `Pulse` or `Worksheet`. The room feed reads only `Pulse` rows.

---

## If something breaks on the day

1. **Nothing appears on the board.** Check the flow run history in Power
   Automate. Every submission is also saved in the participant's browser, so
   nothing is lost — they can press Send again later, or Export.
2. **A participant sees "That did not go through".** Their answers are still
   in local storage. Have them use **Export my answers** and send the file.
3. **The board shows rehearsal data.** You are on `?room=1&all=1`. Drop `all=1`.
4. **Total failure.** The playbook works fully offline; only the live
   share-back stops. Run the pulses on a flip chart.

---

## Rebuilding

```
python C:\temp\mlf\build\gen.py "Manulife-Champions-HK-2026" "<collector url>" "<room feed url>"
```

Trigger URLs come from `C:\temp\mlf\flow-urls.json` (regenerate with
`flowurls.py`). Flow definitions live in `wireflows.py` — re-running it
re-applies both flows from scratch.
