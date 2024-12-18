## Table of Content
  - [Project Overview](#Project-Overview)
  - [Data Scources](#Data-Scources)
  - [Tools](#Tools)
  - [Data cleaning and preparation](#Data-cleaning-and-preparation)
  - [Exploratory Data Analysis](#Exploratory-Data-Analysis)
  - [Findings](#Findings)
  - [Recommandations](#Recommandations)
  - [Limitations](#Limitations)

### Project Overview

This data analysis project is focused on uncovering insights in the trends associated with patients diagnosed with Obsession-Compulsion Disorder (OCD) through a detailed examination of a comprehensive dataset comprising 1500 individuals.
The primary objective of this project is to analyse demographic and clinical characteristics that may influence the manifestaion and treatment of OCD.

![Untitled design (2)](https://github.com/user-attachments/assets/d3447608-9c68-4022-bb17-95bb756f80b9)

### Data Scources

The dataset used in this analysis is fictional and contains 1,500 patients diagnosed with Obsessive-Compulsive Disorder. It includes demographic and clinical information such as Y-BOCS scores, OCD subtypes, and co-occurring conditions. [Kaggle link](https://www.kaggle.com/datasets/ohinhaque/ocd-patient-dataset-demographics-and-clinical-data/)

### Tools
  - Python Pandas : Data Cleaning and Manipulation
  - Python Matplotlib and Seaborn: Data Visulaisation
  - Python Scipy : Scientific and Technical computing

### Data cleaning and preparation

In the initial data preparation phase, the following steps were performed:
  1. **Data inspection**: Reviewed the dataset for missing or inconsistent values.
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
  2.**Handling missing values**: Addressed missing values in relevant columns, particularly in demographic and clinical information.
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
3. **Data transformation**: Reformatted the columns for consistency, such as ensuring all dates were in a standard format, and transformed the Y-BOCS score into categorical ranges.
~~~python
     #Date variable in Object format converting that to 'DateTime'
df_patient['OCD Diagnosis Date']=pd.to_datetime(df_patient['OCD Diagnosis Date'])
~~~

### Exploratory Data Analysis
#### KEY AREA OF EXPLORATION WITHIN THIS DATASET INCLUDE:
    - Demographic Characteristics**:
        o Understanding the age, gender and geographical distribution of individuals diagnosed with OCD to identify potential patterns and differences across populations.
    - Duration of Symptoms:
        o Analysing how long patients have been experiencing OCD Symptoms. This information can provide valuable context for treatment approaches and understanding the chronicity of the disorder.
    - Y-BOCS Scores:
        o Utilizing the Yale-Brown Obsessive Compulsive Scale (Y-BOCS) score to gauge the severity of OCD symptoms among the population, which can inform treatment efficacy and degree of impact on patients daily lives.
    - Co-occuring Mental Health Conditions:
        o Investigating the presence of other mental health disorders that often accompany OCD, such as anxiety disorders and depression, understanding these relationships can help in designing more comrehensive treatment plans.

#### OCD Patient Diagnosis over a Time
~~~python
#Grouping Data by Diagnosis Year and Month
monthly_counts=df_patient.groupby(df_patient['OCD Diagnosis Date'].dt.to_period('M')).size()

# Convert to a more suitable format for plotting
monthly_counts.index = monthly_counts.index.to_timestamp()  # Convert PeriodIndex to Timestamp
monthly_counts = monthly_counts.reset_index(name='Count')  # Reset index to have DataFrame format

#Plotting a graph 
plt.figure(figsize=(14,4))
plt.plot(monthly_counts["OCD Diagnosis Date"],monthly_counts["Count"],color="black",marker="o")
plt.title("Number of Diagnosis Over Time\n")
plt.xlabel("Date")
plt.ylabel("Number of Diagnosis")
plt.grid()
#plt.xticks(df_patient["OCD Diagnosis Date"].dt.year)
plt.show()
~~~
![Screenshot (35)](https://github.com/user-attachments/assets/db11fcbc-c52b-4255-9c72-51de51f853e3)


#### Distribution of Obsession Type and Compulsion Type
~~~python
#Create a facet grid
com_type = sns.FacetGrid(df_patient, col='Obsession Type', col_wrap=5, height=5)

#Map to count plot
com_type.map(sns.countplot, 'Compulsion Type', order=df_patient['Compulsion Type'].value_counts().index,hue=df_patient['Compulsion Type'],palette='light:#1f77b4')

for ax in com_type.axes.flatten():  # Loop through each axis
    for p in ax.patches:  # Loop through the bars
        ax.annotate(f'{p.get_height()}',  # Height of the bar (count)
                    (p.get_x() + p.get_width() / 2., p.get_height()),  # Position (x, y)
                    ha='center', va='bottom',  # Horizontal and vertical alignment
                    fontsize=10, color='black',  # Font size and color
                    xytext=(0, 5),  # Adjust the label position
                    textcoords='offset points')  # Offset points

#set axis label and title 
com_type.set_axis_labels("Compulsion Type")
com_type.set_xticklabels(rotation=90,label=df_patient["Compulsion Type"])
com_type.set_titles(col_template='{col_name}')

#Update subplot total obsession counts
total_count=df_patient["Obsession Type"].value_counts()

#Update subplot titles to include obsession counts
for ax, obs_type in zip(com_type.axes.flatten(),total_count.index):
    count=total_count[obs_type]
    ax.set_title(f"{obs_type} (Total : {count})")
com_type.fig.suptitle('Distribution of Obsession Types by Compulsion Type\n\n', fontsize=16)

plt.legend()
com_type.fig.subplots_adjust(top=0.75)
plt.show()
~~~
![Screenshot (36)](https://github.com/user-attachments/assets/e528732d-55b7-48ba-b477-7ac96da88efd)

#### Variation of Y-BOCS Score across Ethnicity
~~~python
#Grouping data by Ethnicity  
ethnicity_dist=df_patient.groupby("Ethnicity",as_index=False).agg(
    Ethnicity_count=("Ethnicity","size"),
    Avg_Age=("Age","mean"),
    Mean_Duration_symptoms_months=("Duration of Symptoms (months)","mean"),
    Mean_Y_BOCS_Score_Obsession=("Y-BOCS Score (Obsessions)","mean"),
    Mean_Y_BOCS_Score_Compulsion=("Y-BOCS Score (Compulsions)","mean")
)

#Sorting data by Ethnicity count 
ethnicity_dist.sort_values(by="Ethnicity_count",ascending=False)

#Styling the table
ethnicity_dist.style.format({
    'Avg_Age':'{:,.2f}',
    'Mean_Duration_symptoms_months':'{:,.2f}',
    'Mean_Y_BOCS_Score_Obsession':'{:,.2f}',
    'Mean_Y_BOCS_Score_Compulsion':'{:,.2f}'
}).set_table_attributes('style="font-size: 14px; border-collapse: collapse;"').set_table_styles(
    [{'selector': 'th', 'props': [('background-color', 'lightblue'), ('border', '1px solid black')]},
     {'selector': 'td', 'props': [('border', '1px solid black')]}]
)
~~~
![Screenshot (37)](https://github.com/user-attachments/assets/9682b36a-e334-40d2-9163-be7063e578bc)


#### Statistical Analysis to find the Variation of Y-BOCS among Obsession and Compulsion Types
~~~python
#perform the ANOVA for Obsession Score
print("Result of ANOVA Test for Obsession Score:\n")
model_obs=ols('Q("Y-BOCS Score (Obsessions)") ~ Q("Obsession Type")', data=df_patient).fit()
annova_table_obs=sm.stats.anova_lm(model_obs,typ=2)
print(annova_table_obs,"\n\n\n")

#Perform the ANOVA for Compulsion Score
print("Result of ANOVA Test for Compulsion Score:\n")
model_com=ols('Q("Y-BOCS Score (Compulsions)") ~ Q("Compulsion Type")', data=df_patient).fit()
annova_table_com=sm.stats.anova_lm(model_com,typ=2)
print(annova_table_com)
~~~
![Screenshot (40)](https://github.com/user-attachments/assets/f2de55ac-49d8-46b5-9519-ac9fb1278d9b)

~~~python
#Performing Post Hoc Testing

#For Compulsion Score
print("\nResult of Post Hoc Testing for Compulsion Score:\n")
tukey_results_com = pairwise_tukeyhsd(endog=df_patient['Y-BOCS Score (Compulsions)'], groups=df_patient['Compulsion Type'], alpha=0.05)
print(tukey_results_com)
~~~
![Screenshot (41)](https://github.com/user-attachments/assets/8226179f-5596-4343-980c-a258a7fe0c60)

**Conclusion**:
    - The Tukey HSD post-hoc analysis revealed a significant difference in compulsion score between the 'Checking' and 'Praying' group scoring higher. 
    - No meaningful differences among the other groups (Counting, Ordering and Washing)
    - These findings highlights the uniques influence of the 'Praying' compulsion type and further analysis.
#### CORRELATION ANALYSIS OF DIFFERENT CLINICAL DATA
~~~python
#Convert Categorical Variable to binary
df_patient['Anxiety Diagnosis binary']=df_patient["Anxiety Diagnosis"].apply(lambda x: 1 if x=='Yes' else 0)
df_patient['Depression Diagnosis binary']=df_patient["Depression Diagnosis"].apply(lambda x: 1 if x=='Yes' else 0)

#Selecting Relavant columns for analysis
relavant_columns=[
    'Y-BOCS Score (Compulsions)',
    'Y-BOCS Score (Obsessions)',
    'Anxiety Diagnosis binary',
    'Depression Diagnosis binary',
    'Medications', #Categorical Data
    'Compulsion Type', #Categorical Data
    'Obsession Type', #Categorical Data
    'Previous Diagnoses' #Categorical Data
]

#Create a new DataFrame with relavant columns
df_relavant=df_patient[relavant_columns]

#One-hot encoding on the relavant Categorical data columns to convert it to numerical format
df_relavant=pd.get_dummies(df_relavant,columns=["Medications","Compulsion Type","Obsession Type","Previous Diagnoses"])


#Calculate Pearson correlation coefficients
corr=df_relavant.corr(method='spearman')

#Filter the correlation matrix for values >0.03 and <-0.03
mask=(corr>0.05) | (corr<-0.05)
filtered_corr= corr[mask]

#Plot the heatmap for the filtered correlation matrix
plt.figure(figsize=(20,14))
sns.heatmap(filtered_corr, annot=True, fmt='.3f', cmap='coolwarm', square=True, mask=filtered_corr.isnull())

#sns.heatmap(corr,annot=True,fmt='.2f',cmap='coolwarm',square=True)
plt.title("Correlation Heatmap of Clinical Factors and Compulsion Scores\n")
plt.show()
~~~
![Screenshot (33)](https://github.com/user-attachments/assets/a53a7f8f-b776-4c3c-9d12-58990c90b6e1)

**Correlation Analysis Summary**

- **Y-BOCS Compulsion Score**: Weak negative correlation of **-0.071** with checking compulsions indicates that checking behaviors are linked to lower compulsion severity.  
- **Benzodiazepines**:  
  - Strong negative correlation with SNRIs: **-0.591**  
  - Strong negative correlation with SSRIs: **-0.570**  
  - Suggests patients on benzodiazepines are less likely to use these antidepressants concurrently.
- **SSRIs**: Weak negative correlation with compulsions: **-0.046**, indicating minimal impact on reducing compulsion severity.
- **Compulsion and Obsession Types**: Significant negative correlations of **-0.23 to -0.27** among different compulsion and obsession types highlight specific patterns in OCD symptoms.
- **Previous Diagnoses**: Weak negative correlation with MDD: **-0.087** indicates possible independent diagnoses, with significant overlaps (-0.25 to -0.27) among other conditions.
  
### Findings
#### KEY FINDINGS AND THEIR SIGNIFICANCE:

**1. Prevalence of Symptoms**:
    - Y-BOCS Scores indicate that the average scores for obsessions **(20.1)** and compulsions **(19.6)** show **significant levels of distress** in OCD patients. The high standard deviation (11.78) suggests considerable variability among individuals, highlighting the **need for tailored treatment plans**.
    - Significance: This variability is cruicial for clinicians as it emphasized treatment strategies. It shows that common treatment protocols may not be effetive for all patients.

**2. Correlation Among Variables**:
    - The **"negative correlations"** observed between types of compulsions imply that individuals exhibiting one type of symtoms are less likely to display another.For instance, the significant negative correlation among **compulsion types (-0.23 to -0.27)** and **obsession types (-0.24 to -0.27)** indicates **distict symptoms** presentations within OCD.
    - Significance: Understanding these patterns can help in refining treatment approaches by targeting specific symptoms, which could improve treatment efficacy.

**3. Influence of Medications**:
    - The **"strong negative correlation"** between **Benzodiazepines and SNRIs/SSRIs** indicates that patients taking benzodiazepines are less likely to be on these antidepressants simultaneously. This could that clinicians need to navigate thoughtfully.
    - Significance: These findings can guide prescribing practices and highlight areas where further education regarding medication management is needed.

**4. Demographic Insights**:
    - There is **relatively balanced** gender representation among OCD patients, and differences in Y-BOCS scores with age and gender suggest variations in symptom severity. For instance, **older women reported higher obsession severity**.
    - Significance:This findings underlines the necessity for age and gender specific therapeutic strategies, recognizing how demographic factors can influence symptoms expression.
**5. Previous Diagnoses**:
    - The significant **negative correlations** among **previous diagnoses (GAD,MDD,PTSD)** suggest that having one diagnosis could preclude the occurence of another.
    - Significance: This calls for clear diagnostic criteria and an understanding that comorbidity might not be as common as preceived. It helps in refining assessment protocols.

**6. Family History**:
    - Patients **with** a family history of OCD **show higher mean Y-BOCS scores for obsessions**, although the statistical significance is borderline.
    - Significant : This points towards potential genetic or environmental factors influencing OCD severity, which should be factored into clinical assessments.

### Recommandations
1. **Personalized Treatment Plans**  
   - **Finding:** High variability in Y-BOCS scores indicates significant differences in symptom severity among patients.  
   - **Recommendation:** Clinicians should adopt individualized treatment strategies tailored to specific symptoms (obsessions or compulsions) rather than relying solely on standardized protocols.  

2. **Refined Diagnostic Framework**  
   - **Finding:** Negative correlations among previous diagnoses (e.g., GAD, MDD, PTSD) suggest comorbidity may be less common than perceived.  
   - **Recommendation:** Diagnostic criteria should be updated to account for these findings, ensuring clearer differentiation between overlapping disorders.  

3. **Demographic-Specific Strategies**  
   - **Finding:** Age and gender differences influence OCD symptom severity, with older women showing higher obsession scores.  
   - **Recommendation:** Treatment plans should consider demographic factors, incorporating age- and gender-specific therapeutic strategies to address variations in symptom presentation.  

4. **Medication Management**  
   - **Finding:** A strong negative correlation between Benzodiazepines and SNRIs/SSRIs highlights distinct prescribing trends.  
   - **Recommendation:** Provide clinicians with updated guidelines to navigate medication combinations more effectively and ensure optimal treatment outcomes.  

### Limitations
1. **Small Sample Size**  
   - The dataset of 1,500 patients limits the ability to generalize findings across diverse populations and accurately represent the full spectrum of OCD symptoms and treatment responses.  

2. **Lack of Longitudinal and Socioeconomic Data**  
   - The absence of long-term and socioeconomic data restricts insights into symptom progression, treatment outcomes, and disparities in healthcare access.  
