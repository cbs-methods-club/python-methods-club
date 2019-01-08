---
title: Wrangling presentation log files with `pandas`
authors:
- drose
tags:
- python
- pandas
- csv
created_at: 2018-11-15 00:00:00
updated_at: 2019-01-08 17:28:27.416830
tldr: "We show how the `pandas` package can be used to parse a presentation log file.\
  \ \n"
---

# Python Methods Club
## 2018-11-15
`pandas`, `DataFrames` and wrangling presentation log files

# Part 1: Pandas Introduction

The first part of the session was based on a page from the pandas documentation: 
[10 minutes to pandas](https://pandas.pydata.org/pandas-docs/stable/10min.html).

I will skip the content here, since it is a redundant and incomplete copy of the original. Instead, I strongly recommend, going through the [tutorials from the pandas documentation](http://pandas.pydata.org/pandas-docs/stable/tutorials.html).

# Part2: Parsing a Presentation log file :-)

We will parse presentation log files to `pandas.DataFrame`s and query them a little.

## First step: Define a filename.


```python
import os  # for path string manipulation

directory = "/data/t_pythonmethodsclub/sessions/2018-11-15-Pandas/"
filename = "Beispiel_2.log"
path = os.path.join(directory, filename)
```
## Second step: Try to parse the file

Let's first look at the file


```python
with open(path, "r") as file:  # the `with` statement ensures, the file will be closed
    # read the first 10 lines
    for i in range(10):
        # line by line
        line = file.readline()
        # print with line number for orientation
        print(f"Line #{i} ", line)
```
Let's now try to do it with `pandas`.


```python
import pandas as pd  # <-- default alias in the community
%matplotlib inline  
# ^ for plotting
```
The pandas function `read_csv` is there to read text files that contain comma (or similarly) separated values. 


```python
df = pd.read_csv(path)
df.head()  # only display first few entries
```
This looks a bit weird. As we could see above, the first two lines contain header information. The dataset starts on line #3. We need to skip the lines before.


```python
df = pd.read_csv(path, header=2)
df.head()
```
Better - the table head now contains all the fields, but everything in one column. Seems like values are not separated by commas, but instead by tabs `\t`.


```python
df = pd.read_csv(path, header=2, sep="\t")
df.head(20)  # let's display 20 rows
```
Great! Now let's find all the events of type "Response".


```python
df[df["Event Type"] == "Response"]
```
and restrict to only the "Time" value. `loc` lets you index by label.


```python
answers = df[df["Event Type"] == "Response"].loc[:,"Time"]
answers
```
We have 92 responses. Next, let's find the time point of the first pulse (important for fMRI timing correction). `iat` lets you index by number.


```python
first_pulse = df[df["Event Type"] == "Pulse"]["Time"].iat[0]
first_pulse
```
Now correct all timings by this value. The `apply` method lets you apply a function on your data.


```python
answers.apply(lambda x: (x - first_pulse)/10000)
```
Let's plot our response times per trial (TTime vs. Time).


```python
df.plot(x="Time", y="TTime")
```
Convert from "wide" to "long format with the `melt` method.


```python
df.melt(id_vars="Time")
```
Filter operations do not affect the original dataframe (unless you overwrite it, of course).


```python
df
```