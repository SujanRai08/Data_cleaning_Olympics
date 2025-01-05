# Olypics Datasets

This repository contains a compreshensive datasets oon oolympic althelets & their result between 1896-2022 ( will be updated with 2024 results)

# Dataset information & Collection
The data was sourced from olympedia.org through web scraping using the Python Beautiful Soup library (refer to `scrape_data.py`).

- `athletes/bios.csv` contains the raw biographical data for each athlete.
- `results/results.csv` holds a detailed record of each event that athletes participated in, along with their results for those events.

It's important to note that temporary CSV files were generated during the scraping process to track progress, but these checkpoint files have since been removed from the repository for simplicity.

# Cleaning Data
*1. Reading CSV and Data Copying*
```python
bios = pd.read_csv('./athelets/bios.csv')
df = bios.copy()
```
- bios = pd.read_csv('./athelets/bios.csv'): Reads the CSV file 'bios.csv' into a Pandas DataFrame bios. The data contains biographical information about athletes.

- df = bios.copy(): Creates a copy of the bios DataFrame to avoid modifying the original data. We will perform transformations on df.


*2. Removing • from the used name columns*
```python
df['name'] = df['Used name'].str.replace('•',' ')
```
- This creates a new column name in the DataFrame df by cleaning up the Used name column. It removes the characters • by replacing it with a space.

- df['Used name'].str.replace('•',' '): Replaces • with a space in the Used name column for better readability.

*3. Correcting Units for height_cm
python
Copy code
*
```python
def correct_units(values):
    if isinstance(values, str) and 'kg' in values:
        return values.replace('kg', 'cm')
    return values

df['height_cm'] = df['height_cm'].apply(correct_units)
```
- correct_units: This function checks if the input value contains the string 'kg' (which is an incorrect unit for height) and replaces 'kg' with 'cm'.
 
- df['height_cm'].apply(correct_units): Applies the correct_units function to each element of the height_cm column.
  
*4. Processing Ranges in height_cm (e.g., '50-55')*
``` python
import re

def process_ranges(value):
    if isinstance(value, str) and '-' in value:
        # Extract numbers, calculate average, and append 'cm'
        numbers = list(map(int, re.findall(r'\d+', value)))
        return f'{sum(numbers) / len(numbers):.1f} cm'
    return value

df['height_cm'] = df['height_cm'].apply(process_ranges)
```
- process_ranges: This function checks if the height value contains a range (i.e., a hyphen -). If so, it extracts the numbers from the range, calculates the average, and returns the result as a string with 'cm' added at the end.

- re.findall(r'\d+', value): Extracts all numbers from the string value using regular expressions.

- df['height_cm'].apply(process_ranges): Applies the process_ranges function to each value in the height_cm column.

*5. Cleaning height_cm and Handling Multiple Formats*
```python
def clean_height(value):
    if pd.isna(value):  # Handle NaN values
        return None
    value = value.strip()  # Remove extra spaces
    if 'kg' in value:
        value = value.replace('kg', 'cm')
    if ',' in value:
        numbers = list(map(int, re.findall(r'\d+', value)))
        average = sum(numbers) / len(numbers)
        return f'{average:.1f}cm'
    elif '-' in value:
        numbers = list(map(int, re.findall(r'\d+', value)))
        average = sum(numbers) / len(numbers)
        return f"{average:.1f} cm"
    elif isinstance(value, str) and re.search(r'\d+cm', value):  # Ensure space before 'cm'
        return add_space_to_unit(value)
    elif re.search(r'\d+ cm', value):
        return value
    else:
        return None
```
- clean_height: This function is responsible for cleaning and standardizing the height values in the height_cm column. It handles multiple cases:
- pd.isna(value): Checks if the value is NaN. If it is, the function returns None.
- value.strip(): Removes any leading or trailing spaces from the value.
- 'kg' in value: If the value contains 'kg', it replaces it with 'cm' since we want height in centimeters.
- Handling comma-separated values: If the value contains a comma (e.g., '70, 72'), it extracts the numbers, calculates their average, and appends 'cm'.
- Handling hyphen-separated ranges: If the value contains a hyphen (e.g., '70-72'), it extracts the numbers, calculates the average, and returns the result with 'cm'.
- Ensuring space before cm: If the value ends with 'cm', it checks and ensures there’s a space before 'cm'.
- If none of the conditions match, the function returns None.
