# IgA-Nephropathy

Team members: [Amelia Spivak](), [Unnati Nigam](), and [Steve Manns](https://github.com/manns79)

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

To construct our data from the files specified above, we first used d_icd_diagnoses to identify 88 `icd_codes` associated with a diagnosis of some form of kidney disease. With these `icd_codes` at hand, we were then able to turn to diagnoses_icd and identify the subgroup of patients that received a kidney disease diagnosis. This search yielded 41,386 different patients, from which we randomly selected 1,000 for the purpose of analysis. For these 1,000 randomly selected patients with kidney disease, the next step was obtaining all of their lab measurements. In order to be able to work with a more manageable-sized dataset, we used d_labitems to determine the `itemdid` associated with all lab tests that are relevant to our study. This yielded 105 different `itemid` values (see `Data/Minimal Reducedlabitems - Reducedlabitems.csv`) that were used to filter the lab data for the 1,000 randomly selected patients with kidney disease. In other words, from the original labevents file, we extracted 105 different types of lab readings for each of the 1,000 patients. The result of this procedure was `Data/trimmed_kidney_disease_random_1000_patients.csv.gz`.

In addition to studying patients with some form of kidney disease, our analysis also includes patients with healthy kidneys. However, to qualify as a patient with healthy kidneys, we strengthened our requirements on the diagnoses by expanding the number of `icd_codes` to 107. This implies that being part of the complement of the set of patients with kidney disease is a necessary but not sufficient requirement for a patient to fall into the healthy kidney category. Furthermore, due to the larege size of MIMIC-IV, we also required that the patient has a serum creatinine lab reading. After imposing all of these conditions, 1,088 patients remained in the dataset. For these 1,088 patients with healthy kidneys, we then extracted the same 105 different types of lab readings for each patient. The result was `Data/trimmed_normal_kidney_patients.csv.gz`.

Finally, for the purpose of further exploration, we sought to determine all diagnoses that the patients in our two datasets received. To accomplish this task, we searched through diagnoses_icd for the `seq_num`, `icd_code`, and `icd_version` associated with each diagnosis that a patient in one of our two datasets received. This information was stored in a Python dictionary to ensure that we have access to some of each patient's medical history. The result for the patients with kidney disease was `Data/kidney_disease_patients_diagnoses.json`, while the analogous result for patients with health kidneys was `Data/normal_kidney_patient_diagnoses.json`.

The notebooks used to generate our datasets from the MIMIC-IV files may be found in the Data_Generation folder of this repository. The files containing the data may be found in the Data folder of this repository.



## Exploratory Data Analysis

Describe results from EDA here

## Modeling

Describe modeling approach here

## Results

Describe results here
