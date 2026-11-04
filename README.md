# Shopper: browsing patterns and purchase conversion

Я разбил 500 сессий интернет-магазина на группы по поведению на сайте (K-means) и посмотрел, как часто в каждой группе заканчивается покупкой. В небольшой группе интенсивного просмотра товаров купили 9 из 16 сессий (56,3%), среди «лёгкого просмотра» 38 из 332 (11,4%). Группа маленькая, поэтому это гипотеза для проверки, а не вывод; в разделе Product hypothesis описано, как её проверить A/B-тестом.

**Question:** Which types of online store visits end in a purchase more often? In this sample, 9 of 16 product-intensive sessions ended in a purchase (56.3%), compared with 38 of 332 light-browsing sessions (11.4%). The small product-intensive group is a reason to validate this pattern before acting on it.

This HSE Basic Methods of Data Analytics course project uses a random sample of 500 sessions (seed 42) from the [UCI Online Shoppers Purchasing Intention dataset](https://doi.org/10.24432/C5F88Q) (C. Sakar and Y. Kastro, CC BY 4.0). Each row represents a visit. Revenue is a purchase indicator, not a monetary amount.

## Results

| Browsing profile | Sessions | Purchases | Conversion |
|---|---:|---:|---:|
| Light browsing | 332 | 38 | 11.4% |
| Administrative browsing | 119 | 19 | 16.0% |
| Information-intensive | 33 | 7 | 21.2% |
| Product-intensive | 16 | 9 | 56.3% |
| **All sessions** | **500** | **73** | **14.6%** |

Groups were formed using six standardized page-count and duration features. Purchase status was excluded from clustering and used only to describe the resulting groups. For each K (4 and 7), the best of 15 random starts was selected by within-cluster sum of squares. Four clusters were easier to interpret. For the product-intensive group, a 95% Wilson interval for the purchase proportion is approximately 33–77%; this interval treats the fitted partition as fixed.

## Product hypothesis

Customers browsing several products may benefit from a comparison tool. First collect product-level event data: the aggregate dataset cannot show which products were compared. Then randomize visitors when they open a third distinct product page, showing the tool to the treatment group only. Compare purchase conversion within seven days per randomized visitor; monitor page load time. The threshold is a proposed eligibility rule to validate, not a finding. No A/B test was conducted. The full-session clusters are retrospective and should not be used as experiment eligibility criteria.

**Test size, as an illustration.** For a baseline conversion of about 15% and a +3 percentage-point effect (15% to 18%), a two-sided test at α = 0.05 with 80% power needs about 2,400 visitors per group. The baseline for visitors who open a third product page is unknown in this dataset, so the real calculation must wait for event-level data.

## Limitations

- The sample has 500 sessions, so small groups (16 and 33 sessions) give wide intervals.
- The clusters were chosen on the same sessions on which conversion is reported. A small group with a high purchase rate is likely to look less extreme on new data.
- The analysis is descriptive: browsing profile and purchase are associated here, and this does not show that browsing causes buying.

## Notebooks and reproduction

- [K-means and cluster profiles](BDA_HW3_KMeans.ipynb)
- [PCA of browsing activity](BDA_HW2_PCA.ipynb)
- [Regression and error analysis](BDA_HW1_regression.ipynb)
- [500-session input](shopper_sample_500_seed42.csv)

To reproduce the conversion table, run HW3 and append this cell:

```python
profiles = {1: "Administrative browsing", 2: "Product-intensive",
            3: "Information-intensive", 4: "Light browsing"}
profile = pd.Series(best[4].labels_ + 1, index=df.index).map(profiles)
summary = df.groupby(profile)["Revenue"].agg(sessions="size", purchases="sum")
summary["conversion_pct"] = 100 * summary["purchases"] / summary["sessions"]
print(summary.round(2))
```

The ID mapping corresponds to the saved HW3 run. If refitting in a different environment, match labels by their centers before assigning profile names.
