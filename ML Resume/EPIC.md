1. RDL:
	- Datalake for Radiologists to download DICOM images to train lesion detection algorithms.
	- DICOM images were filtered through enriched data from Clarity instance.

2. Claims Appeals pipeline:
	- review the denied claims.
	1. **Use case 1: Denial reason classification:**
		- multi class classification
		- model predicts which category of denial it is — prior auth missing, coding error, eligibility issue, duplicate claim, etc.
		- XGBOOST + fine-tuned BERT for case notes
		- **Data**: plan details, procedure details, prior auth
	 2. **Use case 2: Appeal success prediction** 
		 - binary classification problem — will this appeal be **overturned** (win) or **upheld** (lose)
		 - **Data**: clinical history, payer profile