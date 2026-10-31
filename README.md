# Career Transition Log

PMO → Data Analyst / Analytics Engineer, documenting the process in public.

**Timeline:** Jul 2026 – Jan 2027 (26 weeks)

## Progress
- [X] Week 1–4: SQL + Statistics (July 9th to August 5th)
- [X] Week 5–8: Python (pandas) + first EDA project (August 6th to August 26th)
- [X] Week 9–13: Data storytelling + 統計検定2級 (August 27th yo October 31st)
- [ ] Week 14–26: Job search + portfolio

*Started: July 9th 2026*

## Monthly Review

---

### Week 1–4 

**How I work:** 

Turning vague tasks into concrete next steps

Through this transition I've learned a lot about how I work most effectively —
and built habits around it:

- **I break ambiguous work into a concrete first action.** Open-ended tasks
  (e.g. "study statistics" or "write my own examples") used to feel hard to
  start. I found that the blocker was almost always an unclear next step — so I
  now begin with something small and concrete (e.g. "do practice problem 1")
  rather than trying to plan the whole thing first. Momentum follows.

- **I learn by doing, then fill gaps as they appear.** Instead of working
  through material front-to-back, I start with hands-on problems and look things
  up when I get stuck. This keeps the work concrete and moving.

- **I reflect regularly on my own process** and adjust my approach based on what
  I notice — treating how I work as something to iterate on, the same way I'd
  iterate on an analysis.

---

### Week 5-8

**Mental Health in Tech — Exploratory Analysis**

Exploring the OSMI Mental Health in Tech Survey with pandas: whether seeking
mental-health treatment varies by industry, country, and gender.

**Key point:** 

the data measures *treatment-seeking*, not illness prevalence —
conclusions are framed accordingly.

**Tools:** 

Python, pandas, seaborn. 

**Skills:** 

data cleaning (50+ raw gender
values → standardized), groupby aggregation, boolean-to-percentage, visualization.

### Week 9–13

## 📊 Project: Testing the Lipstick Effect — Economic Cycles and Cinema Demand

**Overview**
An analysis testing whether the "lipstick effect" — the idea that people spend
more on small affordable pleasures during downturns — applies to cinema demand
in Japan. Box-office statistics from the Motion Picture Producers Association of
Japan (1955–2025) were combined with economic indicators including real GDP and
the Consumer Confidence Index.

**Findings**
The hypothesis was not supported. While raw values showed a positive correlation
between the economy and cinema demand, that correlation disappeared once tested
on year-over-year changes — revealing it to be a spurious correlation driven by a
shared time trend. Instead, cinema demand appears to be shaped far more by
structural factors, such as the spread of television and streaming services, than
by economic cycles.

**Tech Stack**
`Python` `pandas` `matplotlib` `statsmodels`

**Methods**
- Joining and cleaning multiple data sources (reconciling different periods,
  frequencies, and base years)
- Time-series visualization (dual-axis charts)
- Simple linear regression (OLS)
- Multi-angle testing: raw values, growth rates, and lagged effects
- Checking for spurious correlation driven by time trends
- Robustness check via outlier removal

🔗 [View the analysis notebook](./path-to-notebook)

---

## 🎓 Certifications

- **Statistics Test Level 2 (統計検定2級)** — Japan Statistical Society, Oct 2026
  Covers descriptive and inferential statistics, probability distributions,
  hypothesis testing, and regression analysis.

---
