# IgA-Nephropathy

Team members: [Amelia Spivak](https://github.com/AmeliaSpivak) and [Steve Manns](https://github.com/manns79)

# Table of Contents
1. [Introduction](#introduction)
2. [Dataset Generation](#dataset-generation)
3. [Exploratory Data Analysis](#exploratory-data-analysis)
4. [Modeling](#modeling)
5. [Results](#results)

## Introduction

Add intro here

## Dataset Generation

Our study utilizes data from the fourth iteration of the Medical Information Mart for Intensive Care ([MIMIC-IV](https://physionet.org/content/mimiciv/3.1/)) database. MIMIC-IV is a freely accessible, de-identified electronic health record (EHR) database containing comprehensive information on hospital stays for patients at Beth Israel Deaconess Medical Center (BIDMC). The following files from MIMIC-IV were used to generate our data:
* labevents: Laboratory measurements sourced from patient derived specimens.
* d_labitems: Dimension table for labevents; provides a description of all lab items.
* diagnoses_icd: Billed ICD-9/ICD-10 diagnoses for hospitalizations.
* d_icd_diagnoses: Dimension table for diagnoses_icd; provides a description of ICD-9/ICD-10 billed diagnoses.

To construct our data from the files specified above, we first used d_icd_diagnoses to identify 88 `icd_codes` associated with a diagnosis of some form of kidney disease. With these `icd_codes` at hand, we were then able to use diagnoses_icd to identify the subgroup of patients that received a kidney disease diagnosis. This search yielded 41,386 different patients from which we randomly selected 1,000 for the purpose of analysis. For these 1,000 randomly selected patients with kidney disease, the next step was obtaining all of their lab measurements. In order to be able to work with a more manageable-sized dataset, we used d_labitems to determine the `itemdid` associated with all lab tests that were relevant to our study. This yielded 105 different `itemid` values (see `Data/Minimal Reducedlabitems - Reducedlabitems.csv`) that were used to filter the lab data for the 1,000 randomly selected patients with kidney disease. In other words, from the original labevents file, we extracted 105 different types of lab readings for each of the 1,000 patients. The result of this procedure was `Data/trimmed_kidney_disease_random_1000_patients.csv.gz`.

In addition to studying patients with some form of kidney disease, our analysis also included patients without a kidney disease diagnosis. We refrained from referring to this group as "healthy kidney patients", as it's possible that some of these patients had a form of kidney disease that went undiagnosed (e.g., their admission to BIDMC was due to a different medical problem). Therefore, we called this group "non-kidney patients". To qualify as a non-kidney patient, we strengthened our requirements on the diagnoses by expanding the number of `icd_codes` to 107. This implies that being included in the complement of the set of patients with kidney disease is a necessary but not sufficient condition for a patient to fall into the non-kidney category. Furthermore, due to the large size of MIMIC-IV and the important role played by serum creatinine, we also required that the patient had a serum creatinine lab reading. After imposing all of these conditions, 1,088 patients remained in the dataset. For these 1,088 non-kidney patients, we then extracted the same 105 different types of lab readings for each patient. The result was `Data/trimmed_normal_kidney_patients.csv.gz`.

Finally, for the purpose of further exploration, we sought to determine all diagnoses that the patients in our two datasets received. To accomplish this task, we searched through diagnoses_icd for the `seq_num`, `icd_code`, and `icd_version` associated with each diagnosis that a patient in one of our two datasets received. This information was stored in a Python dictionary to ensure that we had access to some of each patient's medical history. The result for the patients with kidney disease was `Data/kidney_disease_patients_diagnoses.json`, while the analogous result for non-kidney patients was `Data/normal_kidney_patient_diagnoses.json`.

The notebooks used to generate our datasets from the MIMIC-IV files may be found in the Data_Generation folder of this repository. The files containing the data may be found in the Data folder of this repository.



## Exploratory Data Analysis

As a first step in our exploratory data analysis (EDA), we sought to gather evidence that the health of the kidney disease and non-kidney patients was comparable (excluding the fact that patients in one group received a kidney disease diagnosis). Furthermore, we wanted to understand any minor differences that were identified between the two groups. Our approach to handling this task entailed analyzing the following three lab tests:
1. C-Reactive Protein (CRP) (`itemid` = 50889)
2. Sedimentation Rate (`itemid` = 51288)
3. Hemoglobin (`itemid` = 50811)

CRP and sedimentation rate are two measures of inflammation, which is a metaphor for how much illness there is to fight off in the body. Starting with CRP, we found that 324 kidney patients and 219 non-kidney patients in our dataset have a CRP reading. For these patients, we found their first CRP lab reading and used this data to produce the histogram shown below. Note that the additional tick marks included at 5 and 10 are intended to indicate the normal range. Namely, CRP readings equal to or greater than 8 mg/L or 10 mg/L are considered high (see: [Mayo Clinic](https://www.mayoclinic.org/tests-procedures/c-reactive-protein-test/about/pac-20385228)). Focusing on the normal range, roughly 10% of the non-kidney patients and 7% of the kidney disease patients had CRP readings between 0 mg/L and 5mg/L. Results for the remaining bins may be interpretted similarly. Importantly, the percentages for the two groups are comparable across the different bins, which agrees with the assertion that the health of the kidney disease and non-kidney patients is similar. Lastly, we remark that the abnormally high readings were present in the data, which was suspected to be an artifact of the patients being in an ICU. To avoid detracting from the data of interest (i.e., patients with reasonabl health), we chose to exclude CRP readings greater than 150 from the plot shown below. However, for the purpose of completeness, such readings are recorded here. For kidney patients, 26 had a CRP reading greater than 150. For non-kidney patients, 13 had a CRP reading greater than 150. 

![CRP_EDA](Assets/crp_eda.png)

Sedimentation rate was the next lab that was analyzed. As mentioned above, sedimentation rate is another measure of inflammation. According to the [Cleveland Clinic](https://my.clevelandclinic.org/health/diagnostics/17747-sed-rate-erythrocyte-sedimentation-rate-or-esr-test), values greater than 30 are considered high, although this can vary based on factors such as age and sex. In our dataset, there were 187 kidney patients with a sedimentation rate reading and 94 non-kidney patients with a sedimentation rate reading. The first sedimentation rate lab reading for these patients was extracted and used to create the histogram shown below. Based on the histogram, about 3.5% fall into the bin 0-10, while about 1.3% of the kidney patients fall into this bin. This is the most significant difference between the two groups, with percentages in the other bins differing by less than 0.5%. With that said, the sedimentation rate labs offer additional evidence that the health of the kidney and non-kidney patients is comparable. 

![sed_rate_eda](Assets/sed_rate_eda.png)

The final lab that was analyzed in the first part of our EDA was hemoglobin. Hemoglobin is a protein in red blood cells, and low hemoglobin levels may be an indication that a disease or condition is affecting the body's ability to produce red blood cells ([Cleveland Clinic](https://my.clevelandclinic.org/health/symptoms/17705-low-hemoglobin)). The consequence of low hemoglobin is that body doesn't get enough oxygen, which causes one to feel very tired and weak. What is considered the normal range for hemoglobin can vary slightly depending on sex, but 12 g/dL is considered low. In our dataset, 382 kidney patients had a hemoglobin reading and 148 non-kidney patients had a hemoglobin readings. Similar to the previous two labs, for this subset of patients, we extracted their first hemoglobin lab reading. The data obtained through this procedure is shown in the histogram below. Notice that the two histograms are roughly centered around the same value, but there are some noticeable differences between the two groups. To that end, kidney patients have relatively lower levels of hemoglobin, on average. We suspected that this observation was a consequence of a larger proportion of kidney patients having heart disease (relative to non-kidney patients).

![hemoglobin_eda](Assets/hemoglobin_eda.png)

As a second step in our EDA, we sought to:
- analyze feature distributions
- explore feature-target relationships
- examine relationships among the features

Importantly, this analysis was conducted using only the training data. In order to analyze the feature distributions, we constructed a histogram for each feature using all of the patients (in the training data). The result is shown in the figure below. The most noteworthy takeaway from this histogram is that the distribution of creatinine and bun are right-skewed. This is expected from a dataset where half of patients have kidney disease, as both high creatinine and bun are indicators for kidney disease. The figure shown below also indicates that, to a lesser extent, the distribution of potassium and calcium are also right-skewed. 

![feature_distribution_eda](Assets/feature_distribution.png)

To explore feature-target relationships, we again constructed a histogram for each feature. However, in contrast to the plot shown above, this time we created a histogram for each of the two groups. That is, one histogram for kidney disease patients, and one histogram for non-kidney patients. These are shown in red and blue, respectively, in the figure below. Focusing for now on the creatinine histograms, notice that the non-kidney patients are clustered very tightly at low values of creatinine. On the other hand, the vast majority of the kidney disease patients lie entirely to the right of the non-kidney patient distribution. In other words, patients with kidney disease have noticeably higher levels of creatinine. As remarked above, this observation is expected. Looking next at the histograms for bun, a similar interpretation can be made. Namely, the majority of non-kidney patients lie to the left of roughly 30, while the majority of kidney disease patients lie to the right of that point. This is also consistent with our medical knowledge about kidney disease. All in all, the histograms for creatinine and bun lead to the expectation that these two features will play a critical role in being able to classify patients according to whether or not they have kidney disease. As for the remaining features, although medical knowledge leads one to expect that they should play a role in the classification, the figure shown below indicates that there significance is not nearly as great as creatinine and bun.

![feature_target_relationship_eda](Assets/feature_target_relationship.png)

Lastly, to examine relationships among the features, we created the pair plot shown below. In this part of the analysis, we were particularly interested in the correlation among the different features. The usefulness of understanding the correlation among the different features is that it indicates which features are providing essentially the same information. For example, the first row of the pair plot shows that there is at least a mild correlation between albumin and calcium (third plot in this row) as well as between albumin and hemoglobin (last plot in this row). This observation can be confirmed by computing the correlation matrix, which in this case yields a correlation of 0.53 for albumin and calcium, and a correlation of 0.61 for albumin and hemoglobin. Therefore, given a model that includes albumin as a feature, one should expect little improvement from adding either calcium or hemoglobin (or both) to the model. Analogous interpretations can be made for the remaining rows of the pair plot, and such interpretations helped inform our modeling process. 

![pair_plot_eda](Assets/pair_plot.png)



## Modeling

Based on our domain knowledge and the available data, nine different lab tests were considered as features during the modeling process. These were: albumin, bicarbonate, bun, bun/creatinine, calcium, chloride, creatinine, hemoglobin, and potassium. Although the EDA suggested that, excluding creatinine, bun would play a critical role in the classification, we considered a variety of combinations of these features. There were two purposes for considering different combinations of the nine features. First, from a medical perspective, the lab results that were selected as features are expected to provide a meaningful indication of kidney health. Therefore, it was of interest to see whether this expectation was lived out in practice. Second, the computational cost and time needed to fit additional models was low, so considering multiple combinations of the features offered a greater level of completeness with little additional expense. That being said, the combinations of the nine features that we considered are:
1. albumin
2. bicarbonate
3. creatinine
4. bun
5. bun/creatinine
6. albumin and bicarbonate
7. albumin and bun
8. albumin and bun/creatinine
9. albumin and creatinine
10. bicarbonate and bun
11. bicarbonate and bun/creatinine
12. albumin, creatinine, and bun
13. albumin, bicarbonate, and bun
14. albumin, bicarbonate, and bun/creatinine
15. albumin, bun, bicarbonate, and creatinine

For each of the 15 combinations of features mentioned above, we used two different classification algorithms. These were logistic regression and k-nearest neighbors. While there are a variety of different classification algorithms that one may consider applying to the data, many of these are not appropriate in this case. This is because the requirement that the patients in our dataset have all of the necessary lab measurements taken on the same day significantly reduced the size of our dataset (i.e., reduced it down to 764 total patients). Therefore, machine learning algorithms which are designed to be used with large datasets can easily memorize our training set, but are not expected to generalize well to new unseen data. We demonstrated this point by using a gradient boosting classifier with albumin, bun, bicarbonate, calcium, chloride, hemoglobin, and potassium as features. Specifically, this gradient boosting classifier obtained 99.67% accuracy on the training set, with a 0.66% false negative rate. Such seemingly impressive metrics would make our results comparable to those in papers such as [Chronic kidney disease prediction based on machine learning algorithms](https://doi.org/10.1016/j.jpi.2023.100189); however, as part of an effort to produce models with a greater potential to generalize well, we stuck with logistic regression and k-nearest neighbors. 

Hyperparameter tuning was performed for each of the two classification algorithms that were applied to the data. For logistic regression, this entailed tuning the value of the inverse of regularization strength (`C`). The values of `C` that were tested were [0.001, 0.01, 0.1, 1, 10, 100], For k-nearest neighbors, this entailed tuning the value of the number of nearest neighbors (`n_neighbors`). The values of `n_neighbors` that were tested were 1, 2, ..., 20. All hyperparameter tuning was accomplished by using 5-fold cross-validation. In particular, the hyperparameter value with the highest average accuracy score was selected. It was the accuracy score, false negative rate, and interpretation of such classifiers that were considered when determining our best model and interpretting the results. As a final sanity check, the best model was evaluated on the test set to ensure that a comparable level of accuracy was obtained. 


## Results

Describe results here
