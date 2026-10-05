# Price Confidence - Team Edge Case

Final submission pack.

Team Edge Case, IIT Roorkee. Meesho DICE Challenge S3 (pricing case).

## Demo video

[![Price Confidence demo video on Loom](https://cdn.loom.com/sessions/thumbnails/c300fa0f2e26450d9860fd87da4de9cf-with-play.gif)](https://www.loom.com/share/c300fa0f2e26450d9860fd87da4de9cf)

Watch on Loom: https://www.loom.com/share/c300fa0f2e26450d9860fd87da4de9cf

## What's in this folder

| File | What it is |
|---|---|
| `Price_Confidence_DICE_Round2.pptx` | The Round 2 deck: 10 slides including the cover (slide 9 is data, tech stack and running costs), plus appendix A1 with every source linked (DICE template, speaker notes on every slide). Slide 8 links to the team's demo video on Loom through a clickable storyboard, a button and a QR code, so it works in PowerPoint and in a PDF export. |
| `Price_Confidence_DICE_Round2.pdf` | The same deck as a PDF (11 pages, all links clickable). The demo video on slide 7 opens on Loom from the picture, the button or the QR code. |
| `Price_Confidence_Models_Explained.pdf` | Two-page explainer of the three models, shareable with the team |
| `OPEN_THIS_Price_Confidence_demo.html` | The working demo. Double-click it, or drag it into Chrome. Works offline. |
| `Price_Confidence_Synthetic_Dataset.xlsx` | The dataset with formula-driven checks |
| `code/` | Everything needed to regenerate the data, retrain the models and rebuild the demo |

**All data is synthetic.** It is generated from assumed buyer behaviour to demonstrate the method. Nothing here is a forecast for real Meesho sellers.


## Pipeline (run inside `code/`)

```
python generate_dataset.py     # 150 soap + 150 t-shirt listings, weekly from launch to 21 Sep 2026
python train_models.py         # fits A1, A2, B; hold-out backtest; writes model.json
python build_app.py            # model.json + engine.js + app_template.html -> dist/index.html
node test_engine.js            # pricing engine on five sample sellers
python truth_compare.py        # runs Seller 6's soap through the simulator's own buyer rules -> truth.json
node model_compare.js model.json   # compares the models' answers with that ground truth
python build_workbook.py       # optional: Excel workbook of the dataset
```

## The models

All three give every listing its own baseline (fixed effects), so effects are learnt from a listing changing over time, never from comparing good products with bad ones. A second step puts those baselines on listing quality, so a brand-new listing gets a starting level.

- **A1 Conversion:** orders per impression (Poisson, log-impressions offset, so exposure is handled and more views are never mistaken for better conversion). Inputs: price relative to the market median (quadratic), rating shown, reviews so far, festival weeks.
- **A2 Visibility:** weekly impressions (Poisson). Inputs: launch phase, recent sales vs the category norm, reviews, rating, category demand, sale weeks.
- **B Keep share:** RTO rate; return rate (logistic, rises with price above market, lower for better listings); spread of return rates between products; resale share; review rate.
- **Product scenarios:** a good product (top quarter) gets a better rating, a higher baseline in A1 and A2, and fewer returns; a weak one the opposite.

## Key results (synthetic data)

- +10% price: about -20% orders per impression for soap, -26% for t-shirts. Comparing different listings suggests only -7% and -10%.
- Selling at twice the category norm lifts next week's impressions by about 35%; new listings are shown about 1.8x at first.
- Backtest, 8-week orders on held-out listings: median error 31% (soap) and 41% (t-shirt), against 153% and 87% for a simple average.
- Checked against the simulator (Seller 6, 26 weeks, change vs starting at Rs 209):

| Plan | Good: model / simulator | Typical: model / simulator | Weak: model / simulator |
|---|---|---|---|
| Rs 129 for 4 weeks (below the Rs 145 floor) | -2,626 / -779 | -2,249 / -1,308 | -1,688 / -1,698 |
| Rs 169 for 4 weeks (above the floor) | +72 / +1,690 | -241 / +531 | -277 / -15 |

Both agree that launching below the floor loses money for every kind of product. The models are more cautious than the simulator about discounts above the floor: sellers in the data cut prices when sales are slow, which blurs the price effect. Only randomised price tests remove that, so the pilot includes them.

## Response to the external review

| # | Review point | Valid? | What we changed |
|---|---|---|---|
| 1 | Deck has 11 slides; brief allows 6-10 | Yes | Merged the lifecycle and funnel-signals slides. The deck is now 10 slides including the cover. |
| 2 | "Rs 380 per extra review" did not match the chart | Yes | The figure came from the typical-rating run while the chart showed good/weak runs. All launch-window figures now come from the good-product lines on the chart: week 4 is -Rs 2,175 vs +Rs 2,624, a Rs 4,799 gap for about 20 extra reviews, so about Rs 235 each. The slide shows the division. |
| 3 | 30-day rollout said the floor needs only data Meesho holds | Yes | Slide 10 now says the seller types making cost, packaging and gifts; Meesho fills fees, return and RTO rates and GST. Slide 3 adds a "Comes from" column; slide 8 lists who provides what. |
| 4 | Slide 4's 12-week profit needed a clear denominator | Yes | Now "about 69 orders placed, 58 of them paid; 58 x Rs 61 = Rs 3,537". The app says the same. |
| 5 | Slide 10 (now 9) did not state exposure treatment | Yes | A1 is orders per impression with a log-impressions offset; stated on the slide, in the app and in the PDF. |
| 6 | Competitor claims need traceable sources; "no tool" overclaims | Yes | Each tool row has a reference number; a numbered list with links is on the slide and full URLs are in the notes. The claim is narrowed to "none we found models full seller cost", and the gap box says three tools use cost partly. |
| 7 | The 15% willingness-to-pay premium should be framed as a demo finding | Yes | Labelled "in our demo data" on the slide, in the app and in the notes. After retraining, the figure is about 8%. |

While checking, we also fixed:
- The visibility model had the same quality trap as conversion. Fitted within listings, the effect of recent sales went from 0.007 to 0.30.
- "Good" and "weak" products now differ in conversion, visibility and returns, not only in rating.
- The launch-lower verdict was tied to the loss budget, so an empty budget always meant "not worth it". Worth-it and affordability are now separate.
- The app now shows a gentler option (a discount that stays just above the floor), which buys reviews for about Rs 147 each (Rs 1,103 / 7.5) and never goes below zero.

## Response to the second review

| # | Review point | Valid? | What we changed |
|---|---|---|---|
| 1 | "Rs 235 per extra review" did not match "Rs 4,799 / 20" (that gives Rs 240) | Yes, a display issue | The exact figure is Rs 4,799 / 20.4 extra reviews = Rs 235; the slide rounded the divisor to 20. Review counts are model averages, so the slide and the app now show them to one decimal (26.5 vs 6.1) and print the exact division. The gentler option had the same issue and now shows Rs 1,103 / 7.5 = Rs 147. |
| 2 | Rs 129 is about 11% below the Rs 145 floor, not 10% | Yes | The engine starts 10% below the floor and rounds to a price ending in 9, which lands at Rs 129, 11.0% below. The slide says "about 11%", and the app now states the actual percentage for whatever price is shown. |
| 3 | The Mercari claims were supported only by a Kaggle price-prediction challenge | Yes | Replaced with Mercari's own engineering blog on its price guidance system and Smart Pricing. That source also corrected our row: Mercari does suggest a starting price (and a floor) from the item's photo and text with neural-network models, so "Price for a new product" is now Yes, and Image AI and Text AI are documented. Smart Pricing only lowers the price, toward a floor the seller sets. The "ideas we borrow" line for Mercari now matches what the source says. |

## Final edits (team review)

- Slide 3: the dark box no longer repeats the floor; it now covers the very first price with no sales history. Card 03 no longer says "lower half", which contradicted Seller 6's Rs 209 start.
- Slide 5 (merged old slides 5 and 6): the five lifecycle moments as an arrow; the buyer funnel with what each drop-off means and what Price Confidence does; and a compact chart of 26-week profit at each launch price for good, typical and weak products (from the simulator that generated the data, 150 runs per price). A good product earns most near the typical price (Rs 179-189); a weak product earns about the same at any price from Rs 179 to 209, so a price cut does not rescue it; below the Rs 145 floor every product loses. Our model is more cautious about price cuts (it starts Seller 6 at Rs 209); a randomised price test in the pilot settles it.
- Slide 9 (new, in the main deck): data needed, how it runs, and monthly running costs for 10 million listings, about Rs 64,000 to Rs 1.25 lakh a month. Prices checked against AWS Mumbai on-demand rates in October 2026 at Rs 96.2 per dollar: c6i.xlarge $0.170/hour (about Rs 4.1 per vCPU-hour), g4dn.xlarge T4 GPU $0.579/hour (about Rs 56), S3 Standard $0.025 per GB-month (about Rs 2.4). The first version mixed T4 and L4 GPU prices (Rs 65/hour) and used an old exchange rate; the GPU line is now T4 only. Sizing (training hours, servers, GPU hours) are our estimates. Excludes data transfer, disks and salaries.
- Slide numbers and the appendix's cross-references were updated after the merge.
- Links checked: every hyperlink now opens the page its label names. Six source links on slide 6 and in the appendix pointed to other pages (Business Today, an eBay help page, the Mercari Kaggle challenge, Fibre2Fashion, an old Competera URL) and were reset to Meesho's Price Recommendation article (Meesho Tech on Medium; the old meesho.io blog address now returns 404), the eBay/ZenML talk, Mercari Engineering's blog, the Chain Store Age article and competera.ai/platform. The Amazon source is now Amazon's official Automate Pricing page, labelled as such. The 307-listings sheet uses its short OneDrive share link everywhere.
