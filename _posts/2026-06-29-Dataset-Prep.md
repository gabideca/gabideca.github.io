---
title: "LKML5Ws Dataset Sentiment Analysis (1/2)"
date: 2026-06-29 09:19:14 -0300
categories: [Dataset]
tags: [Sentiment Analysis, linux, patch, contribution]
---

## Huh? Dataset?
Yeah that's a natural question following what was covered in the previous posts.
Worry not though, dear reader, everything will be explained.

This is just the third and last part of the MAC0470 - free software development subject! And a very special one at that.
This part was only meant for masters and phd students, but due to our performance in the previous two parts, our teacher, PhD Paulo Roberto Miranda Meirelles, allowed us to participate in this one as well. Very special indeed.

Enough fiddling though, let's cut to the chase. What are we doing this time around?

## The LKML5Ws Dataset
Simply put, the LKMLK5Ws dataset is a collection of interactions in the Linux contribution e-mail system.
What we set out to do was submit specifically the amd-gfx thread (due to memory and processing contraints) to a sentiment analysis and explore the gathered data.

## Rebuilding threads
The first hurdle we had to pass was actually rebuilding the threads, which was necessary since we want to analyze entire threads rather than isolated messages.

Here's how we did it, taking advantage of the 'in_reply_to' and 'message_id' columns in order to introduce our newly crafted 'thread_id' colunmn:

```python
import polars as pl
import re


def add_thread_ids(df: pl.DataFrame) -> pl.DataFrame:
    
    #thread_id = root message_id of the conversation.    

    #message_id -> parent message_id
    parent = dict(
        zip(
            df["message_id"].to_list(),
            df["in_reply_to"].to_list()
        )
    )

    cache = {}

    def find_root(msg_id):
        if msg_id is None:
            return None

        if msg_id in cache:
            return cache[msg_id]

        current = msg_id
        visited = set()

        while True:
            # Broken chain
            if current not in parent:
                root = current
                break

            p = parent[current]

            # Reached the beginning of the thread
            if p is None:
                root = current
                break

            # Parent missing from dataset
            if p not in parent:
                root = p
                break

            # Protect against malformed data
            if current in visited:
                root = current
                break

            visited.add(current)
            current = p

        for node in visited:
            cache[node] = root
        cache[msg_id] = root

        return root

    thread_ids = [find_root(mid) for mid in df["message_id"]]

    return df.with_columns(
        pl.Series("thread_id", thread_ids)
    )



```


## E-mail Body Cleanup
Beautiful clear of that last hurdle, but the hurdles have np intention of stopping. Next up: cleaning the e-mail body.

Since our focus is on natural langauge text, code in the messages could very well fog our data.
Hence, through certain tags, pattern checking and a few other heuristics, we aimed to extract the body text:

```python
df = pl.read_parquet("parquets/amd-gfx_list_data.parquet")

df = df.filter(pl.col("has_patch_tag"))

df = add_thread_ids(df)

print("dataset divido em threads:")
print(df)

```

## Aplicando a limpeza ao corpo dos e-mails

A função clean_email() é responsável por limpar o email, como dito anteriormente.


```python
df = df.with_columns(
    pl.col("raw_body")
      .map_elements(clean_email, return_dtype=pl.String)
      .alias("clean_body")
)

print("raw_body limpo:")
print(df['clean_body'])

```

    shape: (134_774,)
    Series: 'clean_body' [str]
    [
    	"AMD General"
    	"<Pratik.Vishwakarma@amd.com> w…
    	"<Pratik.Vishwakarma@amd.com> w…
    	"<Pratik.Vishwakarma@amd.com> w…
    	"<Pratik.Vishwakarma@amd.com> w…
    	…
    	"Since we now raise the clocks …
    	""
    	"Was never used as far as I can…
    	"Am 21.07.2016 um 09:46 schrieb…
    	"Change-Id: If00d5b97ba9e30f9b7…
    ]

A simple validity check then ensued:

