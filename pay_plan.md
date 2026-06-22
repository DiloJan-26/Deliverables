To make your ₹99/month plan profitable while maintaining a good user experience, we have to look at the math.

We need to leave a safe profit margin (for example, target spending no more than **₹50–₹60** out of the ₹99 on API costs per user so you can cover taxes, app store cuts, and database hosting).

---

## The True Cost Breakdown

1. **Layout OCR (Sarvam Vision):** Costs you **₹0.50 per page** (Flat rate).
2. **Scan-to-Word (Gemini 2.5 Flash-Lite):** Costs you roughly **₹0.03 per page** (Calculated based on medium-density 500-word output).

Because Sarvam is **16x more expensive** than Gemini, your page limits will change dramatically based on which feature the user interacts with.

---

## 📊 Feature Combos & Page Limits (Per User / Per Month)

Here is how you should structure the maximum page limits to fit cleanly into your ₹99/month plan:

| Combo Type                                         | Main Feature Behavior                                                      |              Recommended Page Limit / Month | Why this limit?                                                                                                     |
| -------------------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------: | ------------------------------------------------------------------------------------------------------------------- |
| **Combo 1:**<br>Only Layout OCR                    | User processes highly structured text while preserving original alignment. |                   **Max 100 pages / month** | 100 pages × ₹0.50 = **₹50 cost**. This leaves a healthy 50% margin.                                                 |
| **Combo 2:**<br>Only Scan-to-Word                  | User only wants to turn photos/PDFs into cleanly editable Word files.      |                 **Max 1,500 pages / month** | 1,500 pages × ₹0.03 = **₹45 cost**. Gemini is very cheap, so you can be generous here.                              |
| **Combo 3:**<br>Balanced Mix<br>*(Best for Users)* | User needs a day-to-day balance of layout parsing and standard conversion. | **60 Layout Pages**<br>+ **500 Word Pages** | 60 × ₹0.50 = ₹30<br>500 × ₹0.03 = ₹15<br>Total = **₹45 cost**. Gives users flexibility without hurting your wallet. |

---

## 🛑 Setting Fair Limits Per Single Attempt (Per Document Upload)

To prevent a single user from crashing your system or burning your entire API budget in one massive upload, you must enforce a **per-attempt page limit** inside the app.

### 1. If using Layout OCR (Sarvam)

* **Per-Attempt Limit:** **Max 10–15 pages per PDF/upload.**
* *Reason:* Processing a 50-page document in one go will cost you ₹25 instantly on a single user action. Forcing a smaller cap forces them to consider what they actually need to extract.

### 2. If using Scan-to-Word (Gemini 2.5 Flash-Lite)

* **Per-Attempt Limit:** **Max 50–100 pages per PDF/upload.**
* *Reason:* Gemini 2.5 Flash-Lite handles large context windows beautifully and cost-efficiently. A 50-page document will only cost you roughly ₹1.50 to execute.

---

## 💡 Recommendation for Your UI

Instead of giving users 3 separate complex plans, present it to them as a single **"Credit System"** under the ₹99 plan.

Give the user **100 Credits per month**:

* 1 page of Layout OCR = **1 Credit**
* 1 page of Scan-to-Word = **0.06 Credits** (or make 15 pages of Word equal 1 Credit).

This gives them total freedom to use the app however they want while perfectly protecting your profit margin!

======================================

Yes, absolutely! Dynamically tracking actual token and API spending inside your backend is completely doable. In fact, it is the **most professional way** to build a modern SaaS application because it entirely eliminates plan abuse while giving users maximum flexibility.

Instead of hard limits, you are setting up a **Meted Wallet System** tied to their monthly subscription cycle.

---

## 🛠️ The System Logic: How It Works

1. Every month, when the user pays ₹99, your database resets their `current_month_spending` to **₹0.00**.
2. Every time they make an API request, your backend calculates the exact cost of that specific call in real-time.
3. You add that cost to their `current_month_spending`.
4. If `current_month_spending` reaches **₹80.00**, your backend freezes further API requests and prompts them to: *"You have utilized your monthly plan limit. Renew your plan early or upgrade to continue."*

---

## 💻 Technical Implementation (How to calculate the cost)

You need to write a middle-layer code block in your backend server (Node.js, Python, or Go) that intercepts the API responses. Both Google and Sarvam return exact metrics that you can use to compute costs.

### 1. Tracking Layout OCR (Sarvam Vision)

Sarvam’s pricing is straightforward because it is based on pages, not variable text lengths.

* **The Math:** Every successful response from Sarvam contains the page count.
* **Formula:** $\text{Cost} = \text{Pages Sent} \times 0.50$
* *Example:* If a user uploads an 8-page document, add ₹4.00 directly to their tracker.

### 2. Tracking Scan-to-Word (Gemini 2.5 Flash-Lite)

Gemini's pricing depends entirely on input and output tokens. Fortunately, Google's API response always includes a `usageMetadata` object containing the exact token count.

```json
"usageMetadata": {
  "promptTokenCount": 350,
  "candidatesTokenCount": 750,
  "totalTokenCount": 1100
}

```

You capture these numbers in your code and apply your custom pricing logic:

$$\text{Input Cost (INR)} = \left(\frac{\text{promptTokenCount}}{1,000,000}\right) \times 8.35$$

$$\text{Output Cost (INR)} = \left(\frac{\text{candidatesTokenCount}}{1,000,000}\right) \times 33.40$$

*(Note: Calculated at current developer rates of $0.10/1M input tokens and $0.40/1M output tokens, mapped to an ₹83.50/$1 conversion rate).*

---

## 🏛️ Database Structure (Draft Schematic)

You will need a table in your database (like MongoDB or PostgreSQL) tracking the user's plan state. It should look something like this:

```json
{
  "userId": "usr_948201",
  "subscriptionPlan": "Basic_99_INR",
  "billingCycleStarts": "2026-06-01",
  "billingCycleEnds": "2026-07-01",
  "currentMonthSpendingINR": 42.65, 
  "maxAllowedSpendingINR": 80.00,
  "isActive": true
}

```

---

## 📱 User Experience Tip

Do not show raw numbers like *"You have spent ₹42.65"* on the mobile screen. It makes users feel anxious, like a taxi meter running.

Instead, visually translate it into a simple **"Usage Battery Bar"** or a **Percent Circle**:

* Convert the ₹80 maximum cap into **"100% Processing Power"** or **"800 Basic Credits"**.
* Deduct credits seamlessly in the background based on the real money spent. This keeps your margins completely safe while giving your app a clean, premium feel!
