---
layout: post
title: making an append-only log in google sheets
---

as a software engineer I’ve developed an obsession with hoarding data. deleting or overwriting data just rubs me the wrong way - disk space is cheap so just make a copy! so when I found myself opening up a new google sheet to make a quick kanban board that I could share with a teammate my first instinct was to structure it as an append-only log.

append-only logs show up everywhere: database write-ahead logs, kafka topics, git histories. the trick is simple - never rewrite the past, only append new facts. the benefit is that you can always reconstruct how you got to any current state. I wanted that same property in sheets, plus a friendly read-only view so nobody had to scroll through every update just to find "what’s the latest?"

here's the sheet if you want to follow along at home, or adjust for your own uses: <https://docs.google.com/spreadsheets/d/1uSnwpaANF7Ng_4-tehIlXysvoYlZJUtFmdOPUqQlkdg/edit?usp=sharing>

the `log` sheet is the raw feed. four columns: `date`, `task` (basically the topic), `status` (essentially an enum that's used for coloring the columns in the view), and a free-text `update`. `date` is plain text and I stamp it with `ctrl+;` (or type the full timestamp). `status` uses a dropdown for convenience. the rule is absolute: never edit or delete a row - append another one instead.

![append-only log sheet](/assets/google-sheet-append-only-log/log.png)

the `view` sheet has one row for each text - the most recently appended row. so it functions as a "current status" view into every task.

```
=LET(
  headers, log!1:1,
  data, log!A2:Y,
  data_idx, {MAP(INDEX(data,,1), LAMBDA(r, ROW(r))), data},
  idxs, QUERY(data_idx, "select max(Col1) where Col3 is not null group by Col3", 0),
  updates, FILTER(data_idx, ISNUMBER(MATCH(INDEX(data_idx,,1), INDEX(idxs,,1), 0))),
  reversed, SORT(updates, INDEX(updates,,1), FALSE),
  body, CHOOSECOLS(reversed, SEQUENCE(1, COLUMNS(reversed)-1, 2, 1)),
  VSTACK(headers, body)
)
```

`data_idx` glues the original row number onto each record, `QUERY` pulls the newest row for every status bucket, `FILTER` keeps those winners, and `SORT` drops the freshest entries at the top. `CHOOSECOLS` strips the helper column and `VSTACK` restores the header row. the result is a read-only sheet that always shows the latest note per status without touching the underlying history.

add some conditional formatting ("Custom formula" ➡️ `=$C2="done"` ➡️ green cell as an example) and tada:

![query sheet with compacted view](/assets/google-sheet-append-only-log/query.png)

but why bother (besides fending off that pathological need to never lose data)? it unlocks all sorts of future analysis - most of which I never do, but I sleep better knowing I _could_. I can ask how many updates hit `blocked` last month, chart throughput over time, or export the whole thing into bigquery for a real query. if I want new metadata later I just add another column and it flows through automatically. append-only turns a throwaway sheet into something I can keep mining long after the kanban board is gone.

(p.s. although the sheet and this blog post uses kanban as an example, if you change the `task` column to a `topic` column you arrive at a more generic append-only log !)