```python
from pathlib import Path
import polars as pl

OUTPUT_FILE = "first_30_threads.txt"

# Get the first 30 thread IDs
first_threads = (
    df
    .select("thread_id")
    .unique()
    .head(30)
    .to_series()
    .to_list()
)

with open(OUTPUT_FILE, "w", encoding="utf-8") as f:

    for thread_num, thread_id in enumerate(first_threads, start=1):

        f.write("=" * 100 + "\n")
        f.write(f"THREAD {thread_num}\n")
        f.write(f"THREAD ID: {thread_id}\n")
        f.write("=" * 100 + "\n\n")

        thread = (
            df
            .filter(pl.col("thread_id") == thread_id)
            .sort("date")
        )

        for email_num, row in enumerate(thread.iter_rows(named=True), start=1):

            f.write("-" * 100 + "\n")
            f.write(f"EMAIL #{email_num}\n")
            f.write("-" * 100 + "\n")

            f.write(f"Message-ID : {row['message_id']}\n")
            f.write(f"In-Reply-To: {row['in_reply_to']}\n")
            f.write(f"From       : {row['from']}\n")
            f.write(f"To         : {row['to']}\n")
            f.write(f"Date       : {row['date']}\n")
            f.write(f"Subject    : {row['subject']}\n")
            f.write(f"Patch?     : {row['has_patch_tag']}\n")
            f.write("\n")

            f.write("BODY\n")
            f.write("~" * 100 + "\n")
            f.write(str(row['clean_body']))
            f.write("\n")
            f.write("~" * 100 + "\n\n")

        f.write("\n\n")

print(f"Saved to {OUTPUT_FILE}")
```

    Saved to first_15_threads.txt


## Calculating the Score
At last, the good stuff: the sentiment analysis itself. We chose to use VADER, a specialized tool trained on social media posts with a great advantage due to being lightweight.
This makes it faster and less costly than other more complex models, such as transformer-based tools, while still maintaining good accuracy.

VADER scores a given text between [-1, 1], being -1 the most negative and 1 the most positive.

```python
import polars as pl
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer

vader = SentimentIntensityAnalyzer()

def vader_score(text: str) -> float:
    if not text or text.isspace():
        return 0.0

    return vader.polarity_scores(text)["compound"]

```

    /home/gui/main_venv/lib/python3.10/site-packages/tqdm/auto.py:21: TqdmWarning: IProgress not found. Please update jupyter and ipywidgets. See https://ipywidgets.readthedocs.io/en/stable/user_install.html
      from .autonotebook import tqdm as notebook_tqdm
    Warning: You are sending unauthenticated requests to the HF Hub. Please set a HF_TOKEN to enable higher rate limits and faster downloads.
    Loading weights: 100%|██████████| 201/201 [00:00<00:00, 30811.17it/s]
    [transformers] [1mRobertaForSequenceClassification LOAD REPORT[0m from: cardiffnlp/twitter-roberta-base-sentiment-latest
    Key                         | Status     |  | 
    ----------------------------+------------+--+-
    roberta.pooler.dense.weight | UNEXPECTED |  | 
    roberta.pooler.dense.bias   | UNEXPECTED |  | 
    
    Notes:
    - UNEXPECTED:	can be ignored when loading from different task/architecture; not ok if you expect identical arch.


Finally, we apply VADER to each generated line in our dataset and save it onto a new .parquet file ready for our next step :)

```python

texts = (
    df["clean_body"]
    .fill_null("")
    .to_list()
)

print("setting up vader score")
vader_scores = [vader_score(t) for t in texts]

df = df.with_columns([
    pl.Series("vader_score", vader_scores),
])

print(
    df.select(
        "clean_body",
        "vader_score",
    ).head()
)

print(df['vader_score'])
df.write_parquet("amd-gfx_list_data_SCORED.parquet")

```

    setting up vader score
    shape: (5, 2)
    ┌─────────────────────────────────┬─────────────┐
    │ clean_body                      ┆ vader_score │
    │ ---                             ┆ ---         │
    │ str                             ┆ f64         │
    ╞═════════════════════════════════╪═════════════╡
    │ AMD General                     ┆ 0.0         │
    │ <Pratik.Vishwakarma@amd.com> w… ┆ 0.0         │
    │ <Pratik.Vishwakarma@amd.com> w… ┆ 0.0         │
    │ <Pratik.Vishwakarma@amd.com> w… ┆ 0.0         │
    │ <Pratik.Vishwakarma@amd.com> w… ┆ 0.0         │
    └─────────────────────────────────┴─────────────┘
    shape: (134_774,)
    Series: 'vader_score' [f64]
    [
    	0.0
    	0.0
    	0.0
    	0.0
    	0.0
    	…
    	-0.25
    	0.0
    	0.0
    	0.3947
    	0.0
    ]
