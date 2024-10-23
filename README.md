# Ann Arbor Parking Tickets Analysis

## Project Overview

This project analyzes parking ticket data from the city of Ann Arbor obtained through FOIA requests. The data consists of multiple Excel files, each containing information on ticket violations. The primary goal of this project is to load, clean, and analyze this data to gain insights into parking violations, car makes, and license plate distributions.

## Table of Contents
1. [Project Description](#project-description)
2. [Data Sources](#data-sources)
3. [Functionality](#functionality)
    - [Loading Data](#loading-data)
    - [Generating Descriptive Statistics](#generating-descriptive-statistics)
    - [Common Car Make Analysis](#common-car-make-analysis)
    - [License Plate Analysis](#license-plate-analysis)
4. [Setup Instructions](#setup-instructions)
5. [Key Results](#key-results)
6. [Contact Information](#contact-information)

## Project Description

This project involves the following steps:
1. **Loading Data:** Combining multiple sheets from several Excel files into a single pandas DataFrame.
2. **Generating Descriptive Statistics:** Analyzing ticket descriptions to understand their distribution across different times of the day.
3. **Common Car Make Analysis:** Identifying the most common car make among tickets issued to NY plates.
4. **License Plate Analysis:** Categorizing Michigan plates and counting vehicles with vanity plates.

## Data Sources

The data consists of parking ticket records for Ann Arbor from the following Excel files:
- AnnArbor-TicketViolation2015.xls
- AnnArbor-TicketViolation2016.xls
- AnnArbor-TicketViolation2017.xls
- AnnArbor-TicketViolation2018.xls
- AnnArbor-TicketViolation2019.xls
- AnnArbor-TicketViolation-jan2020.xls

## Functionality

### Loading Data

The `load_ticket_data()` function reads and processes multiple sheets from each Excel file, cleans the data by removing headers and footers, and combines them into a single DataFrame.

#### Code:
```python
import pandas as pd
import xlrd
import warnings

warnings.filterwarnings('ignore')

def load_ticket_data():
    def process_file(file):
        data = pd.read_excel(file, sheet_name=None, header=None)
        frames = []
        
        for sheet_name, df in data.items():
            if not df.empty:
                df = df.iloc[4:]  # Adjusting for header rows
                df.columns = ['Ticket #', 'Badge', 'Issue Date', 'IssueTime', 'Plate', 'State', 
                              'Make', 'Model', 'Violation', 'Description', 'Location', 'Meter', 
                              'Fine', 'Penalty']
                
                # Remove footer row in the third sheet if present
                if sheet_name == 'Sheet3':
                    df = df[:-1]
                    
                frames.append(df)

        if frames:
            df_combined = pd.concat(frames, ignore_index=True)
            return df_combined

        return pd.DataFrame()

    files = [
        'C:/Users/akama/Downloads/AnnArbor-TicketViolation2015.xls',
        'C:/Users/akama/Downloads/AnnArbor-TicketViolation2016.xls',
        'C:/Users/akama/Downloads/AnnArbor-TicketViolation2017.xls',
        'C:/Users/akama/Downloads/AnnArbor-TicketViolation2018.xls',
        'C:/Users/akama/Downloads/AnnArbor-TicketViolation2019.xls',
        'C:/Users/akama/Downloads/AnnArbor-TicketViolation-jan2020.xls'
    ]

    all_df = []
    for file in files:
        df = process_file(file)
        if not df.empty:
            all_df.append(df)
        else:
            print(f"No data found in file: {file}")

    if all_df:
        return pd.concat(all_df, ignore_index=True)
    else:
        raise ValueError("No data found in any files.")

# Load the data
df = load_ticket_data()
print(f"DataFrame Shape: {df.shape}")
df.head()
```

### Generating Descriptive Statistics

The `generate_descriptors(df)` function generates a DataFrame of unique ticket descriptions and their frequencies for different times of the day: morning, afternoon, and evening.

#### Code:
```python
def generate_descriptors(df):
    df['IssueTime'] = pd.to_numeric(df['IssueTime'], errors='coerce')
    df = df.dropna(subset=['IssueTime'])
    df['IssueTime'] = df['IssueTime'].astype(int)
    
    def get_period_count(df, start, end):
        return {desc: len(df[(df['Description'] == desc) & (df['IssueTime'] >= start) & (df['IssueTime'] < end)])
                for desc in df['Description'].dropna().unique()}
    
    morning_counts = get_period_count(df, 300, 1200)
    afternoon_counts = get_period_count(df, 1200, 1800)
    evening_counts = get_period_count(df, 1800, 2400) | get_period_count(df, 0, 300)
    
    return pd.DataFrame([morning_counts, afternoon_counts, evening_counts], 
                        index=["Morning", "Afternoon", "Evening"])

# Generate descriptors
df_descriptors = generate_descriptors(df)
print(df_descriptors.shape)
df_descriptors
```

### Common Car Make Analysis

The `common_car_make(df)` function identifies the most common make of car that received tickets from the state of NY.

#### Code:
```python
def common_car_make(df):
    ny_tickets = df[df['State'] == 'NY']
    most_common_make = ny_tickets['Make'].value_counts().idxmax()
    return most_common_make

# Identify the most common car make for NY plates
most_common_make_ny = common_car_make(df)
print(f"The most common car make for NY plates is: {most_common_make_ny}")
```

### License Plate Analysis

The `fine_per_plates(df)` function analyzes the distribution of different types of Michigan license plates that received tickets.

#### Code:
```python
import re

def fine_per_plates(df):
    mi_tickets = df[df['State'] == 'MI']
    plates = mi_tickets['Plate'].dropna()
    
    pattern_abc1234 = re.compile(r'^[A-Z]{3}[0-9]{4}$')
    pattern_abc123 = re.compile(r'^[A-Z]{3}[0-9]{3}$')
    pattern_123abc = re.compile(r'^[0-9]{3}[A-Z]{3}$')
    
    plate_counts = {
        "ABC1234": plates.str.contains(pattern_abc1234).sum(),
        "ABC123": plates.str.contains(pattern_abc123).sum(),
        "123ABC": plates.str.contains(pattern_123abc).sum()
    }
    plate_counts["vanity"] = len(plates) - sum(plate_counts.values())
    return plate_counts

# Analyze Michigan plates
mi_plate_distribution = fine_per_plates(df)
print(mi_plate_distribution)
```

## Key Results

1. **Most Common Car Make for NY Plates:** Identified the most common car make for NY plates.
2. **Distribution of Michigan Plates:** Analyzed the number of vanity plates versus standard formats.
3. **Descriptive Statistics for Ticket Descriptions:** Generated frequency counts for different ticket descriptions at various times of the day.

## Setup Instructions

1. **Clone the Repository**:
    ```sh
    git clone https://github.com/akamal341/your_repository.git
    cd your_repository
    ```

2. **Install Dependencies**:
    ```sh
    pip install pandas xlrd
    ```

3. **Place Data Files**:
    Ensure the data files are placed in the `Downloads` folder or update the file paths accordingly in the `load_ticket_data()` function.

4. **Run the Jupyter Notebook**:
    Open and run the provided Jupyter Notebook to see the data manipulation and analysis.

## Contact Information

For any questions or further information, please contact:
- **Name:** Asad Kamal
- **Email:** aakamal {/@/} umich {/dot/} edu
- **LinkedIn:** [LinkedIn Profile](https://linkedin.com/in/asadakamal)
