### Mining and exploring care pathways from electronic medical records with visual analytics

Perer, Adam, Fei Wang, and Jianying Hu. "Mining and exploring care pathways from electronic medical records with visual analytics." *Journal of biomedical informatics* 56 (2015): 369-378.

Aim : Mine and visualize a set of frequence event sequences(diagnoses, medications) from patient EMR data.
The goal is to utilize historical EMR data to extract common sequences of medical events such as diagnoses and treatments, and investigate how these sequences correlated with patient outcome.

Importance : Clinicians are not only interested in the temporal patterns, but also in the correlations between such patterns and the patients’ outcomes. **Most existing pattern mining techniques lack the capability to elucidate such correlations.**

Method :

1. Event Database : 

   - patient electronic medical records with multiple event types, patient outcomes, and the event hierarchy. 

   - RDBMS

2. Data Preprocessor 

   - Python

3. Frequent Pattern Analytics : 

   - In EMRs, duration information is often not captured. Most EMR data contains point events => Mine patterns from sequences of point events. 
   - Pattern explosion due to simultaneous records (in outpatient records, the finest time resolution is a day) => Detect frequent clinical event pakages (CEPs) and trace as a transaction.
   - Utilize and Extend Sequential PAttern Mining with bitmap representation (SPAM)

   - Python

4. Visual Interface 

   - D3
   - Sankey : Diagnosis -> Medication, Lab -> Diagnosis -> Medication, Lab -> Diagnosis -> Lab

Case : Analyze the diagnoses and treatments of a cohort of hyperlipidemic patients with hypertension and diabetes pre-conditions