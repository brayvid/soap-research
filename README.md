# Soap Data Science

&copy; Copyright 2024-2025 [soap.fyi](https://soap.fyi). All rights reserved.


```python
import os
from dotenv import load_dotenv
import pandas as pd
from sqlalchemy import create_engine, text
import psycopg2

# Load environment variables from .env file
load_dotenv()

# Access the variables
db_host = os.getenv('DB_HOST')
db_port = os.getenv('DB_PORT', '5432')
db_name = os.getenv('DB_NAME')
db_user = os.getenv('DB_USER')
db_pass = os.getenv('DB_PASS')

db_connection_str = None # Initialize
engine = None # Initialize

if not all([db_host, db_name, db_user, db_pass]):
    print("ERROR: Database credentials not fully loaded from .env or environment.")
    print("Please ensure DB_HOST, DB_NAME, DB_USER, and DB_PASS are in your .env file or environment.")
else:
    print("Database credentials loaded successfully.")
    # Construct the SQLAlchemy connection string
    db_connection_str = f'postgresql+psycopg2://{db_user}:{db_pass}@{db_host}:{db_port}/{db_name}'
    try:
        engine = create_engine(db_connection_str)
        
        # Test connection with a simple query
        # Use a context manager for the connection to ensure it's closed
        with engine.connect() as connection:
            # Wrap the SQL string in text() for direct execution
            result = connection.execute(text("SELECT version();"))
            version_row = result.fetchone() # Fetch one row
            if version_row:
                print(f"\nConnection to PostgreSQL successful! Version: {version_row[0]}")
            else:
                print("\nConnection to PostgreSQL successful, but version query returned no result.")
            # The connection is automatically closed when exiting the 'with' block
            
    except Exception as e:
        print(f"\nFailed to create SQLAlchemy engine or connect: {e}")
        engine = None # Ensure engine is None if connection failed
```

    Database credentials loaded successfully.
    
    Connection to PostgreSQL successful! Version: PostgreSQL 16.8 (Debian 16.8-1.pgdg120+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit



```python
if engine:
    sql_query_with_unique_voters = """
    SELECT
        W.word_id,                      
        W.word,
        COUNT(V.vote_id) AS total_votes,
        COUNT(DISTINCT V.user_id) AS unique_voters
    FROM
        words W
    INNER JOIN
        votes V ON W.word_id = V.word_id
    GROUP BY
        W.word_id, W.word  -- Group by ID and text
    ORDER BY
        total_votes DESC; 
    """
    try:
        print("--- Query: All words, total votes, and unique voter counts ---")
        print(sql_query_with_unique_voters)
        df_query_with_unique_voters = pd.read_sql_query(sql_query_with_unique_voters, engine)
        display(df_query_with_unique_voters)
    except Exception as e:
        print(f"Error executing query: {e}")
else:
    print("Database engine not available. Please run the connection cell first.")
```

    --- Query: All words, total votes, and unique voter counts ---
    
        SELECT
            W.word_id,                      
            W.word,
            COUNT(V.vote_id) AS total_votes,
            COUNT(DISTINCT V.user_id) AS unique_voters
        FROM
            words W
        INNER JOIN
            votes V ON W.word_id = V.word_id
        GROUP BY
            W.word_id, W.word  -- Group by ID and text
        ORDER BY
            total_votes DESC; 
        



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>word_id</th>
      <th>word</th>
      <th>total_votes</th>
      <th>unique_voters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4</td>
      <td>ethical</td>
      <td>53</td>
      <td>8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11</td>
      <td>corrupt</td>
      <td>50</td>
      <td>7</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10</td>
      <td>evil</td>
      <td>43</td>
      <td>7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>78</td>
      <td>hateful</td>
      <td>38</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>67</td>
      <td>progressive</td>
      <td>35</td>
      <td>5</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>76</th>
      <td>119</td>
      <td>kind</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>77</th>
      <td>65</td>
      <td>cop</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>78</th>
      <td>145</td>
      <td>asshole</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>79</th>
      <td>143</td>
      <td>socialist</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>80</th>
      <td>61</td>
      <td>bloated</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>81 rows × 4 columns</p>
</div>



```python
if engine:
    sql_query = """
    SELECT
        P.name,
        P.politician_id,
        COUNT(V.vote_id) AS votes
    FROM
        politicians P
    INNER JOIN
        votes V ON P.politician_id = V.politician_id
    GROUP BY
        P.politician_id, P.name
    ORDER BY
        votes DESC;
    """
    try:
        print("--- Query: All politicians, their IDs, and number of submissions ---")
        print(sql_query)
        df_query = pd.read_sql_query(sql_query, engine)
        display(df_query)
    except Exception as e:
        print(f"Error executing query: {e}")
else:
    print("Database engine not available. Please run the connection cell first.")
```

    --- Query: All politicians, their IDs, and number of submissions ---
    
        SELECT
            P.name,
            P.politician_id,
            COUNT(V.vote_id) AS votes
        FROM
            politicians P
        INNER JOIN
            votes V ON P.politician_id = V.politician_id
        GROUP BY
            P.politician_id, P.name
        ORDER BY
            votes DESC;
        



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>politician_id</th>
      <th>votes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Donald Trump</td>
      <td>1</td>
      <td>433</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bernie Sanders</td>
      <td>2</td>
      <td>75</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Cory Booker</td>
      <td>3</td>
      <td>65</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ted Cruz</td>
      <td>5</td>
      <td>49</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alexandria Ocasio-Cortez</td>
      <td>36</td>
      <td>41</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Mitch McConnell</td>
      <td>599</td>
      <td>39</td>
    </tr>
    <tr>
      <th>6</th>
      <td>JD Vance</td>
      <td>591</td>
      <td>37</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Jamie Raskin</td>
      <td>4</td>
      <td>31</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Pete Hegseth</td>
      <td>600</td>
      <td>24</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Eric Adams</td>
      <td>595</td>
      <td>20</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Elizabeth Warren</td>
      <td>597</td>
      <td>15</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Chris Murphy</td>
      <td>601</td>
      <td>14</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Tammy Duckworth</td>
      <td>613</td>
      <td>14</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Lindsay Graham</td>
      <td>614</td>
      <td>9</td>
    </tr>
    <tr>
      <th>14</th>
      <td>John Kennedy</td>
      <td>617</td>
      <td>7</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Tom Cotton</td>
      <td>594</td>
      <td>6</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Jasmine Crockett</td>
      <td>619</td>
      <td>2</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Dick Durbin</td>
      <td>590</td>
      <td>2</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Ron DeSantis</td>
      <td>598</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Joni Ernst</td>
      <td>616</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



```python
if engine:
    # --- Set the target politician ID ---
    target_politician_id = 1
    # --- ---

    sql_query = """
    SELECT
        W.word,
        W.word_id,
        COUNT(V.vote_id) AS votes
    FROM
        words W
    INNER JOIN
        votes V ON W.word_id = V.word_id
    WHERE
        V.politician_id = %(pol_id)s
    GROUP BY
        W.word_id, W.word
    ORDER BY
        votes DESC;
    """
    try:
        print(f"--- Query: Words and their IDs for Politician ID {target_politician_id} ---")
        print(sql_query)
        df_query = pd.read_sql_query(sql_query, engine, params={'pol_id': target_politician_id})
        display(df_query)
    except Exception as e:
        print(f"Error executing query: {e}")
else:
    print("Database engine not available. Please run the connection cell first.")
```

    --- Query: Words and their IDs for Politician ID 1 ---
    
        SELECT
            W.word,
            W.word_id,
            COUNT(V.vote_id) AS votes
        FROM
            words W
        INNER JOIN
            votes V ON W.word_id = V.word_id
        WHERE
            V.politician_id = %(pol_id)s
        GROUP BY
            W.word_id, W.word
        ORDER BY
            votes DESC;
        



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>word</th>
      <th>word_id</th>
      <th>votes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>corrupt</td>
      <td>11</td>
      <td>37</td>
    </tr>
    <tr>
      <th>1</th>
      <td>cruel</td>
      <td>35</td>
      <td>28</td>
    </tr>
    <tr>
      <th>2</th>
      <td>selfish</td>
      <td>14</td>
      <td>27</td>
    </tr>
    <tr>
      <th>3</th>
      <td>evil</td>
      <td>10</td>
      <td>23</td>
    </tr>
    <tr>
      <th>4</th>
      <td>greedy</td>
      <td>110</td>
      <td>23</td>
    </tr>
    <tr>
      <th>5</th>
      <td>narcissistic</td>
      <td>17</td>
      <td>21</td>
    </tr>
    <tr>
      <th>6</th>
      <td>traitor</td>
      <td>142</td>
      <td>20</td>
    </tr>
    <tr>
      <th>7</th>
      <td>insane</td>
      <td>9</td>
      <td>19</td>
    </tr>
    <tr>
      <th>8</th>
      <td>russian</td>
      <td>96</td>
      <td>17</td>
    </tr>
    <tr>
      <th>9</th>
      <td>dangerous</td>
      <td>105</td>
      <td>17</td>
    </tr>
    <tr>
      <th>10</th>
      <td>dictator</td>
      <td>8</td>
      <td>16</td>
    </tr>
    <tr>
      <th>11</th>
      <td>seditious</td>
      <td>130</td>
      <td>15</td>
    </tr>
    <tr>
      <th>12</th>
      <td>sociopath</td>
      <td>129</td>
      <td>14</td>
    </tr>
    <tr>
      <th>13</th>
      <td>genius</td>
      <td>51</td>
      <td>14</td>
    </tr>
    <tr>
      <th>14</th>
      <td>nazi</td>
      <td>5</td>
      <td>12</td>
    </tr>
    <tr>
      <th>15</th>
      <td>scary</td>
      <td>95</td>
      <td>12</td>
    </tr>
    <tr>
      <th>16</th>
      <td>hateful</td>
      <td>78</td>
      <td>10</td>
    </tr>
    <tr>
      <th>17</th>
      <td>intransigent</td>
      <td>3</td>
      <td>10</td>
    </tr>
    <tr>
      <th>18</th>
      <td>disgusting</td>
      <td>104</td>
      <td>9</td>
    </tr>
    <tr>
      <th>19</th>
      <td>unethical</td>
      <td>106</td>
      <td>9</td>
    </tr>
    <tr>
      <th>20</th>
      <td>malevolent</td>
      <td>77</td>
      <td>8</td>
    </tr>
    <tr>
      <th>21</th>
      <td>insidious</td>
      <td>122</td>
      <td>8</td>
    </tr>
    <tr>
      <th>22</th>
      <td>kleptocrat</td>
      <td>139</td>
      <td>7</td>
    </tr>
    <tr>
      <th>23</th>
      <td>cold-blooded</td>
      <td>126</td>
      <td>7</td>
    </tr>
    <tr>
      <th>24</th>
      <td>chaotic</td>
      <td>79</td>
      <td>6</td>
    </tr>
    <tr>
      <th>25</th>
      <td>lucky</td>
      <td>63</td>
      <td>5</td>
    </tr>
    <tr>
      <th>26</th>
      <td>orange</td>
      <td>92</td>
      <td>4</td>
    </tr>
    <tr>
      <th>27</th>
      <td>borders</td>
      <td>109</td>
      <td>4</td>
    </tr>
    <tr>
      <th>28</th>
      <td>scumbag</td>
      <td>127</td>
      <td>4</td>
    </tr>
    <tr>
      <th>29</th>
      <td>mobility</td>
      <td>107</td>
      <td>4</td>
    </tr>
    <tr>
      <th>30</th>
      <td>unintelligent</td>
      <td>54</td>
      <td>4</td>
    </tr>
    <tr>
      <th>31</th>
      <td>autocrat</td>
      <td>140</td>
      <td>3</td>
    </tr>
    <tr>
      <th>32</th>
      <td>clown</td>
      <td>52</td>
      <td>3</td>
    </tr>
    <tr>
      <th>33</th>
      <td>deranged</td>
      <td>121</td>
      <td>3</td>
    </tr>
    <tr>
      <th>34</th>
      <td>orwellian</td>
      <td>74</td>
      <td>3</td>
    </tr>
    <tr>
      <th>35</th>
      <td>impressive</td>
      <td>15</td>
      <td>2</td>
    </tr>
    <tr>
      <th>36</th>
      <td>sloppy</td>
      <td>62</td>
      <td>2</td>
    </tr>
    <tr>
      <th>37</th>
      <td>bloated</td>
      <td>61</td>
      <td>1</td>
    </tr>
    <tr>
      <th>38</th>
      <td>asshole</td>
      <td>145</td>
      <td>1</td>
    </tr>
    <tr>
      <th>39</th>
      <td>ergonomic</td>
      <td>108</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



```python
if engine:
    sql_query = """
    SELECT
        W.word,
        COUNT(V.vote_id) AS votes,
        COUNT(DISTINCT V.politician_id) AS politicians
    FROM
        words W
    INNER JOIN
        votes V ON W.word_id = V.word_id
    GROUP BY
        W.word_id, W.word
    HAVING
        COUNT(DISTINCT V.user_id) = 1  -- Filter for words with exactly one unique voter
    ORDER BY
        votes DESC;
    """
    try:
        print("--- Query: Words with one unique voter ---")
        print(sql_query)
        df_query = pd.read_sql_query(sql_query, engine)
        if not df_query.empty:
            display(df_query)
        else:
            print("No words found with only one unique voter.")
    except Exception as e:
        print(f"Error executing query: {e}")
else:
    print("Database engine not available. Please run the connection cell first.")
```

    --- Query: Words with one unique voter ---
    
        SELECT
            W.word,
            COUNT(V.vote_id) AS votes,
            COUNT(DISTINCT V.politician_id) AS politicians
        FROM
            words W
        INNER JOIN
            votes V ON W.word_id = V.word_id
        GROUP BY
            W.word_id, W.word
        HAVING
            COUNT(DISTINCT V.user_id) = 1  -- Filter for words with exactly one unique voter
        ORDER BY
            votes DESC;
        



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>word</th>
      <th>votes</th>
      <th>politicians</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>clown</td>
      <td>28</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>seditious</td>
      <td>15</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>courageous</td>
      <td>10</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>insidious</td>
      <td>8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>kleptocrat</td>
      <td>7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>cold-blooded</td>
      <td>7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>loquacious</td>
      <td>7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>useless</td>
      <td>6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>honorable</td>
      <td>6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>heartless</td>
      <td>6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>lucky</td>
      <td>5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>11</th>
      <td>scumbag</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>mobility</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>borders</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14</th>
      <td>deranged</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>principled</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>16</th>
      <td>autocrat</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>backward</td>
      <td>3</td>
      <td>2</td>
    </tr>
    <tr>
      <th>18</th>
      <td>eloquent</td>
      <td>3</td>
      <td>2</td>
    </tr>
    <tr>
      <th>19</th>
      <td>reasonable</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>20</th>
      <td>asshole</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>21</th>
      <td>retiring</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>22</th>
      <td>bloated</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>23</th>
      <td>cop</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>24</th>
      <td>sycophant</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>25</th>
      <td>silly</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>sharp</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>27</th>
      <td>ergonomic</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>28</th>
      <td>kind</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>29</th>
      <td>what</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>30</th>
      <td>legend</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>31</th>
      <td>awesome</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>32</th>
      <td>socialist</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>33</th>
      <td>hardworking</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



```python
if engine:
    sql_query = """
    SELECT
        W.word,
        COUNT(V.vote_id) AS votes,
        COUNT(DISTINCT V.politician_id) AS politicians,
        COUNT(DISTINCT V.user_id) AS submitters
    FROM
        words W
    INNER JOIN
        votes V ON W.word_id = V.word_id
    GROUP BY
        W.word_id, W.word
    HAVING
        COUNT(DISTINCT V.user_id) >= 2  -- Filter for words with at least two unique submitters
    ORDER BY
        submitters DESC, votes DESC;
    """
    try:
        print("--- Query: Words with >= 2 unique submitters ---")
        print(sql_query)
        df_query = pd.read_sql_query(sql_query, engine)
        if not df_query.empty:
            display(df_query)
        else:
            print("No words found with at least two unique submitters.")
    except Exception as e:
        print(f"Error executing query: {e}")
else:
    print("Database engine not available. Please run the connection cell first.")
```

    --- Query: Words with >= 2 unique submitters ---
    
        SELECT
            W.word,
            COUNT(V.vote_id) AS votes,
            COUNT(DISTINCT V.politician_id) AS politicians,
            COUNT(DISTINCT V.user_id) AS submitters
        FROM
            words W
        INNER JOIN
            votes V ON W.word_id = V.word_id
        GROUP BY
            W.word_id, W.word
        HAVING
            COUNT(DISTINCT V.user_id) >= 2  -- Filter for words with at least two unique submitters
        ORDER BY
            submitters DESC, votes DESC;
        



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>word</th>
      <th>votes</th>
      <th>politicians</th>
      <th>submitters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ethical</td>
      <td>53</td>
      <td>7</td>
      <td>8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>narcissistic</td>
      <td>21</td>
      <td>1</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>corrupt</td>
      <td>50</td>
      <td>3</td>
      <td>7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>evil</td>
      <td>43</td>
      <td>4</td>
      <td>7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>wise</td>
      <td>17</td>
      <td>2</td>
      <td>7</td>
    </tr>
    <tr>
      <th>5</th>
      <td>hateful</td>
      <td>38</td>
      <td>5</td>
      <td>6</td>
    </tr>
    <tr>
      <th>6</th>
      <td>humane</td>
      <td>24</td>
      <td>2</td>
      <td>6</td>
    </tr>
    <tr>
      <th>7</th>
      <td>patriotic</td>
      <td>17</td>
      <td>3</td>
      <td>6</td>
    </tr>
    <tr>
      <th>8</th>
      <td>dictator</td>
      <td>16</td>
      <td>1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>9</th>
      <td>progressive</td>
      <td>35</td>
      <td>6</td>
      <td>5</td>
    </tr>
    <tr>
      <th>10</th>
      <td>inspiring</td>
      <td>30</td>
      <td>3</td>
      <td>5</td>
    </tr>
    <tr>
      <th>11</th>
      <td>cruel</td>
      <td>28</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>12</th>
      <td>selfish</td>
      <td>27</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>13</th>
      <td>stupid</td>
      <td>13</td>
      <td>3</td>
      <td>5</td>
    </tr>
    <tr>
      <th>14</th>
      <td>scary</td>
      <td>12</td>
      <td>1</td>
      <td>5</td>
    </tr>
    <tr>
      <th>15</th>
      <td>insane</td>
      <td>19</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>16</th>
      <td>genius</td>
      <td>14</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>17</th>
      <td>cretin</td>
      <td>9</td>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>18</th>
      <td>malevolent</td>
      <td>8</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>19</th>
      <td>chaotic</td>
      <td>6</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>20</th>
      <td>constitutional</td>
      <td>6</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>21</th>
      <td>incompetent</td>
      <td>21</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <th>22</th>
      <td>dangerous</td>
      <td>17</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>23</th>
      <td>sociopath</td>
      <td>14</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>24</th>
      <td>intransigent</td>
      <td>10</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>25</th>
      <td>smart</td>
      <td>8</td>
      <td>5</td>
      <td>3</td>
    </tr>
    <tr>
      <th>26</th>
      <td>brave</td>
      <td>4</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>27</th>
      <td>moral</td>
      <td>4</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>28</th>
      <td>orwellian</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
    </tr>
    <tr>
      <th>29</th>
      <td>greedy</td>
      <td>23</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>30</th>
      <td>traitor</td>
      <td>20</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>31</th>
      <td>russian</td>
      <td>17</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>32</th>
      <td>soulless</td>
      <td>17</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>33</th>
      <td>nazi</td>
      <td>12</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>34</th>
      <td>impressive</td>
      <td>12</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>35</th>
      <td>competent</td>
      <td>10</td>
      <td>4</td>
      <td>2</td>
    </tr>
    <tr>
      <th>36</th>
      <td>unethical</td>
      <td>9</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>37</th>
      <td>disgusting</td>
      <td>9</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>38</th>
      <td>ineffectual</td>
      <td>7</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>39</th>
      <td>dumb</td>
      <td>6</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>40</th>
      <td>stooge</td>
      <td>6</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>41</th>
      <td>decrepit</td>
      <td>5</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>42</th>
      <td>orange</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>43</th>
      <td>unintelligent</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>44</th>
      <td>fraudulent</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>45</th>
      <td>compassionate</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>46</th>
      <td>sloppy</td>
      <td>2</td>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



