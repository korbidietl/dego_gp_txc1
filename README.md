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

### Notebook 01: Data Quality

The `01-data-quality.ipynb` notebook represents the core Data Engineering contribution to the credit application governance project. Its purpose is to transform the raw nested JSON dataset (`rawcreditapplications.json`) into a structured, analytically reliable dataset suitable for fairness and governance evaluation. The pipeline addresses four core data quality dimensions—validity, consistency, completeness, and accuracy—and documents each transformation with quantifiable pre- and post-cleaning metrics.

The raw dataset contains 502 loan application records with nested applicant and financial attributes. Through a series of validation, transformation, and filtering steps, the notebook produces a cleaned dataset of 496 records across 34 structured columns, exported as `clean_credit_applications.csv`. Each stage of the pipeline measures the magnitude of detected issues and documents the remediation applied, ensuring transparency and reproducibility.

**Validity Checks**

Several financial variables in the dataset contained values that violated logical or domain constraints. To detect these issues, numeric columns were coerced using `pd.to_numeric(errors='coerce')`, which converted invalid entries to NaN. Additional bounds checks implemented in `check_numerical_for_limits()` were applied to enforce realistic value ranges.

This process identified a number of validity violations, including:
* approximately 0.2% infinite values in `financials.savings_balance`
* 0.4% implausibly large values in `financials.credit_history_months` 
* 0.2% `financials.debt_to_income` ratios exceeding 1

Values falling outside predefined logical bounds—such as negative financial values or debt-to-income ratios greater than one—were replaced with NaN. This step ensured that financial variables used in later modeling stages represent plausible values and conform to expected numeric domains.

**Consistency Standardization**

Several attributes exhibited inconsistent formatting across records and required normalization before analysis.

Dates stored in applicant_info.date_of_birth appeared in multiple formats (for example YYYY-MM-DD, DD/MM/YYYY, and MM/DD/YYYY). A multi-stage parsing procedure standardized these formats into a unified datetime representation, enabling reliable age calculation.

Contact and identity fields were similarly standardized:
* `applicant_info.email` values were converted to lowercase and stripped of whitespace, and entries failing a basic domain structure check were invalidated (approximately 2.0% of records). 
* `applicant_info.ssn` values were validated against the regular expression r"\d{3}-\d{2}-\d{4}", identifying 1.0% invalid entries, which were subsequently set to NaN. 
* `applicant_info.gender` values were harmonized into a consistent categorical representation (Male / Female). 
* `applicant_info.zip_code` values longer than five digits were truncated to enforce standard U.S. ZIP code formatting.

The semi-structured behavioral field spending_behavior required additional transformation. This attribute contained nested arrays describing spending events. The notebook explodes these arrays, normalizes the nested dictionaries, and pivots them into 17 structured spending features (e.g., spendHealthcare). This transformation converts semi-structured behavioral data into a consistent tabular representation suitable for statistical analysis.

**Completeness Assessment**

A systematic missing-value analysis was performed using a custom missingsummary() function. This revealed several columns with substantial data sparsity, including:
* 87.5% missing values in `processing_timestamp`
* 58.9% missing values in `decision.rejection_reason` 
* 41.1% missing values in both `decision.approved_amount` and `decision.interest_rate`

Columns with more than 50% missing observations were removed from the dataset, as their informational value was insufficient for reliable analysis. Additional improvements to completeness were achieved by consolidating redundant financial attributes: missing values in financials.annual_income were filled using `financials.annual_salary` before the salary column itself was removed. Following this consolidation, the final dataset reports 0% missing values for the primary income variable.

**Accuracy and Duplicate Detection**

Finally, the notebook verifies record-level accuracy by identifying duplicate identifiers and potentially duplicated identities. Two duplicate application identifiers were detected and removed, reducing the dataset from 502 to 500 rows. In addition, four duplicate Social Security numbers were detected across multiple records (e.g., applications app016 and app088 sharing the same SSN). These cases were flagged for further investigation and removed from the cleaned dataset, resulting in the final 496-row dataset.

