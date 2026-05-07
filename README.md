# Usenet Thread Reconstruction and Discourse Analysis

This project explores how raw Usenet discussion archives can be transformed into structured data for thread reconstruction, reply-chain analysis, and future computational analysis of online discourse.

The notebook was developed after building a command-line email client, as a way to apply similar ideas — message parsing, headers, identifiers, references, and message bodies — to a larger real-world communication dataset.

## Project Motivation

Usenet archives contain long-running public discussions in a format similar to email. Each post includes metadata such as sender information, subject lines, message IDs, and references to previous messages. These fields make it possible to reconstruct conversation structure from raw text.

The goal of this project was to move from a messy compressed Usenet archive toward a structured dataset that could support analysis of:

- reply relationships between messages,
- discussion depth,
- thread structure,
- message metadata,
- and, eventually, patterns in political or social discourse.

This kind of work connects naturally to interests in cognitive science, computational social science, natural language processing, and human communication networks.

## Dataset

The project uses a Usenet archive from the `alt.politics.british` newsgroup.

Two compressed files were used:

```text
alt.politics.british.20140613.mbox.csv.gz
alt.politics.british.20140613.mbox.gz
```

The `.csv.gz` file contains message-level metadata, while the `.mbox.gz` file contains the raw message archive.

The metadata file includes fields such as:

```text
date
msg_id
from
newsgroups
subject
references
start
length
```

## What the Notebook Does

### 1. Loads and inspects the metadata

The notebook first loads the compressed metadata file with `pandas`, assigns column names, removes an invalid or non-message first row, and inspects the structure of the resulting DataFrame.

This includes checking:

- the first rows of the dataset,
- data types,
- missing values,
- message IDs,
- subjects,
- and reference fields.

### 2. Loads the raw Usenet archive

The raw `.mbox.gz` archive is then loaded separately. Because the raw archive is not a clean table, the notebook treats the file as line-based text data.

The raw message lines are stored in a column named:

```text
msg_parts
```

This makes it possible to manually inspect and parse the structure of individual messages.

### 3. Explores the raw message format

Before building the parser, the notebook inspects slices of the raw archive and searches for important Usenet/email headers, including:

```text
From:
Subject:
References:
Message-ID:
Xref:
```

This exploratory step is important because real-world archive data is messy. The parser had to be designed around the observed structure of the archive rather than around an idealized format.

### 4. Extracts message-level fields

The notebook then builds a custom parser using regular expressions.

For each message, it attempts to extract:

```text
index
sender
subject
references
groups
body
msg_id
```

The parser identifies key header lines and separates them from the message body. The cleaned results are stored in a new DataFrame called:

```python
df_cleaned
```

This converts the raw archive into a more usable structured dataset.

### 5. Reconstructs reply relationships

The project uses the `Message-ID` and `References` fields to infer reply relationships.

The basic idea is:

```text
If message B contains message A in its References field,
then message B is treated as a reply to message A.
```

The notebook builds a lookup table from message IDs to row indexes, then constructs a parent-to-child mapping:

```text
parent message ID -> list of reply message IDs
```

This creates a thread tree representing the structure of replies.

### 6. Calculates thread depth

After building parent-child relationships, the notebook creates a child-to-parent mapping and recursively calculates the depth of each message in a thread.

Depth is interpreted as:

```text
0 = root/original message
1 = direct reply
2 = reply to a reply
3 = deeper nested reply
```

This allows the dataset to capture how deep discussions become, not just how many messages exist.

### 7. Creates a thread link table

Finally, the notebook creates a simplified edge list:

```text
msg_id      references
child       parent
```

This structure is useful for later graph analysis, visualization, and network modeling.

## Main Technical Concepts

This project uses several important data-processing and analysis ideas:

- reading compressed datasets with `pandas`,
- handling messy raw text data,
- exploratory data inspection,
- regular expressions,
- parsing email/Usenet-style headers,
- extracting structured fields from unstructured text,
- reconstructing reply graphs,
- calculating recursive thread depth,
- preparing edge lists for graph analysis.

## Why This Matters

At a technical level, this project shows the process of transforming a raw communication archive into structured data.

At a broader level, it shows how computational methods can be used to study human communication. Usenet discussions are not just text; they are social interactions with structure, memory, replies, conflict, agreement, and topic development over time.

This makes the project relevant to areas such as:

- cognitive science,
- discourse analysis,
- computational linguistics,
- social network analysis,
- online political communication,
- natural language processing,
- and human-computer interaction.

## Current Limitations

This was an exploratory notebook, so the parser is still experimental.

Known limitations include:

1. **Message boundary detection is fragile**

   The current parser uses `From:` lines as part of the message-detection logic. This may fail if quoted message bodies also contain lines beginning with `From:`.

2. **References may contain multiple message IDs**

   In Usenet and email archives, the `References:` field can contain an entire chain of previous messages. The current approach treats the field too simply. A better approach would extract all message IDs and use the last valid reference as the likely direct parent.

3. **Newsgroup extraction needs improvement**

   The current regular expression for newsgroups may not correctly extract groups from `Xref:` lines because those lines do not necessarily begin with the group name.

4. **The parser should be modularized**

   The current logic is notebook-based. For a stronger project, the parser should be moved into reusable Python functions or classes.

5. **No visualization is included yet**

   The thread graph is prepared, but it has not yet been visualized using tools such as NetworkX, Plotly, or Graphviz.

## Possible Future Improvements

Future versions of this project could include:

- a cleaner standalone parser module,
- improved handling of multiline headers,
- robust extraction of all message IDs from `References:`,
- thread graph visualization,
- centrality analysis of active users,
- detection of long or controversial discussions,
- topic modeling,
- sentiment analysis,
- stance detection,
- comparison between different newsgroups,
- and network analysis of user interactions.

## Potential Cognitive Science Extension

This project could be extended into a cognitive science-oriented analysis of online discussion behavior.

Possible research questions include:

- How do online discussions branch and deepen over time?
- Which kinds of messages generate longer reply chains?
- Do certain subjects produce deeper or more fragmented conversations?
- How do people quote, reference, and respond to previous messages?
- Can thread structure reveal patterns of attention, disagreement, or social influence?
- How does the architecture of a communication system shape the way people reason and interact?

These questions connect computational data analysis with human cognition, communication, social behavior, and information processing.

## Technologies Used

```text
Python
pandas
NumPy
regular expressions
collections.defaultdict
Jupyter Notebook
gzip-compressed archive files
```

## Project Status

This is an early exploratory version of the project.

The current notebook successfully demonstrates the core idea: parsing raw Usenet archive data and reconstructing reply relationships from message metadata.

The next step would be to clean the code, improve the parser, and add graph-based analysis and visualization.

## Summary

This project converts raw Usenet archive data into a structured format and uses message identifiers to reconstruct discussion threads.

It began as a follow-up to an email client project and applies similar concepts — message headers, bodies, IDs, and references — to a larger public communication dataset.

The project can be developed further into a portfolio piece focused on computational discourse analysis, graph analysis, or cognitive science applications.
