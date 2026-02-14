**SkyGeni Case Study – Sales Win Rate Drop Investigation**

This repo contains my solution to the SkyGeni case study assignment. The business problem is that a B2B SaaS company’s CRO is complaining that win rate has dropped over the last two quarters, even though pipeline volume looks healthy.

This project includes:

* A Jupyter notebook with data cleaning, EDA, insights, custom metrics, and a decision engine  
* A README explaining the thinking, results, and how this could be productized

## **Repo Structure:**

* skygeni\_main.ipynb  
  Main notebook containing:  
  * data cleaning \+ preprocessing  
  * EDA  
  * Insights  
  * 2 custom metrics  
  * decision engine output (driver ranking)

**How to Run:**

Install dependencies:

* Pandas  
* Numpy  
* matplotlib

Open skygeni\_main.ipynb and run top to bottom.

### **1\. Problem Statement**

1. What do you think is the **real business problem** here?  
   This isn’t just an issue of “win rate is lower”, it’s that the win rate is lower and the CRO doesn't know why,  where to focus or  what action to take.  
2. What key questions should an AI system answer for the CRO?  
   If SkyGeni is supposed to help, a system should answer questions like:  
* Did win rate actually drop, or is this just noise?  
* Is pipeline volume really healthy?  
* Is the decline happening everywhere or only in certain segments?  
* Did the sales process slow down (deals taking longer)?  
* What are the top drivers (lead source / industry / reps)?  
* What should the CRO focus on first?  
    
3. What metrics matter most for diagnosing win rate issues?  
   I’d assume win rate by quarter closed, pipeline inflow by created quarter (deals created \+ total created deal amount), deal duration (duration between created and closed), win rate sliced by categories, custom “product metrics” that are not just standard stats\!  
4. What assumptions are you making about the data or business?  
   A few assumptions that my solution relies on is:  
* outcome is correctly recorded as Won or Lost  
* closed\_date is meaningful (represents when the deal actually closed)  
* deal\_amount is a usable proxy for revenue  
* deal\_stage is available but only as a single value (no full stage history)  
* Also: 2024Q3 has very few closed deals, so Q3 is treated cautiously.

### **2\. Key Insights**

1. ### **The sales cycle slowed down massively in Q2:** One of the clearest signals in the data is deal duration. When I computed deal duration as the number of days between created\_date and closed\_date, the company-wide median deal duration jumped from around 63 days in 2024Q1 to around 86 days in 2024Q2. This is a big shift and suggests that deals were stalling more, the sales cycle was slowing down, or more deals were getting “stuck” in late stages. This is important because a CRO can look at pipeline volume and feel reassured that deals exist, but if those deals are taking much longer to progress, fewer will close successfully within the quarter and more will end up lost.

   ### I originally tried looking at win rate by duration buckets (0–30, 31–60, 61–90, 91–120), expecting a clean monotonic trend (longer deals lose more). But it wasn’t very clean, and the bucket approach was hard to interpret.

2. **Inbound is the biggest lead-source driver of the decline:** After establishing that deals slowed down overall, I looked at where the win rate dropped. I focused on exploring the most interpretable breakdowns first: lead source, industry, and sales rep. I also looked at region and deal stage as potential explanations, but those ended up being less helpful as core drivers.  
   Lead source analysis showed:  
* inbound win rate dropped from \~48.8% to \~40.5%  
* inbound is \~26% of Q2 deals  
  So inbound is both large and declining. Referral actually improves in Q2, which is important because it shows the decline is not universal across lead sources.  
3. **Decline is concentrated in specific industries:** FinTech, Ecommerce, and EdTech show meaningful win-rate declines from Q1 to Q2, and each of these industries also represents a large share of Q2 closed deals (roughly \~18–20%). This would imply that these declines are not just small-sample noise. Meanwhile, SaaS and HealthTech are stable or improving.   
   In a real business context, this could reflect industry-specific headwinds, changes in buyer behavior, competitive shifts, or messaging mismatches. While the dataset cannot prove those contextual things, it can clearly show where performance degraded and where the focus should be.

### **3\. Custom Metrics**

* **Deal Stall Rate**: Once the first insight was gathered, I tried to figure out a  product-style metric that captures stalling in a single interpretable number.  
  This led to the first custom metric: Deal Stall Rate. I defined this as the percentage of deals taking more than 90 days to close. The reason I picked 90 days is because it is a practical “one quarter” threshold that is common in B2B SaaS. The point is to detect whether deals are increasingly exceeding a normal cycle time. This metric increased dramatically from around 25% in Q1 to around 43% in Q2. That is a major change and it supports the idea that Q2 performance issues are strongly tied to pipeline slowdown and stalling, not just random win/loss fluctuations.  
