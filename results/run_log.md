
## Run on 2026-11-15 11:47

- **comid**: 0.1.0
- **Parameters**: PERIOD_TYPE='m', WINDOW=('2023-01', '2024-12'), MIN_PERCENTILE=95, LOG_BASE=2, TAGS=['NOUN', 'PROPN']

### Durations (measured with a stopwatch around each step in this run)
- Loading the JSON files (`load_json_files`): 28.1 s
- Cleaning (`clean_data`): 11.9 s
- Building the Convokit corpus (`ConvoCorpus`): 50.3 s
- `ConvoTextParser.transform` (POS-tagging, 1,391,124 utterances): 51 min 49 s
- `IdentityMetrics.from_corpus`: 1 min 49 s

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
the retention values here are considered correct.

This is demonstrated rather than asserted: replaying the draft's construction over this run's corpus
reproduces the eight December retention values (max absolute difference 5.6e-17), and
changing only that one line gives 0.53x the December values on average (range
0.42 to 0.62). The remaining factor up to this version's values
(1.16x on average) comes from the other differences between the draft's retention and
`IdentityMetrics`: UTC month keys instead of local time, the `WINDOW`, and averaging only over months with
active users instead of counting empty months as zero; it is not decomposed further here.

### Size-ordering check (correlations against natural size)

All correlations use each community's natural size (`n_lexical` of the `window=on` run; `n_active` gives the same rank correlations), not the size left after sampling. Spearman rho, `window=on` -> 150/period -> 500/period -> 30/period:

- eight communities: rho(D) -0.98 -> -0.21 -> -0.83 -> -0.43; rho(N) -0.95 -> -0.40 -> -0.62 -> -0.43.
- seven communities, without veganketo: rho(D) -0.96 -> +0.18 -> -0.75 -> -0.14; rho(N) -0.93 -> -0.11 -> -0.43 -> -0.14.

The size-based ordering does not survive equal-size subsampling (verdict on the seven communities that are really equalised: same sign and at least half of the baseline magnitude, for D and N at 150 and 500 per period).

`sample_per_period=30` equalises all eight communities at the cost of noise. Standard deviation across seeds 0-9, mean (max) over communities: D 0.064 (0.094), N 0.072 (0.107).
