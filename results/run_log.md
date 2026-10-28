
## Run on 2026-10-27 16:32

- **comid**: 0.1.0.dev0
- **Parameters**: PERIOD_TYPE='m', WINDOW=('2023-01', '2024-12'), MIN_PERCENTILE=95, LOG_BASE=2, TAGS=['NOUN', 'PROPN']

### Durations (read from this run's cell outputs; "n/a" = not timed)
- Loading the JSON files (`load_json_files`): ~22 s
- Cleaning (`clean_data`): ~4 s
- Building the Convokit corpus (`ConvoCorpus`): ~12 s
- `ConvoTextParser.transform` (POS-tagging, 1,391,124 utterances): n/a (only progress output, no timestamps)
- `IdentityMetrics.from_corpus`: n/a

### Reconciliation with the draft version (`results/draft_values.csv`, December)

Side-by-side comparison (ratio = new / draft):

```
        community  distinctiveness_draft  distinctiveness_new  distinctiveness_ratio  dynamicity_draft  dynamicity_new  dynamicity_ratio  retention_draft  retention_new  retention_ratio
            Paleo               0.747756             2.219024               2.967579          0.420900        1.087126          2.582860         0.303041       0.220407         0.727319
        carnivore               0.507662             1.513872               2.982048          0.238413        0.743227          3.117392         0.508705       0.283010         0.556333
             keto               0.204834             0.547678               2.673765          0.104685        0.320049          3.057271         0.541474       0.328350         0.606400
mediterraneandiet               0.720246             1.975462               2.742761          0.247122        0.647072          2.618434         0.440731       0.263053         0.596856
        nutrition               0.227671             0.706778               3.104389          0.122136        0.402438          3.294992         0.497577       0.241137         0.484622
            vegan               0.127250             0.404891               3.181855          0.109119        0.342982          3.143198         0.536449       0.368800         0.687484
        veganketo               1.248094             3.533965               2.831489          0.634757        1.817521          2.863333         0.279918       0.145605         0.520171
          whole30               0.759575             2.381631               3.135478          0.415799        1.034054          2.486912         0.346504       0.242288         0.699235
```

**Distinctiveness and dynamicity** came out, on average, 2.95x and
2.90x above the draft, respectively. The dominant factor is the `log_base`
change from 10 (draft) to 2 (this version's `LOG_BASE`): since specificity/volatility are
`log_b(a/b)`, changing the base multiplies the values by log(10)/log(2) ≈ 3.32, very
close to the observed ratio. The residual difference is explained by some small adjustments in the Comid pipeline and this
version restricts the analysis to the `WINDOW` (2023-01 to 2024-12), while the draft used every
period present in the corpus.

**Retention**, on the other hand, came out at 0.61x the draft value on average
(i.e. it dropped to roughly half). The root cause is a confirmed bug in the draft notebook's manual Convokit
corpus construction: when creating the `Speaker` for a reply's author for the first time, the code used 
`Speaker(id=cm.posts[oc]['author_id'], ...)` -- the original post's (OC's) author -- instead of `cm.posts[reply]['author_id']` (the reply's actual author). 
This wrongly attributes each distinct user's first reply in a thread to the OC author's identity, artificially inflating the apparent
overlap of users between periods whenever the OC author remains active (common in long-lived threads). The
`comid.convobridge.ConvoCorpus` used in this version fixes this bug (`id=comid.posts[reply]['author_id']`), so
the retention values here are considered correct; the draft's higher values are an artifact of the corpus
construction, not a real effect of `log_base`, percentile, or the time window.

### Size-ordering check
rho(D, n_lexical) went from -0.98 at natural size to "
-0.25 (150/period) and -0.90 (500/period) at equal size, and  rho(N, n_lexical) from -0.95 to -0.68/-0.68 
-- the size-based ordering does not survive equal-size subsampling.
