Analyzing indicators of spelling uncertainty such as `(sp?)` in Reddit comments.

## Files

- `token_counts.csv`: mapping lowercase tokens to number of times that token was seen followed by an `(sp?)` marker in comments
- `examples_in_context.csv`: csv with a sample of relevant comments. Has 3 columns: comment id, token (the word followed by spelling marker), and full body of the comment. Restricted to tokens with count >= 10. Grepping this is useful for debugging.
- `clustering.ipynb`: Groups the tokens in `token_counts.csv` into clusters of alternative (mis)spellings of the same word.
- `treemaps.ipynb`: Generates visualizations of the clusters induced by `clustering.ipynb`.

## Methodology

### Data collection

My data was gathered using the following query on the Reddit comments dataset hosted on Google BigQuery:

```sql
SELECT id, REGEXP_EXTRACT(body, r"(?m:^[^&].* ([0-9A-Za-z\047-]+) ?\([sS][pP](?:elling)?\??\))") 
  token, body
FROM [fh-bigquery:reddit_comments.all]
HAVING token IS NOT NULL
```

This basically matches a word, an optional space, and indicators of spelling uncertainty such as any of the following: (sp?), (SP?), (sp), (spelling?), (Spelling).

(We insist the line on which the match occurs doesn't start with `&`. This is to avoid lines that start with `>` (encoded as `&gt;`), so that we don't double-count lines that are quoted in comment replies.)

### Cleaning

To generate my final list of top tokens, I did some manual work to weed out false positive spelling indicators, e.g.:

- abbreviations: skill points (sp), Sleep paralysis (SP), the Samajwadi Party (SP)
- use-mention distinction: I love how he put the (sp) in there
- tokens that belong to a wide variety of multi-word phrases: ____ Island, ____ disease, ____ acid

### Clustering

I clustered together word forms based on Levenshtein(sp?) distance, normalized by length. Again, you can see some hand-tuned special cases in `clustering.ipynb` to avoid false positive clustering associations. e.g. 'bechamel' is not a misspelling of 'deschanel'.
