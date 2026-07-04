Unsupervised Learning Report - DS23, Module 3, Assignment 2

Name: Orarr2
Date: 4.7.2026
Chosen task: B - Credit Card Behavioral Segmentation (Clustering)


1. Framing the Problem

   1.1 The business question

      The bank's credit-policy team wants to know which customer segments
      require differentiated risk action - credit-limit increase,
      credit-limit decrease, "watch-list" flagging, or a proactive outreach
      from the retention unit. I chose to frame the task as behavioral
      segmentation that maps onto risk profiles, so that the policy team
      can act differentially on each segment.

      The segmentation is descriptive, not predictive: no "default" or
      "delinquency" outcome labels are available. Every risk claim in this
      report is an inference from behavior, not a validation against real
      outcomes. That limit is part of the answer, not a weakness to hide.

   1.2 Distance metric

      The primary distance is Euclidean. In a risk story, magnitude
      matters: a customer with a 10,000-dollar balance is a completely
      different exposure from one with a 100-dollar balance, even if
      their behavioral pattern looks similar. Cosine would treat the two
      customers as identical because it ignores magnitude, so it was
      rejected early.

      I planned a sanity check with Manhattan Hierarchical as a secondary
      metric. In practice all three linkages (single, average, complete)
      collapsed to a mega-cluster, with a best ARI of 0.168 against
      KMeans; "single-linkage" put 8,946 of 8,950 customers into a single
      cluster. I pivoted to Ward + Euclidean as the sensitivity check and
      report the Manhattan failure as a finding in its own right.

   1.3 Feature selection and engineering

      I started with 17 numeric features (after dropping "CUST_ID"). I
      dropped "TENURE" (84.7 percent of customers have TENURE = 12, a
      near-constant feature) and "PURCHASES" (correlation 0.92 with
      "ONEOFF_PURCHASES", a near-linear sum of "ONEOFF" plus
      "INSTALLMENTS", which double-weights the spending axis under
      Euclidean distance).

      I added three risk-relevant engineered features: "utilization_ratio"
      (BALANCE divided by CREDIT_LIMIT), "payment_to_min_ratio" (PAYMENTS
      divided by MIN_PAY plus 1), and "cash_advance_share" (CASH_ADVANCE
      divided by CASH_ADVANCE plus PURCHASES plus 1). The final feature
      space: 18 features.

   1.4 Handling missing values

      "MINIMUM_PAYMENTS" has 313 missing rows. Investigation revealed
      these are not random NaN, but a specific profile: median BALANCE =
      16.85, the 75th percentile of PAYMENTS = 0, and all 313 have
      "PRC_FULL_PAYMENT" = 0. These are dormant customers.

      Median imputation (about 312) would have implanted them in the
      middle of the distribution and distorted the segmentation. I chose
      to fill with zero, which reflects reality: no balance to charge a
      minimum payment on.

      "CREDIT_LIMIT" has one missing row, filled with the median.

      8 customers with "CASH_ADVANCE_FREQUENCY" above 1 (up to 1.5),
      beyond the documented range of [0, 1]. Kept as is: this is a real
      risk signal from the heaviest users on the cash-advance axis.


2. Results and Validation Table

   The result set reflects the metrics on the primary clustering (KMeans,
   k = 5), including algorithmic sensitivity checks. The list is ordered
   from the overall metric to the per-check sensitivity results.

   2.1 Algorithms tried

      Primary algorithm: KMeans (Euclidean), n_init = 10.
      Secondary algorithm: Ward Hierarchical (Euclidean).
      Failed and reported: Manhattan Hierarchical (all linkages grew a
      mega-cluster).

   2.2 Choosing k

      Both Silhouette and Elbow pointed to k = 3. I chose k = 5 on
      business grounds, acknowledging the metric cost. Full reasoning in
      question 3.2.

   2.3 Silhouette

      At k = 5: 0.237. At k = 3 (metric-optimal): 0.417. This gap is a
      business cost decided in advance, not a surprise.

   2.4 Cluster sizes

      1,152, 1,237, 1,401, 2,332, and 2,828 customers. The smallest
      cluster is 13 percent of the population, so no cluster is a
      "trivial cluster" of isolated outliers.

   2.5 Seed stability

      Mean ARI = 0.992 over 5 seeds, minimum 0.987. The algorithm is
      almost fully deterministic on this data after scaling.

   2.6 Sub-sample stability

      Mean ARI = 0.982 over 5 random 80-percent sub-samples, minimum
      0.968. Cluster assignment for each customer is stable under data
      perturbation.

   2.7 PCA-5 sensitivity

      ARI = 0.954 between clustering in 18D and clustering in PCA-5
      (which preserves 87.5 percent of variance). The structure survives
      dimensionality reduction.

   2.8 Ward sensitivity

      ARI = 0.548 between KMeans and Ward at k = 5 (moderate agreement).
      At k = 3 the same check gives ARI = 0.833 (stronger agreement),
      consistent with the interpretation that k = 5 is a business-driven
      split of a real k = 3 structure.

   2.9 Manhattan failure

      Failure reported. Best ARI = 0.168, under "linkage complete". Under
      "linkage single", 8,946 of 8,950 customers remain in a single
      cluster. The structure is geometry-dependent.