* **Category-adjusted expected WR:** I wanted to separate two possible explanations: whether the win rate dropped because the company had more of the “wrong” types of segments, or because conversion performance worsened even within the same categories (focusing on lead sources because that was a big dip). Hence the second metric, category-adjusted expected win rate. The idea is to use the Q1 lead source win rates as baseline performance, and apply them to Q2 lead source shares to compute what Q2 win rate would have been if only the mix changed. This produced an expected Q2 win rate of about 46.7%, while the actual Q2 win rate is about 43.8%. The gap of about −2.9% suggests that the decline is not explained purely by lead-source mix but the performance within leads worsened.  
    
* **Revenue-weighted WR:** Standard win rate treats every deal equally, but a CRO typically cares more about revenue than deal counts. To check whether the Q2 decline also affected revenue outcomes, I computed a revenue (deal\_amount)-weighted win rate, defined as total revenue won divided by total revenue closed (revenue won \+ revenue lost). This dropped from about 48.1% in Q1 to about 44.2% in Q2. This result is cool because it shows the decline is not only visible in deal counts, but also in the revenue value of deals.Q2 did not just lose more wins/deals,  it also lost a larger fraction of money, making the decline more business-relevant.

### **4\. Decision Engine**

**Option B – Win Rate Driver Analysis**

The purpose of the decision engine is not to predict outcomes or prove causality. Its purpose is to prioritize the most important drivers in a way that is both explainable and useful.

The decision engine works by computing win-rate change from Q1 to Q2 for each segment, and combining that with the segment’s share of Q2 deals. This produces an impact score defined as delta win rate multiplied by Q2 deal share. This ranking avoids misleading conclusions where a segment looks dramatic only because it has very few deals. It prioritizes segments that are both large and declining, because those are the segments most likely to materially influence overall company win rate.

The final decision engine output is a ranked list of top drivers across lead source, industry, and rep. 

* The top driver is inbound, which drops around 8.3 percentage points and represents 26% of Q2 deals.   
* The top industry drivers are FinTech, Ecommerce, and EdTech, all of which have both meaningful deal share and meaningful declines.   
* Rep-level drivers also appear (rep\_11 and rep\_2 show large declines), but their impact scores are smaller than inbound and the top industries because each rep represents only about 5–6% of total Q2 volume.

**From a CRO perspective**, this output gives a clear “what to focus on first” story. The highest ROI investigation targets would be inbound conversion decline, FinTech/Ecommerce/EdTech performance decline, and the subset of rep pipelines showing the largest drops, especially when combined with the stalling signal from the deal duration metrics.

### **5\. Mini System Design**

* If SkyGeni were to productize this work, it  would exist as an automated pipeline connected to the customer’s CRM (Salesforce or HubSpot) that runs on a regular schedule and produces a dashboard plus alerts.  
* The simplest version would ingest deal records daily, clean and standardize fields, compute derived features like deal duration and quarter labels, and then compute the same win-rate tables, stall-rate metrics, and driver rankings automatically. The system would run checks daily (because stalling is time-sensitive), and would run driver analysis weekly or at least once per quarter.  
* The output could produce alerts like deal rate or inbound dips. It could also generate a weekly ranked list of drivers like the decision engine output above. In a real product, this would likely be delivered via dashboard cards, notifications, and drill-down links to the underlying deals driving the change.

### **7\. Limitations**

1. Real-world win rate is influenced by many things that are not present in this dataset so the system can confidently detect what changed and where it changed, but it cannot fully prove why it changed in a causal sense. The category-adjusted expected win rate metric is also limited because it only adjusts for lead source; a stronger version would adjust simultaneously for multiple factors like lead source, industry, region, and deal size.  
2. If I had more time, I would:  
* Extend the expected win-rate model into a multivariate model  
* Add confidence intervals so the CRO knows what is real vs noise  
* Build a risk scoring model using deal age, stage, rep history etc,.  
* Create a simple UI/dashboard mock showing:  
  * top drivers  
  * stalled deals  
  * recommended actions  
3. I’m not super confident about the rep-level interpretation because they can be influenced by so many things and it is hard to separate execution from pipeline composition – rep-level outputs are treated mainly as a prioritization signal rather than a definitive performance conclusion.

**8\. Notes**

* One limitation of this dataset is that 2024Q3 contains very few closed deals, so Q3 results are not treated as reliable. Most of the diagnostic comparisons are focused on 2024Q1 vs 2024Q2, which is also aligned with the CRO’s complaint.  
* The dataset lets us measure pipeline inflow (deals created and created deal amount), but it does not include a true pipeline snapshot (how many deals were open at the end of each quarter).