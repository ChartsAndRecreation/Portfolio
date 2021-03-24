---
date: "2021-03-23"
diagram: true
highlight: true
image:
  caption: 'Image credit: [**Shahadat Rahman**](https://unsplash.com/photos/BfrQnKBulYQ)'
  placement: 3
math: true
title: Import a CSV File to SQL Server with Python
tags:
- Python
- SQL
---

I get asked periodically to import data from a spreadsheet into SQL Server. This is a good option for archiving data you've accumulated over time that may not be used on a daily basis, but comes in handy for that quarterly report. It also reduces the likelihood of the data getting deleted or misplaced when someone new takes over.

In this guide, I'll show how to take a csv file and import it into SQL Server using the [Pandas](https://pypi.org/project/pandas/) and [Pyodbc](https://pypi.org/project/pyodbc/) libraries in Python.

# Step 1: The CSV File

Let's start with the CSV file we want to import. In the spirit of March Madness, my table consists the teams form the Big Ten conference:

| School     | Region   | Seed |
| ---------- | -------- | ---: |
| Illinois   | Midwest  |    1 |
| Iowa       | West     |    2 |
| Maryland   | East     |   10 |
| Michigan   | East     |    1 |
| Ohio State | South    |    2 |
| Purdue     | South    |    4 |
| Rutgers    | Midwest  |   10 |
| Wisconsin  | South    |    9 |

The name of my file is `BigTen.csv` and it's located in `C:\Users\kaleb\Desktop`.

# Step 2: Import the CSV File into a Data Frame

Next, we're going to import the CSV file into data frame in Python using the `pandas` library.

```python
import pandas as pd
import pyodbc
data = pd.read_csv(r'C:\Users\king\Desktop\BigTen.csv')   
df = pd.DataFrame(data, columns= ['School','Region','Seed'])
```

# Step 3: Connect to the SQL Server

We can connect Python to our SQL Server using the `pyodbc` library. In order to do this, we need to know the name of the server and database. In my case, the server is named `My-SQL` and the database is `TestDB`.

```python
conn = pyodbc.connect('Driver={SQL Server};'
                      'Server=My-SQL;'
                      'Database=TestDB;'
                      'Trusted_Connection=yes;')
cursor = conn.cursor()
```

# Step 4: Create the SQL table

Now that we're connected to SQL in Python, we can create a table to store the data from the CSV file. I've called my table `march_madness`.
```python
cursor.execute('''
  CREATE TABLE march_madness (
    School NVARCHAR(50),
    Region NVARCHAR(50),
    Seed INT
  );
  ''')
```

{{% callout note %}}
You can only run this section of code once. If you try to run it multiple times, you'll get an error because the table was already created on the first run.
{{% /callout %}}

# Step 5: Insert the Data Frame into the SQL Table

Our last step will be to import the data frame from Python into the SQL table we created in the previous step.

```python
for row in df.itertuples():
  cursor.execute('''
    INSERT INTO TestDB.dbo.march_madness (School, Region, Seed)
    VALUES (?, ?, ?)
    ''',
    row.school,
    row.Region,
    row.Seed
  )
conn.commit()
```