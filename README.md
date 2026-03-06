# Data Ecosystems and Governance in Organisations Group Project
### TXC Group 1

### Team Members
* Magdalena Krzkhovska (Data Engineer)
* Océane Bort-Aponte (Data Scientist)
* Riccardo Borella (Governance Officer)
* Korbinian Dietl (Product Lead)




* Document what you found and your methodology. 
* Document individual contributions clearly in the README. 
* executive summary of findings 
* key metrics 
* actionable governance recommendations
* __The README.md is your primary written deliverable. It should include an executive summary of findings, key metrics, and actionable governance recommendations. Think of it as the document your CTO would read.__

### Executive Summary

This project analyzes NovaCred’s credit application dataset to identify data quality issues, detect potential algorithmic bias, and propose governance measures to ensure compliant and responsible use of automated lending decisions.

First, we performed a data quality audit of the raw JSON dataset containing over 500 credit applications. Several issues were identified, including: missing values (combining salary and income), inconsistent data formats (gender and dates), duplicated or incomplete records (dropping duplicate entries with the least incomplete values), and invalid numeric entries (income). These issues were quantified and addressed through cleaning steps such as type normalization, removal of problematic records, and creation of a final dataset, which is ready to use for further analysis.

Second, we evaluated potential discrimination in credit decisions using several fairness metrics and statistical tests. The gender disparate impact ratio fell below the regulatory four-fifths threshold (0.8), indicating potential disparate impact against female applicants. A chi-square test confirmed that approval outcomes differ significantly by gender (p < 0.01). Age-based analysis revealed statistically significant differences across age groups (p < 0.05), with the 25–34 age group falling below the regulatory disparity threshold compared with the highest-approval group. Geographic analysis further indicated that applicants from male-dominated neighborhoods in New York City exhibit higher approval rates, with a chi-square test confirming the association between neighborhood type and approval outcomes (p < 0.001). Additional tests suggest that several financial variables may act as proxy variables for protected characteristics: income thresholds disproportionately exclude female applicants and violate the 80% rule (p < 0.01), while stratified analysis of debt-to-income ratios shows that women are approved less often than men at comparable levels (p < 0.01). Together, these findings suggest that the model may indirectly encode demographic disparities through correlated variables such as income, ZIP code, and debt-to-income ratios, highlighting the need for continuous fairness monitoring and governance oversight.

Third, we examined privacy and governance risks in the dataset. Several fields contain personally identifiable information, including names, emails, SSNs, IP addresses, and dates of birth. From a governance perspective, these data elements require strong protection mechanisms such as pseudonymization, which is demonstrated within the notebook, __access controls, and clear data retention policies to comply with GDPR principles.__

**Overall, the analysis highlights that both data quality issues and structural bias patterns can significantly affect automated lending systems. To mitigate these risks, we recommend implementing continuous fairness monitoring, stronger documentation of model decisions, and governance controls such as audit trails, human oversight in decision processes, and stricter management of sensitive personal data.**