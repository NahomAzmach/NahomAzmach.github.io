*CougHacks 2025: 24 hours, Top 5 out of 15+ teams. I was backend lead and integration lead.*

Everyone gets phishing emails. The scary ones aren't the obvious Nigerian-prince ones, they're the "unusual login on your account, verify now" ones that look exactly like the real thing. Q-Mail connects to your Gmail and tells you, email by email, how much to trust what you're looking at, and why.

![Your inbox, scored. Each email gets a trust score, a risk level, and a trusted / untrusted domain badge.](assets/img/qmail-list.webp)

## Two layers, on purpose

We didn't want to just throw every email at an LLM and hope. So the analysis is a **hybrid**.

**Layer 1 is plain rules**, fast and explainable. Pull the sender's domain, check it against trusted domains (major providers, `.edu`, `.gov`), and scan the subject and body for patterns phishing loves: urgency ("act now," "account suspension"), account scares ("verify your account," "unusual login"), money hooks ("claim your prize," "unclaimed funds"), and action prompts ("click here," "update payment").

```python
# Determine overall risk level
if not domain:
    risk_level = "Unknown"
elif trusted and not suspicious_patterns:
    risk_level = "Low"
elif trusted and suspicious_patterns:
    risk_level = "Medium"
elif not trusted and not suspicious_patterns:
    risk_level = "Medium"
else:
    risk_level = "High"
```

**Layer 2 is AI.** OpenAI scores the email on a 0–10 trust scale and explains its reasoning in plain language. Then we blend it back with the rules: known-trusted domains get a minimum score floor, subdomains of trusted domains get slightly less benefit of the doubt, and credible TLDs (`.edu`, `.gov`) get a small bonus. The result covers 7+ risk factors: suspicious links, unusual senders, keyword patterns, grammar red flags and more.

![A flagged email: risk level, score, untrusted domain, the exact suspicious patterns found, and what to do about it.](assets/img/qmail-detail.webp)

## Privacy as a toggle

Reading someone's email with AI is a big ask. So AI is optional, and there are levels. **Header-only analysis** sends just the sender, subject and metadata, and the email body stays hidden. **Full analysis** reads the body too, if you choose to turn it on. You can revoke it any time.

![Header-only mode: AI explains the verdict without ever seeing the email body.](assets/img/qmail-headers.webp)

## 12 hours of integrations

My part of the 24 hours was mostly making things talk to each other: **Gmail API** (with Google OAuth), **OpenAI**, and **IPQS** for URL and sender reputation, all integrated in about 12 hours. The rest was the hackathon stuff nobody puts on slides: chasing auth errors in the middle of the night, fixing integration bugs as they came up, helping four teammates get unblocked, and keeping us on the clock to actually ship a working demo.

The architecture call that I think paid off most: rules first, AI second. When the AI was slow or the API hiccuped, the rules still gave you an answer. And when the AI was right, the rules made it *explainable*.

Next on the list: smarter unsubscribe automation, an AI agent that handles the unsubscribe flows for you without keeping your data around.

**Stack:** Python, Flask, Gmail API, Google OAuth, OpenAI API, IPQS, Flask-SQLAlchemy, Jinja templates.
