### Comparative Analysis of Sepsis Identification Methods in an Electronic Database

Johnson, Alistair EW, et al. "A Comparative Analysis of Sepsis Identification Methods in an Electronic Database." *Critical care medicine* 46.4 (2018): 494-499.

Aim : Compare five different algorithms for retrospectively identifying spesis, including the Sepsis-3 criteria.
Conclusion : The new organ dyfunction-based Sepsis-3 criteria have been proposed as a clinical method for identifying sepsis. There criteria identified a larger, less severely ill cohort than that identified by previously used administrative definitions.

Dataset : 

1. We focused on ICU admissions from years 2008 to 2012 for three reasons: antibiotic prescriptions are only recorded from 2003 onward; explicit sepsis codes were introduced at BIDMC in 2004; the group of admissions between 2008 and 2012 are easily identifiable in the database.
2. Excluded secondary admissions to avoid repeated measures and cardiothoracic surgical service patients since their postoperative physiologic derangements do not translate to the same mortality risk as other ICU patients.
3. **Excluded patients suspected of infection more than 24 hours before and after ICU admission as MIMIC-III only contains ICU data.**

We compared the population identified by the Sepsis-3 criteria with other populations identified by three methods: visually, using Cronbach's alpha, and via their relationship to the primary and secondary outcomes.

- Hospital mortality was higher for patients with SOFA greater than or equal to 2 (13.2%) than those with SOFA less than 2(3.6%).
- cohort size : Sepsis-3 > CDC > Angus > Martin > CMS
- Sepsis-3 criteria primarily rely on the assembly of contributory data elements based on physiology, via the SOFA score, and clinical practice, via the definition of suspicion of infection.

Possily imply that the more restrictive cohorts represent sicker, higher risk segments of the population.

Sepsis-3

- We posit that Sepsis-3 identified a larger and likely less 'pure' cohort of septic patients, but one that still remains at higher risk of mortality (14.5% vs 7.3%) and higher risk of composite mortality/excess LOS(50.0% vs 21.9%).
- Sepsis-3 criterion provides the temporal context.
  The algorithm for sepsis-3 requires the delineation of a time point at which the patient may be septic(suspected of infection with associated organ failure).
- Sepsis-3 aligns with the contemporary understanding of the pathophysiology of sepsis.
- (단점) Both the sepsis-3 and CDC criteria rely on treatments as surrogate for organ failure. For Sepsis-3, the retrospective definition of suspicion of infection is entirely dependent the actions of the clinician.