3. Guiding Questions

   3.1 No ground truth

      In the absence of labels, I argued for the quality of the
      clustering through four indirect channels: a non-trivial Silhouette
      (0.237, moderate but above random), balanced cluster sizes
      (smallest at 13 percent, so no "pseudo cluster"), stability under
      both seed and sub-sample perturbations (ARI above 0.98), and a
      sensitivity check via PCA-5 dimensionality reduction (ARI = 0.954).

      The evidence is weak for two reasons: every check uses the same
      Euclidean geometry, so any algorithm-level bias repeats across all
      checks (and the Manhattan failure is a strong hint of this); and
      there are no outcome labels (default, delinquency, churn) to
      validate the risk inferences I draw for each segment.

   3.2 Choosing k

      Elbow and Silhouette do not disagree with each other: both pointed
      to k = 3. The Silhouette score peaked at 0.417, and the drop in
      inertia from k = 2 to k = 3 (minus 29,000) was much larger than
      every subsequent drop.

      The real disagreement was between the metrics and the business
      question: at k = 3 the data breaks into Dormant (13 percent),
      Transactors (15 percent), and a mega-cluster of 72 percent that
      mixes Cash advance heavy, Heavy spenders, and Light revolvers -
      three completely different risk profiles with no effective policy
      lever between them.

      I believed the business framing over the metrics and chose k = 5,
      acknowledging the Silhouette cost (0.417 dropping to 0.237). The
      lesson: metric-optimal clustering and business-useful clustering
      can genuinely differ, because a Silhouette score has no business
      unit of measure.

   3.3 Scaling

      Scaling was the most important decision after framing. The amount
      features ("BALANCE", "PAYMENTS", "CASH_ADVANCE", "MINIMUM_PAYMENTS")
      show skew between 5.2 and 13.6, and the engineered
      "payment_to_min_ratio" reaches a skew of 60.5.

      Under StandardScaler alone, the max z-score of "payment_to_min_ratio"
      = 77, meaning one customer completely dominates the feature. The
      clustering result was a mega-cluster of 3,912 (44 percent) plus a
      tiny cluster of 85 customers.

      After log1p and then RobustScaler, the values dropped to a max z
      of 10, and cluster sizes rebalanced (1,152 to 2,828, all between
      13 and 32 percent). Silhouette also rose from 0.206 to 0.237 even
      though k did not change.

   3.4 Stability

      Across 5 KMeans seeds, the mean pairwise ARI was 0.992 (minimum
      0.987): the algorithm is almost fully deterministic on this data
      after scaling.

      Across 5 random 80-percent sub-samples, the mean ARI against the
      primary clustering = 0.982 (minimum 0.968): cluster assignments
      per customer are stable under data perturbation.

      I would rely on this with next month's data if (a) the
      distributions of the behavioral features stay stable, and (b) we
      re-fit rather than re-use old centroids. I would not rely on this
      under a regime shift (for example after a recession that shifts
      the utilization distribution), because the scaler was fit to this
      window, and the cluster boundaries reflect the scale of the
      current window.

   3.5 What defines each cluster

      A Kruskal-Wallis test with eta-squared as effect size ranks the
      discriminators: "cash_advance_share" (eta squared = 0.643),
      "PRC_FULL_PAYMENT" (about 0.45), "utilization_ratio" (about 0.38),
      and "BALANCE_FREQUENCY" (about 0.36) are the strongest.

      The per-cluster profile medians map cleanly onto a behavioral
      story: Dormant (BAL = 25, util = 0.01), Transactors (PRC_FULL =
      0.88, payment_to_min = 7.1), Light revolvers (util = 0.26, no full
      payment), Heavy spenders (BAL = 1,906, PURCH = 2,075), Cash
      advance heavy (cash_advance_share = 1.0, util = 0.69).

      These five personas are familiar in the credit-card industry (the
      "transactor vs revolver" dichotomy is textbook), which is mild
      evidence that the structure is real and not invented.

   3.6 Real or artifact

      KMeans assumes compact, roughly spherical clusters. I therefore
      checked whether the structure survives methods that do not share
      that assumption.

      Seed and sub-sample stability (ARI above 0.98) rule out an
      initialization coincidence. Clustering in PCA-5 agrees with the
      18D result at ARI = 0.954, so it is not high-dimensional noise.

      Ward Hierarchical - a different algorithm, with no centroid or
      sphericity assumption - agrees at ARI = 0.548 at k = 5 and at ARI
      = 0.833 at the metric-optimal k = 3, which is consistent with k =
      5 being a business-driven split of a real k = 3 structure, not a
      KMeans artifact.

      The cleanest artifact finding is the Manhattan failure: under the
      L1 distance the structure collapses into a mega-cluster (best ARI
      = 0.168 against KMeans). The structure is therefore real under
      Euclidean geometry but geometry-dependent, and I report it as
      such.

   3.7 Business action

      One concrete action and one KPI per segment, framed as policy or
      marketing levers, not as default predictions.

      Dormant, minimal exposure - re-engagement campaign with 20 dollars
      cashback. KPI: percent of dormants who purchase within 90 days
      (target 15 percent).

      Transactors, healthy, low risk - premium card upgrade with
      cashback program. KPI: upgrade percent times incremental annual
      revenue (target 8 percent upgrade and 120 dollars extra ARPU).

      Light revolvers, moderate risk - balance transfer at 0-percent APR
      for 12 months. KPI: 6-month retention vs a control group (target
      plus 5 percent).

      Heavy spenders, elevated risk - credit-limit increase combined
      with a linked savings product. KPI: utilization 6 months after the
      offer (target below 0.5) and savings open rate (target 25 percent).

      Cash advance heavy, high risk - offer of a personal loan at a rate
      lower than cash advance. KPI: percent migrating to the loan within
      6 months (target 25 percent).

      If the team cannot name an action for a given cluster, that
      cluster does not justify its existence. All five here pass the
      bar. The Heavy spenders offer is the one I am least certain about:
      their behavior might also be a profile of a high-income
      transactor, and a credit-limit increase for a customer whose
      utilization is about to spike is the exact opposite of correct
      risk policy. This is reported in question 3.8.

   3.8 False-alarm cost

      "Candidates", not "high risk", because no outcome label is
      available to validate the inferred risk level.

      The most expensive false alarm in this segmentation is in the Cash
      advance heavy segment: labeling a customer as high risk when in
      fact the heavy cash-advance use is a temporary liquidity tool of a
      solvent customer (for example a small business owner bridging cash
      flow gaps). The cost of treating them as high risk is brand damage
      and losing a profitable customer to a competitor.

      I mitigate the risk by framing the segmentation as a starting
      point for review, not a default decision, and by recommending an
      income or employment cross-check before any credit-limit reduction.


