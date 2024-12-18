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

####Distribution of Obsession Type and Compulsion Type
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
####Variation of Y-BOCS Score across Ethnicity
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
### Findings