**Governance and Reproducibility**

The entire pipeline is implemented using iterative transformations on `df.copy(deep=True)` to ensure reproducibility and traceability. Each cleaning step logs pre- and post-transformation metrics (for example “Rows before/after: 502 --> 500 --> 496”), providing a transparent audit trail for governance review.

The resulting dataset, exported as `clean_credit_applications.csv`, forms the foundation for subsequent fairness and bias analysis. By systematically addressing issues of validity, consistency, completeness, and accuracy, the notebook ensures that downstream governance evaluations are performed on a dataset that is both analytically reliable and compliant with data quality best practices.

### Notebook 02: Bias Analysis

The 02-bias-analysis-2.ipynb notebook represents the Data Scientist contribution to the project and fulfills the assignment’s Bias Detection and Fairness objective. Building on the cleaned dataset (`clean_credit_applications.csv`), the notebook evaluates potential discriminatory outcomes in the credit approval process using statistical tests, visualization, and regulatory benchmarks. The analysis assesses fairness against the Equal Credit Opportunity Act (ECOA) four-fifths rule (requiring a disparate impact ratio >= 0.8) and considers governance implications under the EU AI Act, which classifies automated credit decision systems as high-risk AI systems requiring transparency and bias monitoring.

**Gender Disparate Impact**

The primary fairness test evaluates approval disparities between male and female applicants. The calculated disparate impact ratio (DIR) is 0.767, meaning the female approval rate is approximately 76.7% of the male approval rate. Specifically, female applicants are approved at a rate of 51.0% (249 records) compared to 66.5% for male applicants (245 records). A chi-square test confirms that the difference is statistically significant (x^2 p = 0.001 < 0.05). Because the observed DIR falls below the 0.8 threshold defined by the four-fifths rule, the results indicate a potential case of disparate impact. These differences are illustrated through approval-rate bar charts and contingency tables comparing gender-specific approval outcomes.

**Age-Based Disparities**

The analysis also examines approval rates across age cohorts. The 25–34 age group exhibits the lowest approval rate at 44.6%, while the 45–54 group shows the highest approval rate at 66.3%. A chi-square test (x^2 p = 0.002) confirms that these differences are statistically significant. The pattern suggests that younger applicants may face systematic disadvantages relative to middle-aged applicants, raising concerns that age-related variables could be functioning as implicit risk proxies.

**Proxy Discrimination**

Beyond direct demographic comparisons, the notebook investigates whether non-protected variables act as proxy indicators for protected characteristics. Several patterns emerge:
* ZIP code clusters reveal different approval outcomes across neighborhoods. Applicants from male-heavy ZIP code areas exhibit approval rates of 64.5%, compared to 52.2% in more gender-balanced neighborhoods, with the relationship confirmed by a chi-square test (x^2 p = 0.007). 
* Debt-to-income (DTI) bands show a persistent gender gap, with male applicants enjoying a 15–16% approval advantage within comparable DTI categories (x^2 p < 0.001). 
* Income distributions correlate with gender differences, suggesting that income thresholds may amplify existing demographic disparities.

Together, these findings indicate that seemingly neutral variables such as ZIP code, income, and DTI may indirectly encode demographic patterns and produce proxy discrimination.

**Interaction Effects**

The notebook further examines interaction effects between demographic variables. In particular, the age to gender interaction reveals compounded disparities. For example, female applicants in the 35–44 age group exhibit lower approval rates than both younger women and male applicants of the same age group. A chi-square interaction test (x^2 p < 0.001) confirms that these differences are statistically significant. Visualizations such as heatmaps and stacked bar charts highlight how overlapping demographic characteristics can amplify disparities that might not appear when variables are examined individually.

Overall, the notebook provides a systematic fairness audit of the credit approval model, identifying several areas where model decisions may disproportionately affect certain demographic groups. These findings support governance recommendations such as ongoing disparate impact monitoring, periodic fairness audits, and enhanced documentation of model decision criteria to ensure compliance with regulatory expectations and responsible AI practices.