4. Structure Card

   4.1 Overview

      Owner: Orarr2.

      Task: unsupervised segmentation, 8,950 credit-card customers.

      Business framing: risk profiles for downstream credit-policy
      decisions.

      Dataset: Kaggle - "arjunbhasin2013/ccdata", 8,950 rows, 17 numeric
      features after dropping CUST_ID.

      Feature space after engineering: 18 features (15 originals plus
      "utilization_ratio", "payment_to_min_ratio", "cash_advance_share").
      I dropped "TENURE" and "PURCHASES", and filled "MINIMUM_PAYMENTS"
      with zero for the 313 dormants.

   4.2 Method and performance

      Primary algorithm: KMeans, Euclidean, n_init = 10.
      Secondary algorithm: Ward Hierarchical (Euclidean).
      Failed and reported: Manhattan Hierarchical.

      Choice of k: Silhouette and Elbow pointed to k = 3. I chose k = 5
      on business grounds (must distinguish Cash advance heavy, Heavy
      spenders, and Light revolvers).

      Cluster sizes at k = 5: [1,152, 1,237, 1,401, 2,332, 2,828].
      Silhouette at k = 5: 0.237.
      Seed stability: mean ARI = 0.992.
      Sub-sample stability: mean ARI = 0.982.
      PCA-5 sensitivity: ARI = 0.954.
      Ward sensitivity: ARI = 0.548.

   4.3 The segments

      Cluster 1, Dormant, minimal exposure (n = 1,152): BALANCE close to
      zero (median 25), utilization 0.01, no payment activity. Risk: low
      (no exposure to the bank, and no revenue either).

      Cluster 4, Transactors, healthy, low risk (n = 1,237):
      PRC_FULL_PAYMENT = 0.88, payment_to_min_ratio = 7.1, low balance
      carry. Risk: low (paying in full each month).

      Cluster 3, Light revolvers, moderate risk (n = 2,332): utilization
      = 0.26, no full payment, mid-range purchases. Risk: moderate
      (revenue source, but exposure exists).

      Cluster 0, Heavy spenders, elevated risk (n = 1,401): BALANCE =
      1,906, PURCHASES = 2,075, utilization = 0.37, no full payment.
      Risk: high (large absolute exposure, ambiguous behavior).

      Cluster 2, Cash advance heavy, high risk (n = 2,828):
      cash_advance_share = 1.0, utilization = 0.69, CASH_ADVANCE =
      1,488. Risk: high (industry signal of financial distress).

   4.4 Real or artifact

      Evidence that the structure is real: seed stability (ARI = 0.99
      over 5 seeds), sub-sample stability (ARI = 0.98 over 5 random
      80-percent sub-samples), PCA-5 sensitivity (ARI = 0.95 against the
      primary 18D clustering), and Ward Hierarchical (ARI = 0.55 at k =
      5, ARI = 0.83 at the metric-optimal k = 3).

      Weaknesses in the evidence: all checks use Euclidean geometry, and
      the Manhattan variants failed (a geometry-dependent failure);
      there is no ground truth (no default/delinquency labels), so the
      risk inferences are not validated; and k = 5 was chosen on
      business rather than metric grounds, and the Silhouette at k = 5
      (0.237) is moderate, not strong.

   4.5 Business action

      Dormant - reactivation campaign, KPI = purchase percent within 90
      days (15 percent).

      Transactors - premium card upgrade, KPI = upgrade percent (8
      percent) and incremental annual revenue.

      Light revolvers - balance transfer at 0-percent APR, KPI = plus 5
      percent retention vs a control.

      Heavy spenders - credit-limit increase plus savings linking, KPI
      = utilization trajectory (below 0.5) and savings open rate (25
      percent).

      Cash advance heavy - personal loan offer at a lower rate, KPI =
      migration percent (25 percent). False-alarm cost: brand damage to
      a profitable customer who uses cash advance as a temporary
      liquidity tool, not as a distress signal.


