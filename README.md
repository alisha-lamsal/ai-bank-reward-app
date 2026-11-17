# AI-Powered Bank Rewards App

A prototype rewards section for a banking app that uses machine learning to recommend the right reward at the right moment. It predicts when a customer is likely to make a purchase in each category, then ranks their available rewards by **habit × location × point value**. It sends a notification only when a match is strong.

Built for the **FIS Innovation Challenge (October 2026)** and presented at FIS headquarters.

**Live demo:** https://bankfeatureapp.lovable.app

---

## How it works

**1. Predict purchase timing (Python, scikit-learn)**

Separate logistic regression models for coffee, food, grocery, online and ride purchases estimate how likely the customer is to buy in each category during a given hour. The features include:
- time of day (morning, lunch, afternoon, evening, late night) and weekday vs. weekend
- whether it's the customer's usual purchase time for that category, and how many hours away from it
- days since their last purchase in the category, and whether they bought recently
- spending pace compared with a normal week

**2. Score each reward**

```
match = visit rate × location score × points value
```

- **Visit rate:** the model's predicted purchase probability
- **Location:** 1.0 at the store, then lower with distance (0.6 within half a mile, 0.3 within a mile, 0.1 beyond that). Online rewards always score 1.0.
- **Points value:** how much the reward is worth to the customer

**3. Notify only when it matters**

The top reward triggers a notification only above a match threshold, within a daily notification limit and outside the customer's quiet hours. Otherwise the app stays quiet.

**Example (simulated student customer):**

| Moment | Top pick | Match | Action |
|---|---|---|---|
| Tuesday 8 AM, library | Free drink at campus café | 0.34 | Notify |
| Tuesday noon, student union | 15% off lunch | 0.30 | Notify |
| Sunday 3 PM, town center | $3 off groceries | 0.12 | Notify |
| Wednesday 9 PM, dorms | (none strong enough) | 0.07 | Stay quiet |

## Model results

The models were trained on 5,600 hourly records of simulated purchase history for one student customer and evaluated on hours held out from training:

| Category | Purchases | AUC |
|---|---|---|
| Food | 119 | 0.82 |
| Coffee | 77 | 0.74 |
| Ride | 52 | 0.86 |
| Grocery | 24 | 0.73 |
| Online | 14 | 1.00* |

*The online model has very few purchases, so its perfect score likely reflects the small sample rather than real-world accuracy.

> The purchase data is **synthetic**. It was generated to model a realistic student's spending habits. No real customer data was used.

The trained coefficients are exported to the app (`src/lib/model.ts`), where scoring runs entirely in the browser.

## App features

- **For You:** the top reward right now, with a breakdown of how its score was calculated
- **Learning from feedback:** skipping a suggestion lowers the score for that category during the session
- **Redemption:** pay with bank points, use offers at checkout, and track total savings
- **AI assistant:** answers questions about rewards and spending
- **Demo controls:** change the time, location and spending pace to see recommendations update

## Tech stack

- **Model:** Python, pandas, NumPy, scikit-learn (logistic regression)
- **App:** React, TypeScript, TanStack Start, Tailwind CSS; the front end was prototyped with Lovable
- **Testing:** Vitest unit tests for scoring, redemption, notification and preference rules

## Files

- `model/reward_prediction_model.ipynb`: generates the simulated data, trains the models and exports the coefficients
- `src/lib/model.ts`: the trained model and scoring engine
- `src/lib/`: business logic and tests for rewards, redemption, savings, notifications and preferences
- `src/routes/`: app screens

## Run locally

```sh
git clone https://github.com/alishalamsal42-byte/bankfeatureapp.git
cd bankfeatureapp
npm install
npm run dev
```

Run the tests with `npm test`.
