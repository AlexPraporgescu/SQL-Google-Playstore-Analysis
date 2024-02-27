  # Google Playstore Analysis (SQL+Power BI)

  ## <ins>Table of Contents</ins>

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Preparation and Cleaning](#data-preparation-and-cleaning)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Visualisation](#visualisation)
- [Results and Findings](#results-and-findings)
- [Recommendations](#recommendations)
- [Limitations](#limitations)
- [References](#references)
- [Notes](#notes)

  ## <ins>Project Overview</ins>

The goal is to analyze the Google Playstore Apps dataset to derive insights into the app market.
I aim to understand the factors that contribute to an app's success, including its user ratings, reviews and category.
I want to explore user sentiments towards apps by analyzing the user reviews dataset.
Additionally, insights into the popularity of app categories based on the total number of installs and the sentiment polarity of user reviews can be seen.
The ultimate objective is to provide recommendations for app developers to enhance their app's performance and user satisfaction.

  ## <ins>Data sources</ins>

App data: The primary dataset used for this analysis was the "Playstore.csv" containing detailed information about
Android apps, categories, genres, installs, etc.

Review data: The "Reviews.csv" dataset containing users reviews, sentiments and polarity of sentiment was also used.
These datasets were obtained from Kaggle.com.

  ## <ins>Tools</ins>

 - MySQL - Data cleaning, preparation and exploratory data analysis (EDA) [Download here](https://dev.mysql.com/downloads/).
 - Power BI - Visualisation and creating the [Dashboard](#project-overview) [Download here](https://www.microsoft.com/en-us/download/details.aspx?id=58494).

  ## <ins>Data Preparation and Cleaning</ins>

After creating the 'google_playstore' database in MySQL, I imported the data from the two .csv files as text data type to ensure that there will be no data loss.
The next step was to check the tables and modify the columns to their corresponding format type:

```sql
SELECT * FROM playstore;

ALTER TABLE playstore MODIFY COLUMN rating DOUBLE;

ALTER TABLE playstore MODIFY COLUMN reviews BIGINT;

ALTER TABLE playstore MODIFY COLUMN installs BIGINT;

ALTER TABLE playstore MODIFY COLUMN price DOUBLE;

ALTER TABLE playstore MODIFY COLUMN last_updated DATE;

SELECT * FROM reviews;

ALTER TABLE reviews MODIFY COLUMN sentiment_polarity DOUBLE;

ALTER TABLE reviews MODIFY COLUMN sentiment_subjectivity DOUBLE;
```

In order to clean and prepare the data for analysis I had to perform several tasks such as:

 - Identifying and deleting NULL values

```sql
-- Identifying NULL values.
SELECT *
FROM playstore
WHERE App IS NULL
  OR Category IS NULL
  OR Rating IS NULL
  OR Reviews IS NULL
  OR Size IS NULL
  OR Installs IS NULL
  OR Type IS NULL
  OR Price IS NULL
  OR Content_Rating IS NULL
  OR Genres IS NULL
  OR Last_Updated IS NULL
  OR Current_Ver IS NULL
  OR Android_Ver IS NULL;

-- Deleting NULL values.
DELETE
FROM playstore
WHERE App IS NULL
  OR Category IS NULL
  OR Rating IS NULL
  OR Reviews IS NULL
  OR Size IS NULL
  OR Installs IS NULL
  OR Type IS NULL
  OR Price IS NULL
  OR Content_Rating IS NULL
  OR Genres IS NULL
  OR Last_Updated IS NULL
  OR Current_Ver IS NULL
  OR Android_Ver IS NULL;
```

 - Identifying and deleting duplicate values, keeping only the records with the maximum number of reviews:

```sql
-- Identifying duplicate values.
SELECT 
    App, 
    COUNT(App)
FROM playstore
GROUP BY App
HAVING COUNT(App) > 1;

-- Deleting duplicate values.
DELETE A FROM playstore AS A
INNER JOIN playstore AS B
WHERE 
    A.App = B.App AND
    A.Reviews < B.Reviews;
```

  ## <ins>Exploratory Data Analysis</ins>

Exploratory data analysis involved exploring the Playstore data to fulfil the following requirements of the stakeholders:

### **1) Provide the overview of the dataset by how many total unique apps and categories are in our dataset.**

```sql
SELECT 
	COUNT(DISTINCT App) AS total_distinct_apps,
	COUNT(DISTINCT Category) AS total_distinct_category
FROM playstore;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/68689296-d91f-4824-a942-41cc1726fd24)

### **2) Retrieve the unique app categories and the count of apps in each category.**

```sql
SELECT
	Category,
	COUNT(App) AS total_apps
FROM playstore
GROUP BY Category
ORDER BY total_apps DESC;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/25e57eff-a0a3-4858-8878-489a7e5fef0c)

### **3) Identify the top-rated free apps.**

```sql
SELECT
	App,
	Category,
	Rating,
	Reviews
FROM playstore
WHERE TYPE='Free'
ORDER BY Rating DESC
LIMIT 10;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/efb84b70-d165-46c8-9033-42a13df4f806)

### **4) Find the apps with the highest number of reviews.**

```sql
SELECT
	DISTINCT App,
	Category,
	Reviews
FROM playstore
ORDER BY Reviews DESC
LIMIT 10;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/e6f89628-79b8-4796-9c0d-85b8e0e89abf)

### **5) Calculate the average rating for each app category.**

```sql
SELECT
	Category,
	AVG(DISTINCT Rating) AS average_rating
FROM playstore
GROUP BY Category
ORDER BY average_rating DESC
LIMIT 10;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/30a0924a-4bc9-4cfe-ab87-91bc1ca322e5)

### **6) Total amount of revenue generated by Google Playstore by hosting apps.**

```sql
SELECT
	SUM(Price*Installs) AS total_revenue
FROM playstore;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/71412e4a-dbb3-4da5-b403-3c98409f35ab)

### **7) Identify the app categories with the highest total number of installs.**

```sql
SELECT
	Category,
	SUM(Installs) AS total_installs
FROM playstore
GROUP BY Category
ORDER BY SUM(Installs) DESC
LIMIT 10;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/145378fd-5bbd-44b1-9f41-d04b57e14815)

### **8) Find the genre with the most number of published apps.**

```sql
SELECT
	Genres,
	COUNT(DISTINCT App) AS published_apps
FROM playstore
GROUP BY Genres
ORDER BY published_apps DESC
LIMIT 1;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/36286805-8ea2-4481-943b-6838a1ebd38e)

### **9) Provide the list of apps that can work on Android version 4.0.3 and up.**

```sql
SELECT
	DISTINCT App
FROM playstore
WHERE Android_Ver='4.0.3 and up';
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/17128008-25c6-4b89-830d-fbc4d2e60b89)

### **10) Identify the best gaming app (the best = having the highest number of reviews).**

```sql
SELECT
	App,
	Reviews
FROM playstore
WHERE Reviews=(SELECT MAX(Reviews) FROM playstore WHERE Category='GAME')
    AND Category='GAME';
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/e3ddb5ee-0634-4540-b4e9-6b1054d4af97)

### **11) Analyze the average sentiment polarity of user reviews for each app category.**

```sql
SELECT
	Category,
	AVG(sentiment_polarity) AS avg_sent_polarity
FROM playstore
INNER JOIN reviews
ON playstore.App=reviews.App
GROUP BY Category
ORDER BY avg_sent_polarity DESC;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/28939325-9dfc-4f69-bc0c-c91b8dc562c2)

### **12) Provide the distribution of sentiments across different app categories.**

```sql
SELECT
	Category,
	Sentiment,
	COUNT(*) AS total_sentiment
FROM playstore
INNER JOIN reviews
ON playstore.App=reviews.App
GROUP BY Category, Sentiment
ORDER BY total_sentiment DESC;
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/2be2fcb5-ac4d-42ac-8b94-2d968bb2e831)

### **13) Extract all negative sentiment reviews for 'Clash of Clans' with their sentiment polarity and sentiment subjectivity.**

```sql
SELECT
	Translated_Review,
	sentiment_polarity,
	sentiment_subjectivity
FROM reviews
WHERE App='Clash of Clans' AND Sentiment='Negative';
```

Result:

![image](https://github.com/AlexPraporgescu/SQL-Google-Playstore-Analysis/assets/158141333/e7fa1d6c-425e-4ae7-bbf6-ea47714a4e45)

  ## <ins>Visualisation</ins>

The visualisation of the key aspects of the dataset was done by using the following tools available in Power BI:
 - Cards.
 - Bar charts.
 - Donut charts.
 - Line graphs.
 - Tables.
 - Slicers (for creating the interactive dashboard).

The dashboard that I created can be seen [here](++++++++++)

  ## <ins>Results and Findings</ins>

Some of the most important insights are:
 - The app category with the best average rating is the education category having an average rating of 4.28.
 - The apps categories with the highest number of installs are communication followed by game and tools.
 - The genre with the highest number of published apps is tools with 716 published apps.
 - The best gaming app (highest number of rewiews) is Clash of Clans.
 - The category with the most positive sentiments is game with a total of 3860.

  ## <ins>Recommendations</ins>

Based on the analysis, the following actions can be recommended:
 - The best recommendation for app developers is to develop an educational or communication app, as those are the most installed/have the best ratings.
 - Developers should stay away from the tools genre, as it has the highest number of published apps.
 - If the subject is a gaming app, the data shows that the style of Clash of Clans game is the most engaging for users to write reviews on.
 - Also, game category has the most positive sentiments, so a gaming app is the best option.

  ## <ins>Limitations</ins>

 - The dataset is very limited from the point of view of data entries and may be not truthful for the entire population that should be considered.
 - The dataset may be outdated and should be constantly refreshed with new data.

  ## <ins>References</ins>

1. [Youtube](https://www.youtube.com/)
2. [Kaggle.com](https://www.kaggle.com/datasets/reenapinto/google-play-store)

  ## <ins>Notes</ins>

- This project represents my practice work in the field of data analysis. The dataset used may not represent the reality and therefore the findings should be treated in accordance.
- For more details about me and my work, or if you want to get in touch with me, please access my [LinkedIn profile](https://www.linkedin.com/in/alexpraporgescu/).