5. Reflection

   The thing that surprised me most was that the metric-optimal k (k =
   3) was the wrong choice for the business question. Both Silhouette
   and Elbow agreed on k = 3, and yet at k = 3 the segmentation
   collapses 72 percent of customers into a single mixed cluster that
   hides the cash-advance signal completely. The metrics measured how
   clean the partition is, not how useful it is. This is exactly the
   trap the assignment warned about: a Silhouette score has no business
   unit of measure.

   The second surprise was the Manhattan failure. The original plan was
   a clean pattern of "Euclidean primary, Manhattan sanity check". In
   practice every Manhattan linkage grew a mega-cluster. Reporting that
   failure honestly turned out to be a stronger answer to the "real or
   artifact" question than the original plan: the structure shows up
   under one geometry and not under another, and that itself is a real
   property of the data worth knowing.

   Will these segments survive next month's data? Probably yes, if the
   distributional shape stays similar and we re-fit. The strongest
   worry is that the scaler was fit to this specific window, so a shift
   in the utilization distribution (a recession year, a policy change)
   would move the cluster boundaries even if individual customer
   behavior did not change. A production version would re-fit
   periodically and monitor cluster-size drift as an early signal of
   structural change.

   How does this connect to the mid-term project? The most transferable
   artifact is framing discipline: choose the business question first,
   name a candidate action per cluster early on, and only then choose
   the algorithm and the metric. Second in importance is the habit of
   pre-committing to a sensitivity check (in our case: "if the
   structure dies under Manhattan, that is a finding, not a bug to
   fix"). Technically, the pipeline of log1p plus RobustScaler is
   expected to transfer well to most heavy-tailed financial datasets.
