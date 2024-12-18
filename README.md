# OCD Patient Data Analysis

### Project Overview

This data analysis project is focused on uncovering insights in the trends associated with patients diagnosed with Obsession-Compulsion Disorder (OCD) through a detailed examination of a comprehensive dataset comprising 1500 individuals.
The primary objective of this project is to analyse demographic and clinical characteristics that may influence the manifestaion and treatment of OCD.


### Data Scources

The dataset used in this analysis is fictional and contains 1,500 patients diagnosed with Obsessive-Compulsive Disorder. It includes demographic and clinical information such as Y-BOCS scores, OCD subtypes, and co-occurring conditions. [Kaggle link](https://www.kaggle.com/datasets/ohinhaque/ocd-patient-dataset-demographics-and-clinical-data/)

### Tools
  - Python Pandas : Data Cleaning and Manipulation
  - Python Matplotlib and Seaborn: Data Visulaisation
  - Python Scipy : Scientific and Technical computing

### Data cleaning and preparation

In the initial data preparation phase, the following steps were performed:
  1. Data inspection: Reviewed the dataset for missing or inconsistent values.
     - Handling Inconsitent Data in Categorical Column
       ~~~python
        #Function to convert various values of degree graduate into single value "Graduate"
def graduate(degree):
    if degree=="Some College" or degree=="College Degree" or degree=="Graduate Degree":
        return "Degree Graduate"
    return degree
#Apply the graduate function to the 'Education Level' column in the predefined values
df_patient["Education Level"]=df_patient["Education Level"].apply(graduate)
df_patient["Education Level"].unique()
 ~~~
  2. Handling missing values: Addressed missing values in relevant columns, particularly in demographic and clinical information.
  ~~~python
#Previous Diagnoses and Medications are having more missing values.
#Create indicators for missing values before imputation
df_patient["Previous Diagnosis Missing"]=df_patient["Previous Diagnoses"].isnull().astype(int)
df_patient["Medications Missing"]=df_patient["Medications"].isnull().astype(int)

#Fill missing values with mode as both are categorical data
mode_medications=df_patient["Previous Diagnoses"].mode()[0]
df_patient.fillna({"Previous Diagnoses": "Not Diagnosed"},inplace=True)

mode_medications=df_patient["Medications"].mode()[0]
df_patient.fillna({"Medications": mode_medications},inplace=True)

#Apply one-hot encoding to the categorical variables
df_patient_encoded=pd.get_dummies(df_patient,columns=["Previous Diagnoses","Medications"],drop_first=True)
~~~
3. Data transformation: Reformatted the columns for consistency, such as ensuring all dates were in a standard format, and transformed the Y-BOCS score into categorical ranges.
     ~~~python
     #Date variable in Object format converting that to 'DateTime'
df_patient['OCD Diagnosis Date']=pd.to_datetime(df_patient['OCD Diagnosis Date'])
~~~

### Exploratory Data Analysis
#### KEY AREA OF EXPLORATION WITHIN THIS DATASET INCLUDE:
    - Demographic Characteristics:
        o Understanding the age, gender and geographical distribution of individuals diagnosed with OCD to identify potential patterns and differences across populations.
    - Duration of Symptoms:
        o Analysing how long patients have been experiencing OCD Symptoms. This information can provide valuable context for treatment approaches and understanding the chronicity of the disorder.
    - Y-BOCS Scores:
        o Utilizing the Yale-Brown Obsessive Compulsive Scale (Y-BOCS) score to gauge the severity of OCD symptoms among the population, which can inform treatment efficacy and degree of impact on patients daily lives.
    - Co-occuring Mental Health Conditions:
        o Investigating the presence of other mental health disorders that often accompany OCD, such as anxiety disorders and depression, understanding these relationships can help in designing more comrehensive treatment plans.

### Data Analysis
- Handling Missing Values
~~~python
#Previous Diagnoses and Medications are having more missing values.
#Create indicators for missing values before imputation
df_patient["Previous Diagnosis Missing"]=df_patient["Previous Diagnoses"].isnull().astype(int)
df_patient["Medications Missing"]=df_patient["Medications"].isnull().astype(int)

#Fill missing values with mode as both are categorical data
mode_medications=df_patient["Previous Diagnoses"].mode()[0]
df_patient.fillna({"Previous Diagnoses": "Not Diagnosed"},inplace=True)

mode_medications=df_patient["Medications"].mode()[0]
df_patient.fillna({"Medications": mode_medications},inplace=True)

#Apply one-hot encoding to the categorical variables
df_patient_encoded=pd.get_dummies(df_patient,columns=["Previous Diagnoses","Medications"],drop_first=True)
~~~
- Handling Incosistent Data
### Findings
