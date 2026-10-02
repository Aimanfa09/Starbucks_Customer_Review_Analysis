# Starbucks Customer Feedback Intelligence & AI-Powered Customer Recovery System

**Author: Aiman Fathima**

Python Foundations & Gen AI Internship Assessment

## Project Overview

This project reads Starbucks customer reviews, finds the critical ones with simple rules, understands what customers are complaining about, ranks the cases by priority, and uses Google Gemini to draft apology emails for the top 3 cases. A person reviews the drafts before any outreach.

## Objective

Automate the first part of a support team's job: filter the critical feedback and draft a personalized, empathetic reply, without using machine learning for the filtering.

## Files

| File | What it is |
|---|---|
| Aiman_Fathima_Starbucks_Customer_Feedback_Analysis.ipynb | The main notebook |
| Aiman_Fathima_README.md | This file |
| reviews_data.csv | The Starbucks reviews dataset |
| requirements.txt | Python libraries needed |

## Dataset

`reviews_data.csv` has 850 rows and 6 columns:

- `name`: first name of the reviewer
- `location`: written as "City, ST"
- `Date`: review date text
- `Rating`: star rating from 1 to 5 (145 values are missing)
- `Review`: the review text
- `Image_Links`: links to images (not used)

There is no email column in this dataset.

## Data Cleaning

- Removed duplicate review texts (36 found), leaving 814 reviews.
- Missing `Review` and `location` values are filled with a simple default.
- Missing `Rating` values are not filled, because a guessed rating would be made-up data. Those reviews (110 after removing duplicates) are kept but left out of the rating analysis.
- Review text is cleaned with basic string methods: lowercase, remove apostrophes, special characters and numbers, remove extra spaces. The original text is kept for the emails.
- A `state` column is taken from the end of the `location` text.

## Rule-Based Logic

- **Critical reviews:** a rating of 1 or 2 stars. No machine learning.
- **Complaint keywords:** a function using `collections.Counter` finds the most common words in critical reviews, and a second count is made for a list of specific complaint words (rude, wrong, wait, charged and so on).
- **Complaint categories:** simple keyword dictionaries (Staff / Service, Product Quality, Waiting Time, Pricing / Billing, Order / Payment / App, Store Experience). A review with no matching word is marked Other / General.
- **Priority score:** a simple project-level rule, not a validated model.

| Rule | Points |
|---|---|
| 1-star review | +3 |
| 2-star review | +2 |
| Contains a strong complaint word | +2 |
| Long, detailed review (100 words or more) | +1 |
| Fits 2 or more categories | +2 |

Score 7 or more is Urgent, 5 to 6 is High, below 5 is Moderate.

## GenAI

The 3 top cases are chosen by a fixed rule: highest priority score first, then the longest review. For each one, a prompt with the rating, complaint category, priority and review text is sent to the Google Gemini API, asking it to act as a customer support agent and write a short, personalized, empathetic apology email. The prompt tells the model not to promise refunds, vouchers or investigations, not to take sides, and not to repeat employee names.

## Simulated Customer Outreach

The dataset does not contain customer email addresses, so the email stage is a simulated/test workflow. No real Starbucks customer is contacted. The notebook builds an email log table (Review ID, Rating, Complaint, Priority, Generated Response, Email Status) and the optional send code in the "Additional Upgrades" section can only send to a test address that I own.

## Human Review

The AI emails are only drafts. In the notebook I read each draft and add the approved Review IDs to a list called `approved_ids`. Only approved emails can be marked "Approved for Test".

## Architecture

```
                 CUSTOMER REVIEWS
                       │
                       ▼
                DATA CLEANING
                       │
                       ▼
             RULE-BASED ANALYSIS
              /        │        \
             /         │         \
        Rating     Keywords     Category
             \         │         /
              \        │        /
               ▼       ▼       ▼
                PRIORITY SCORE
                       │
                       ▼
                 TOP CRITICAL
                    CASES
                       │
                       ▼
                  GENAI MODEL
                       │
                       ▼
              PERSONALIZED EMAIL
                       │
                       ▼
              HUMAN REVIEW
                       │
                       ▼
              TEST EMAIL / LOG
```

## How to Run

1. Install Python 3.9 or newer and open a terminal in this project folder.
2. Install the libraries:

   ```
   pip install -r requirements.txt
   ```

3. Get a free Gemini API key (see below) and set it as an environment variable.
4. Start Jupyter and open the notebook:

   ```
   jupyter notebook
   ```

5. In the notebook choose Kernel > Restart and Run All. Keep `reviews_data.csv` in the same folder as the notebook.
6. Save the notebook after it finishes, so the charts and the 3 generated emails are saved in it.

## API Key Instructions

I use the free Google Gemini API.

1. Go to https://aistudio.google.com/apikey and sign in with a Google account.
2. Click "Create API key" and copy the key.
3. Set it as an environment variable named `GEMINI_API_KEY` before starting Jupyter.

   Windows (Command Prompt):

   ```
   setx GEMINI_API_KEY "paste-your-key-here"
   ```

   Close and reopen the terminal after using `setx`.

   Mac / Linux:

   ```
   export GEMINI_API_KEY="paste-your-key-here"
   ```

4. The notebook reads the key with `os.getenv("GEMINI_API_KEY")`. If the variable is not set, it asks for the key when you run the cell. The key is never written in the code, so do not paste a real key into the notebook or upload it to GitHub.

The free tier has rate limits, which is fine for 3 emails. If a model name is no longer available, change the names in the `MODEL_NAMES` list in the notebook.

## Optional: Test Email Sending (Additional Upgrades)

This is off by default (`SEND_TEST_EMAILS = False`). To try it with a Gmail account I control:

1. Turn on 2-step verification in the Google account and create an App Password.
2. Set these environment variables: `EMAIL_ADDRESS` (my Gmail), `EMAIL_APP_PASSWORD` (the app password) and `TEST_EMAIL_ADDRESS` (an address I own, it can be the same Gmail).
3. In the notebook, put the approved Review IDs in `approved_ids`, run the email log cell again, then set `SEND_TEST_EMAILS = True` and run the send cell.

## Notes

- The keyword categories are simple and a review can be put in a slightly wrong category.
- The priority score is a simple rule I designed for this project.
- Location results are based on small numbers of reviews per state.
