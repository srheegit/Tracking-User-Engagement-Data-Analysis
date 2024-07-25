# Tracking-User-Engagement

We shall use SQL, Excel, and Python to track user engagement of an education platform.  We will determine whether the new additions to the platform (new courses, exams, and career tracks) have increased student engagement. The data, as well as the background of this data analysis, came from 365datascience.com; all work are my own.

"The first half of 2022 was expected to be profitable for the company. The reason was the hypothesized increased student engagement after the release of several new features on the company’s website at end-2021. These include enrolling in career tracks and testing your knowledge through practice, course, and career track exams. Of course, we have also expanded our course library to increase user engagement and the platform’s audience as more topics are covered. By comparing different metrics, we can measure the effectiveness of these new features and the overall engagement of our users."

Here is the outline of this project:

## 1. Data Preparation with SQL

### I. Calculating a Subscription's End Date

We use the `student_purchases` table from the `data_scientist_project` database to create a result set with the following columns: 
* `purchase_id`
* `student_id`
*  `plan_id`
*  `date_start`
*  `date_end`
*  `date_refunded`

The `date_start column` is the renamed `date_purchased` column from the database, adjusted for consistency with the subsequent date_end column.

To calculate the end date of a subscription (date_end), add one month, three months, or 12 months to the start date of a subscription for a Monthly (represented as `0` in the plan_id column), Quarterly (`1`), or an Annual (`2`) purchase, respectively. The only exception is the lifetime subscription (denoted by `3`), which has no end date. Refunds will be handled in the next step.

### II. Recalculating a Subscription's End Date

We now create a new view that has all columns except the `date_refunded` column from the table/view created from part I, with the `date_end` column recalcuated so that if an order was refunded - indicated by a non-`NULL` value in the `date_refunded` field - the student's subscription terminates at the refund date.

### III. Created Two 'paid' columns and a MySQL View

We use the result from part II to create a new view called `purchases_info` that we'll use in subsequent parts. The view has the following colunns:
* `purchase_id`
* `sutdent_id`
* `plan_id`
* `date_start`
* `date_end`
* `paid_q2_2021`
* `paid_q2_2022`

Here, the `paid_q2_2021` and `paid_q2_2022` columns have binary values showing whether a student had an active subscription during the respecrtive year's second quarter (from April 1st to June 30th, inclusive). A `0` in the column indicates a free plan student in Q2, while a `1` represents an active subscription in that period.

## 2. Date Preparation with SQL - Splitting into Periods

Now that we have the view `purchases_info`, We classify students as free-plan and paying in Q2 of 2021 and Q2 of 2022.

### I. Calculating Total Minutes Watched in Q2 2021 and Q2 2022

We’re now interested in analyzing the engagement of our users in terms of the total minutes watched during Q2 2021 and Q2 2022 separately. Additionally, we want to identify which users were paid subscribers during each of these periods.

We first write a SQL query that returns two columns: `student_id`, which is a list of student IDs, and `minutes_watched`, which is the total minutes a student have watched in both periods.

After running a simple SQL query, we may find that we have 7,639 students who watched a lecture in Q2 2021 and 8,841 students who wathced a lecture in Q2 2022.

### II. Creating a 'paid' Column

From the result of part I, we then create four datasets, according to the year that a student was engaged and according to whether he/she has paid for that year. Each dataset will contain three columns: `student_id`, `minutes_watched`, and `paid_in_q2`.

Each of the resulting four dataset will be saved as a separate CSV file, as follows:

* minutes_watched_2021_paid_0.csv, for students who engaged in Q2 of 2021 but didn't pay in that period
* minutes_watched_2022_paid_0.csv, for students who engaged in Q2 of 2022 but didn't pay in that period
* minutes_watched_2021_paid_1.csv, for students who engaged in Q2 of 2021 and paid in that period
* minutes_watched_2022_paid_1.csv, for students who engaged in Q2 of 2021 and paid in that period

## 3. Data Preparation with SQL - Certificates Issued

We shall now retrieve information on the total number of minutes watched and the certificates issued to a student. Later, we shall use our result to study the correlation between these two metrics.

### I. Studying Minutes Watched and Certificates Issued

We create a SQL query to extract the following information for each student who has been issued a cedrtificate:

* The student ID
* The total number of minutes watched
* The total number of certificates issued.

We furthermore assign `0` as the total number of minutes watched for students who have no minutes recorded. We then save the resulting table as minutes_and_certificates.csv for later use.

By using a simple query, we find that we have a total of 658 rows (i.e. students) with an issued certificate.

## 4. Data Preprocessing with Python - Removing Outliers

Throughout this section, we use Jupyter notebook to run Python.

### I. Plotting the Distributions

Using the four csv files from part 2, we plot the distribution of the `minutes_watched` variable of each of the four datasets and examine its shape. We find that the distributions are right-skewed, telling us that there are several outliers in the right tail end of the distribution, i.e. there are some students who watched lectures a lot more than other students. You may find a more detailed explanation of the preprocessing, analysis, and visualization in the Python file. I have used `pandas`' `kdeplot()` method to create a kernel density estimation.

### II. Removing the Outliers

As the outliers are not representative of the general trends of the data (i.e. students who are exceptionally engaged don't tell us much about the general trend of how much or how little other students have engaged), we remove the outliers. Because the data is right-skewed, I chose to remove the top 1% of the distribution, rather than removing the data of students who are outside of the Median &#177; 1.5 IQR.

After removing the outliers, we store the resulting truncated dataset into four new datasets:
* minutes_watched_2021_paid_0_no_outliers.csv
* minutes_watched_2022_paid_0_no_outliers.csv
* minutes_watched_2021_paid_1_no_outliers.csv
* minutes_watched_2022_paid_1_no_outliers.csv

