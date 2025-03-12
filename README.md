# sh-takehome

Author: Kara Feilich  
Date Created: 2025MAR13  
Date Last Updated: 2025MAR13  
<br> 
**Description**:  
This repo merges two tables provided by Serif Health describing Payer and Provider negotiated rates into a shared schema, identifies discrepencies, and where possible, resolves those discrepencies. 

**Requirements**: 

* Python (v.3.11.7)
	* pandas (v.2.1.4)
* Jupyter Notebook (v.7.0.8)

**Installation/Running**:

1. Install [Python](https://www.python.org/downloads/) and install [Jupyter Notebook](https://jupyter.org/install). I recommend doing all of this with an Anaconda distribution instead of the individual links: [Anaconda Install](https://www.anaconda.com/download).
2. Install pandas. (If you used an Anaconda distribution, skip this step).
3. Download or clone this repo to your machine. 
4. Run jupyter notebook (either from the console or from Anaconda Navigator), navigate to merge_rates_20250313.ipynb in your cloned repo, and open the notebook. 
5. Code can be executed within the browser-based Jupyter notebook interface. 
_


### 0. README Contents:
1. Repo Contents
2. Data Description
3. General Approach
4. Challenges
5. Assumptions and Justifications
6. Suggested Improvements, Scaling, and Next Steps

-
### 1. Repo Contents:

* data/
	* hpt_extract_20250213.csv
	* tic_extract_20250213.csv
* output/
	* merged_rates_20250313.csv
* merge_rates_20250313.ipynb

<br>
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
* code_type = code_type
* raw_code = code

I will attempt to keep all of the other fields since I don't know what will be important yet. I'll prefix the columns from each table to make it easier for me to parse later.

<br>
### 3. General Approach
These source tables are not large, so for this, I can use pandas to do the merge. For scaling, would probably use a distributed approach using SQL and/or PySpark. I have already been told there are discrepencies among the tables, and I'm not 100% sure where they might be--so an outer join will allow me to look at the unmatched rows. Inconsistencies in data formatting can be addressed at that point, the merge re-run, and then I can calculate the relative delta for matching rows. Subsequent analysis of the unmatched rows will hopefully inform me about challenges for missing values and possible approaches for imputation.

<br>
### 4. Challenges
TBD

<br>
### 5. Assumptions and Justifications
TBD

<br>
### 6. Suggested Improvements, Scaling, and Next Steps
 * The approach used here (jupyter notebook and pandas-based) is not going to work for very large datasets--but it is suitable for identifying patterns and challenges from small data samples, and planning for scaled implementation. If the datasets do become large enough to justifty distributed computing, it would be worth investing in a cloud distributed computing pipeline, ideally with tools for standardized data processing pipelines and analysis. (I've used Palantir and MS Azure, but I'm sure there are others). 
 * Ideally, I would like to see standardized primary and foreign keys in the schema for these source tables, so I could merge on network_id. 
 * _Imputation_: With this limited a datset, it is hard to propose the best means of imputation for missing values if there is no legitimate match. Some options I might consider depending on the availability of the data:
 	* If there are MANY matches, is it possible to determine a modal match? (simplest option), or look at claims data to see what is actually being covered/charged. 
 	* I suspect there is a geographic cost of living component to the rates agreed upon by payers and providers--NY probably charges more than AK. Again, if there are enough possible values, you may be able to use clustering or distributional information to make a best guess for a given hospital-plan-procedure/visit relationship.
 	* If you have historical data that includes some accurate matches, you may be able to extrapolate from that to estimate a current rate.
* _Beyond the current use-case_: There are a number of future directions I could see Serif going in, depending on the healthcare payer-provider ecosystem (and surrounding regulations). 
	* From providing existing data to providing suggested solutions: If you have enough data, you could also, probably, determine a "best value" rate estimate for a given hospital, giving them something to target in their negotiations. Similarly, you could provide data to plans/networks to give a better idea of the landscape. (Note: both sides are probably already doing it to some extent, but having a third party independently doing so may be both a convenience and a negotiation tool.)
	* Navigating changing regulatory environments: In an extreme case, if Medicare and Medicaid are dissolved, it may be prudent to simulate the effects of such a shock to overall rates (almost a consulting/research role). This would likely involve a healthcare market econometrician. In another extreme (longer term, possibly) case, if a public option, or even a Medicare/Medicaid expansion were available, there might also be a market for predictive or exploratory analytics.
	* Independent cost-efficiency research: This is probably MAJOR scope creep, but, if there are MAJOR discrepencies/residuals in rates outside of those due to geography for a given procedure, it may make sense to partner with healthcare providers and plans to determine what, if anything, is driving those outliers. Is it pure profit margin? Is there a questionable supply chain issue? Are providers underutilized? There are a bunch of ways to estimate/calculate healthcare costs (I've got a little experience with time-driven activity based costing). I imagine both payers and facilities would like to drive down rates. (Providers don't love this kind of effort... but that's why more work is always required to research value-based care and other appropaches to fund healthcare. Other countries are **much** better at this than we are.

