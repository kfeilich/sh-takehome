# sh-takehome

Author: Kara Feilich  
Date Created: 2025MAR13  
Date Last Updated: 2025MAR14   
<br> 
**Description**:  
This repo merges two tables provided by Serif Health describing Payer and Provider negotiated rates into a shared schema, identifies discrepencies, and where possible, resolves those discrepencies. 

**Requirements**: 

* Python (v.3.11.7)
	* pandas (v.2.1.4)
 	* seaborn (v.0.12.2) 
* Jupyter Notebook (v.7.0.8)

**Installation/Running**:

1. Install [Python](https://www.python.org/downloads/) and install [Jupyter Notebook](https://jupyter.org/install). I recommend doing all of this with an Anaconda distribution instead of the individual links: [Anaconda Install](https://www.anaconda.com/download).
2. Install pandas and seaborn. (If you used an Anaconda distribution, skip this step).
3. Download or clone this repo to your machine. 
4. Run jupyter notebook (either from the console or from Anaconda Navigator), navigate to serif_ds_takehome.ipynb within the jupyter notebook interface, and open the notebook. 
5. Code can be executed within the browser-based Jupyter notebook interface. 
_


### 0. README Contents:
1. Repo Contents
2. Data Description
3. General Approach
4. Challenges
5. Assumptions and Justifications
6. Suggested Improvements, Scaling, and Next Steps


### 1. Repo Contents:

* data/
	* hpt_extract_20250213.csv
	* tic_extract_20250213.csv
* output/
	* test_merge2_wdelta.csv
* serif_ds_takehome.ipynb

### 2. Data Description
For this task, I was provided with two source tables with the following field values (and my assumed interpretations of them where not obvious): 

 * **hpt_extract_20250213.csv**
 	* source_file_name (str)
 	* hospital_id	 (str): unique identifier for each facility
 	* hospital_name (str): facility name
 	* last_updated_on (date)
 	* hospital_state (str)
 	* license_number (str)
 	* payer_name (str)
 	* plan_name (str)
 	* code_type (str): kind of procedural code (e.g., CPT)
 	* raw_code (int): code value
 	* description (str): procedure/visit description
 	* setting (str): outpatient, inpatient, or both
 	* modifiers (str)
 	* standard_charge_gross (double)
 	* standard_charge_discounted_cash (double)
 	* standard_charge_negotiated_dollar (double)
 	* standard_charge_negotiated_percentage (double)
 	* standard_charge_min (double)
 	* standard_charge_max (double)
 	* standard_charge_methodology (str): some categorical descriptors (e.g., fee schedule, case rate)
 	* additional_payer_notes (str)
 	* additional_generic_notes (str)
 	
* **tic_extract_20250213.csv**
	* payer (str)
	* network_name (str): the specific plan network (primary key)
	* network_id (str) unique identifier (primary key)
	* network_year_month (int)
	* network_region	 (str): geographic region of service
	* code (int)
	* code_type (str)
	* ein (int): federal tax ID?
	* taxonomy_filtered_npi_list (array): provider list?
	* modifier_list (str or array)
	* billing_class (str): institutional or professional
	* place_of_service_list	 (array)
	* negotiation_type (str): negotiated, percentage, or fee schedule
	* arrangement	rate (str): guessing "ffs" is "fee for service"
	* cms_baseline_schedule	 (str): governmental comparison code
	* cms_baseline_rate (double): governmental baseline for that procedure


Based on these fields, it looks like I could potentially join on:

* payer_name = payer_name,
* plan_name = network_name (or a combination of network name and payer) _(Ideally, would like to join on network\_id)_
	* From post-mortem: This ended up not being the case. 
* code_type = code_type
* raw_code = code

I will attempt to keep all of the other fields since I don't know what will be important yet.

### 3. General Approach
These source tables are not large, so for this, I can use pandas to do the merge. For scaling, would probably use a distributed approach using SQL and/or PySpark. I have already been told there are discrepencies among the tables, and I'm not 100% sure where they might be--but I know there should be rows for each hospital/payer-plan/code. Inconsistencies in data formatting can be addressed at that point, the merge re-run, and then I can calculate the relative delta for matching rows. Subsequent analysis of the unmatched rows will hopefully inform me about challenges for missing values and possible approaches for imputation.
* Post-mortem: Subsequent analysis was not conducted because the matching was not successful. Reasons discussed below.

### 4. Challenges
* Inconsistent field codings. Payer names were inconsistent in name and capitalization/whitespace. The same codes were encoded differently across facilities. 

* I'm not convinced there were enough joinable keys in these two tables to even begin to be certain of correct matches. In theory, for each facility/plan-accepted/procedure combination, there should be a single negotiated rate. I could NOT match on facility here, so I had to match on payer, code type, and raw code--- which results in MULTIPLE rows per combination. (More evidence-there are only 1104 unique combinations of hospital charge and rate here). There could be many reasons for this-- If a hospital owns an outpatient clinic vs a hospital, they may have different staff performing the same procedure, etc. This would have been remedied by a clear facility-id in the payer table, or a clear plan-id in the hospital table. Something is missing. This is reinfoirced by the phrasing of your prompt: "The payer extract has the same three billing codes, extracted from three of the largest (Cigna / Aetna / UHC) commercial payers' national PPO files, for the relevant hospitals"--how can I tell that these payer extracts were for these hospitals? There did not appear to be a way of matching these. 

* Because I could not be confident of the matches, and the deltas are all over the place, I really don't have a great idea of how I would interpolate missing values. One thing I am wondering is, given that there are so few actual unique combinations of "rate" and what I got as "hospital charge", it seems the big deltas would be common EVEN if I got the matches right. In hindsight, it seems that the magnitude of delta relative to the size of the rate is very important. I find myself wondering what, exactly, the payer "rate" is. Because if you have a "rate" of 46.39, and a standard_charge_negotiated_dollar that is 50 times larger (of which there are many), I wonder if that rate is multiplied by something to reach the hospital charge. This would be useful information for appropriate metadata for these tables.

* Because I spent my entire available time feeling like I was missing important data and testing everything to try and see how I could make up for it, my code in this notebook is understandably sloppy and inefficient. Normally when I'm trying to understand a dataset, I try to brute force things and clean up once I know how it all works. Didn't have the time.

### 5. Assumptions and Justifications
Assumption 1: I am not messing with any of the data values (e.g. making any artificially equivalent) by putting them all in lowercase, and replacing any white space with underscores.  
Justification 1: I already see there are formatting issues, and this will at least standardize them a little.

Assumption 2: There are no payers other than the three in the payers datset represented in the hospital data set that are actually owned by one of the three payers in the payer dataset.  
Justification 2: I don't have time to do a market scan of payers.

### 6. Suggested Improvements, Scaling, and Next Steps
 * The approach used here (jupyter notebook and pandas-based) is not going to work for very large datasets--but it is suitable for identifying patterns and challenges from small data samples, and planning for scaled implementation. If the datasets do become large enough to justifty distributed computing, it would be worth investing in a cloud distributed computing pipeline, ideally with tools for standardized data processing pipelines and analysis. (I've used Palantir and MS Azure, but I'm sure there are others). 
 * Ideally, I would like to see standardized primary and foreign keys in the schema for these source tables, so I could merge on other features. Like hospital_id.
 * _Imputation_: With this limited a datset, it is hard to propose the best means of imputation for missing values if there is no legitimate match. Some options I might consider depending on the availability of the data:
 	* If there are MANY matches, is it possible to determine a modal match? (simplest option), or look at claims data to see what is actually being covered/charged. 
 	* I suspect there is a geographic cost of living component to the rates agreed upon by payers and providers--NY probably charges more than AK. Again, if there are enough possible values, you may be able to use clustering or distributional information to make a best guess for a given hospital-plan-procedure/visit relationship.
 	* If you have historical data that includes some accurate matches, you may be able to extrapolate from that to estimate a current rate.
* _Beyond the current use-case_: There are a number of future directions I could see Serif going in, depending on the healthcare payer-provider ecosystem (and surrounding regulations). 
	* From providing existing data to providing suggested solutions: If you have enough data, you could also, probably, determine a "best value" rate estimate for a given hospital, giving them something to target in their negotiations. Similarly, you could provide data to plans/networks to give a better idea of the landscape. (Note: both sides are probably already doing it to some extent, but having a third party independently doing so may be both a convenience and a negotiation tool.)
	* Navigating changing regulatory environments: In an extreme case, if Medicare and Medicaid are dissolved, it may be prudent to simulate the effects of such a shock to overall rates (almost a consulting/research role). This would likely involve a healthcare market econometrician. In another extreme (longer term, possibly) case, if a public option, or even a Medicare/Medicaid expansion were available, there might also be a market for predictive or exploratory analytics.
	* Independent cost-efficiency research: This is probably MAJOR scope creep, but, if there are MAJOR discrepencies/residuals in rates outside of those due to geography for a given procedure, it may make sense to partner with healthcare providers and plans to determine what, if anything, is driving those outliers. Is it pure profit margin? Is there a questionable supply chain issue? Are providers underutilized? There are a bunch of ways to estimate/calculate healthcare costs (I've got a little experience with time-driven activity based costing). I imagine both payers and facilities would like to drive down rates. (Providers don't love this kind of effort... but that's why more work is always required to research value-based care and other appropaches to fund healthcare.) Other countries are **much** better at this than we are.

