## Problem: 
Lack of trust in users because properties in similar areas have different prices.
## Action:
Build a price discovery and price regulation model to estimate if the listed price is out of the estimated price range. 
## Business Issue:
1. **Trust**: Since this company is a two-sided marketplace where users can upload a listing of the property they want to sell and users can also browse the property catalog to see the properties they want to rent or buy, trust needs to be built on both sides:
	1. **Supply side**: Vendors / Retailers
		- Needs answer to the question: “Why is my listing not getting traction”
		- Build a price regulation model so that when a vendor adds a listing, the model returns a range for the listing price. This prevents the vendors from overpricing or underpricing the listing
	2. **Demand Side**: The Renters / Buyers
		-  Needs answer to the question: “Why are two properties in the same area different?”
		-  build a SHAP explainability.
## EDA:
1. Description column: extract number of amenities or sentiment analysis to know the luxuriousness of the apartment
2. price per sq ft distributions across sub-areas
3. property type with + ie 3+2 or 2+2:  this is where domain knowledge comes in. This convention is a Indian real estate convention 3+2 BHK means 3 bedrooms + 2 servant quarters or utility rooms.  it's not a truly 3+2 = 5 bedroom but its bigger than a true 3 bed room. so an assumption we can make is that it >3 and <5 so a 4 bed room.
   **The "shop" row.** One commercial property among 199 residential ones. I'd drop it. It's a fundamentally different asset class with different pricing dynamics — including it would just add noise. Mention in your notebook why you dropped it (different valuation model for commercial vs residential), and that in production you'd build a separate model for commercial listings.
4. create a new column, that only has decimal bedrooms.
5. Encoding:
	1. **Binary Yes/No columns (amenities: gym, pool, clubhouse, etc.)** — Just map Yes→1, No→0.
	2. **Ordinal data with a natural order** — Your BHK count after cleaning (1, 1.5, 2, 2.5, 3, ..., 6). This has a meaningful numeric ordering — a 3BHK is "more" than a 2BHK in a way that matters for price. Keep it as a number. Don't one-hot encode it, because that would destroy the ordinal relationship and the model would have to re-learn from scratch that 3 > 2 > 1.
6. **Amenity Score:** Rather than feeding 7 sparse binary amenity columns into the model, engineer a single amenity_score (0–7 count). Then show in EDA that higher amenity scores correlate with price. This demonstrates feature synthesis thinking.  I am thinking if we should consider a dynamic amenity score where a few amenities would have higher weightage.
7. - Use 5-fold cross-validation  as the primary evaluation method, not a single train/test split - For sub-area encoding, use target encoding with cross-fold smoothing (not one-hot, which would create 39 sparse columns on 200 rows) — or group rare sub-areas into an "Other" bucket
## Models:
1. Start with linear regression as baseline benchmark:
	1. if features not correlated → L1
	2. if features correlated → L2
	3. if mix of correlated and non correlated → ElasticNet
2. try linear regression ElasticNet: for regression standardize.
3. XGBoost regression: skip standardization.
4. build a separate model for commercial listings
## Notes:
1. **Linear Regression:**
	1. **Ridge (L2 regularization)** adds a penalty proportional to the _square_ of each coefficient. The loss function becomes: squared errors + λ × (sum of coefficients²). The effect is that it _shrinks_ all coefficients toward zero, but never exactly to zero. Every feature stays in the model, just with dampened influence. It's good when you believe most features are somewhat relevant and you just want to prevent any single one from dominating.
	2. **Lasso (L1 regularization)** adds a penalty proportional to the _absolute value_ of each coefficient. The loss becomes: squared errors + λ × (sum of |coefficients|). The critical difference is that L1 can shrink coefficients _all the way to zero_, effectively removing features from the model. It performs automatic feature selection. This is powerful when you suspect many features are irrelevant or redundant — which is likely your case, since the amenity columns (gym, pool, clubhouse, etc.) are probably highly correlated with each other.
	3. **ElasticNet (L1 + L2)** combines both penalties. You get a mixing parameter α that controls the ratio — at one extreme it's pure Ridge, at the other it's pure Lasso. Why would you want both? Lasso has a known weakness: when features are highly correlated (like your amenity columns), it tends to arbitrarily pick one and zero out the others. Ridge handles correlated features more gracefully by shrinking them together. ElasticNet gives you the feature selection of Lasso with the stability of Ridge for correlated groups.
2. **Standardization (StandardScaler)** transforms each feature to have mean=0 and standard deviation=1. It doesn't bound the values — outliers can still be large numbers. Use this when your model assumes or benefits from normally distributed inputs. For your case study, this is what you want for Ridge/Lasso regression, because the regularization penalty is sensitive to coefficient magnitude, and coefficients are sensitive to feature scale. If "Property Area" is in the hundreds and "Township Acreage" is in the thousands, the model would penalize the area coefficient more harshly just because of the units — not because it's less important. Standardization puts them on equal footing.

## Assumptions:
1. Equal weights to the amenities in the 

## PPT:
1. How i would handle model drift 
2. how to monitoring
3. how i would handle a new class thats not in training data (class “others” → own class promotion and then re training when a certain threshold is met)

## Decision Justification: 
1. Prices in lakh:
	1. **null handling:** fit a LR model to predict price for 3bhk based on area.
	2. **outlier analysis:** 
		1. 923 lakh and 930 lakh properties dropped: 930 lakhs (93 million) is way too much for a 3 bhk property in Pune. These numbers could be justified if the property area were correspondingly large. In this case , the 923 lakh property is 923 sq ft and the 930 lakh property is 930 sq ft. This looks like an error.
		2. log transforming because dropping the outliers would cap the model's price range at ~200 lakhs, meaning premium listings would be priced like mid-range ones — breaking vendor trust (Pillow's core problem). Encoding on log-transformed prices keeps the means stable.
2. Why i dropped shop row: outlier. in production, commercial estates will have a separate model
3. sub_area: since bavdhan and bavdhan budruk are the same, we can club them in one. 
4. property_area_in_sq_ft: 





```
"""
The "shop" row: One commercial property among 199 residential ones. It is a fundamentally
      different asset class with different pricing dynamics — including it would add noise to the
      model. It is dropped here. In production, a separate model would handle commercial listings.
"""

# Drop shop row (no_of_bedrooms is NaN)
# df = df.dropna(subset=['no_of_bedrooms']).reset_index(drop=True)
df
```