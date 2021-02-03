# Library Data Analysis: Part 2

Author: Andrea Mock

After having explored the library books dataset, we can look at another dataset that contains information about the most popular books that were borrowed each week throughout 2020. 

The notebook is broken down into the following sections: <br/>
[1. Importing libraries and data](#intro)<br/>
[2. Examining temporal trends in borrowing](#borrow_hist)<br/>
[3. Merging datasets](#merge)<br/>
[4. Hypothesis testing](#ttest)<br/>
[5. Suggestions to the library administrators](#feedback)<br/>

<a id="intro"></a>
## 1. Importing data and libraries
To begin we import the two datasets that are part of this project: 
- the overview of all books in our library
- the borrowing trends for the top books in 2020


```python
from collections import Counter
import json
import pandas as pd

%matplotlib inline
import matplotlib.pyplot as plt
plt.style.use('ggplot')
```


```python
# import dataset that contains all library books info 
books_df = pd.read_csv('all_books.csv', index_col=0)
books_df.head()
```




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
      <th>isbn</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0195153448</td>
      <td>Classical Mythology</td>
      <td>Mark P. O. Morford</td>
      <td>2002</td>
      <td>Oxford University Press</td>
      <td>http://images.amazon.com/images/P/0195153448.0...</td>
      <td>http://images.amazon.com/images/P/0195153448.0...</td>
      <td>http://images.amazon.com/images/P/0195153448.0...</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0002005018</td>
      <td>Clara Callan</td>
      <td>Richard Bruce Wright</td>
      <td>2001</td>
      <td>HarperFlamingo Canada</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0060973129</td>
      <td>Decision in Normandy</td>
      <td>Carlo D'Este</td>
      <td>1991</td>
      <td>HarperPerennial</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>9</td>
      <td>7</td>
      <td>0.777778</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0374157065</td>
      <td>Flu: The Story of the Great Influenza Pandemic...</td>
      <td>Gina Bari Kolata</td>
      <td>1999</td>
      <td>Farrar Straus Giroux</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0393045218</td>
      <td>The Mummies of Urumchi</td>
      <td>E. J. W. Barber</td>
      <td>1999</td>
      <td>W. W. Norton &amp;amp; Company</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>7</td>
      <td>4</td>
      <td>0.571429</td>
      <td>1990</td>
    </tr>
  </tbody>
</table>
</div>




```python
# create a dataframe that contains historical book data  
topBooks = pd.read_json('cscl_2020_top_hundred_books.json',lines=True)
topBooks.head()
```




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
      <th>week</th>
      <th>isbn</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>195153448</td>
      <td>93</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>887841740</td>
      <td>100</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>971880107</td>
      <td>25</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>375759778</td>
      <td>21</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>3404921038</td>
      <td>88</td>
    </tr>
  </tbody>
</table>
</div>




```python
# convert values to ints from string
topBooks['count'] = topBooks['count'].apply(lambda x:int(x))
topBooks['week'] = topBooks['week'].apply(lambda x:int(x))
```

<a id="borrow_hist"></a>
## 2. Examining lending history 
Our topBooks dataframe contains the isbn of a particular book as well as the number of times that book title was borrowed in each week. Since our dataset contains more than 5000 entries we first want to see how many unique books (isbn) are in the dataset. As we can see there are 106 unique books that appear in the topBooks dataframe, meaning that for 106 books we have borrowing history for a whole year. 


```python
# check out distribution of titles included each week 
topBooks.groupby(by=["week"])['isbn'].count().unique()
```




    array([106])



### 2.1 Borrowing frequency by book 
By grouping the books by isbn we can create a dataframe that includes the total number of times a book was borrowed throughout the year. In addition we can focus on the top 10 books and create a graphical representation of their borrowing history throughout the year.


```python
# create dataframe that counts total number of times a book was borrowed
bookFreq = pd.DataFrame(topBooks.groupby(by=["isbn"])['count'].sum())
bookFreq = bookFreq.reset_index().sort_values('count', ascending=False).reset_index(drop=True)
```


```python
# example of books and number of times they were borrowed
bookFreq.head()
```




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
      <th>isbn</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>345465083</td>
      <td>3440</td>
    </tr>
    <tr>
      <th>1</th>
      <td>618119760</td>
      <td>3419</td>
    </tr>
    <tr>
      <th>2</th>
      <td>446612545</td>
      <td>3270</td>
    </tr>
    <tr>
      <th>3</th>
      <td>385509456</td>
      <td>3238</td>
    </tr>
    <tr>
      <th>4</th>
      <td>385495145</td>
      <td>3221</td>
    </tr>
  </tbody>
</table>
</div>



Now we can extract the top books and filter to only include the temporal data for those ten books.


```python
# grab the top 10 books checked out in 2020
top10 = bookFreq.head(10)
top10
```




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
      <th>isbn</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>345465083</td>
      <td>3440</td>
    </tr>
    <tr>
      <th>1</th>
      <td>618119760</td>
      <td>3419</td>
    </tr>
    <tr>
      <th>2</th>
      <td>446612545</td>
      <td>3270</td>
    </tr>
    <tr>
      <th>3</th>
      <td>385509456</td>
      <td>3238</td>
    </tr>
    <tr>
      <th>4</th>
      <td>385495145</td>
      <td>3221</td>
    </tr>
    <tr>
      <th>5</th>
      <td>425182908</td>
      <td>3219</td>
    </tr>
    <tr>
      <th>6</th>
      <td>312995423</td>
      <td>3192</td>
    </tr>
    <tr>
      <th>7</th>
      <td>042518630X</td>
      <td>3188</td>
    </tr>
    <tr>
      <th>8</th>
      <td>749748001</td>
      <td>3187</td>
    </tr>
    <tr>
      <th>9</th>
      <td>743470389</td>
      <td>3176</td>
    </tr>
  </tbody>
</table>
</div>




```python
# gather the isbn of the top 10 books to use as filter
topIsbns = top10['isbn'].to_list()
topIsbns
```




    ['345465083',
     '618119760',
     '446612545',
     '385509456',
     '385495145',
     '425182908',
     '312995423',
     '042518630X',
     '749748001',
     '743470389']




```python
def extractTopBooks(entry, isbnList):
    """
    returns true if a book entry's isbn is in a list of isbn, otherwise false
    """
    return (entry['isbn'] in isbnList)
```


```python
# filtered dataframe with only information about top 10 books
top10Info = topBooks[topBooks.apply(lambda x: extractTopBooks(x, topIsbns), axis=1)]
top10Info.head()
```




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
      <th>week</th>
      <th>isbn</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6</th>
      <td>1</td>
      <td>425182908</td>
      <td>21</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1</td>
      <td>042518630X</td>
      <td>36</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>345465083</td>
      <td>38</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1</td>
      <td>446612545</td>
      <td>18</td>
    </tr>
    <tr>
      <th>44</th>
      <td>1</td>
      <td>385509456</td>
      <td>43</td>
    </tr>
  </tbody>
</table>
</div>




```python
# visualize borrowing trends for top 10 books
import plotly.express as px
fig = px.area(top10Info, x="week", y="count", color="isbn", line_group="isbn", title= "borrowing trends for top 10 books")
fig.show()
```


<div>


            <div id="d9fde76a-e70a-4258-b32f-a93209e7101c" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("d9fde76a-e70a-4258-b32f-a93209e7101c")) {
                    Plotly.newPlot(
                        'd9fde76a-e70a-4258-b32f-a93209e7101c',
                        [{"hovertemplate": "isbn=425182908<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "425182908", "line": {"color": "#636efa"}, "mode": "lines", "name": "425182908", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [21, 52, 39, 61, 79, 56, 14, 74, 85, 92, 28, 63, 74, 78, 40, 76, 73, 88, 57, 100, 46, 100, 55, 89, 40, 47, 22, 78, 51, 92, 28, 77, 72, 78, 39, 24, 50, 88, 25, 96, 83, 76, 60, 85, 100, 90, 31, 82, 40, 16, 66, 43], "yaxis": "y"}, {"hovertemplate": "isbn=042518630X<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "042518630X", "line": {"color": "#EF553B"}, "mode": "lines", "name": "042518630X", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [36, 31, 78, 30, 10, 43, 27, 60, 62, 99, 38, 44, 37, 54, 100, 99, 53, 46, 49, 54, 79, 53, 90, 12, 90, 62, 40, 89, 62, 61, 92, 64, 61, 100, 30, 67, 93, 93, 53, 63, 77, 95, 29, 43, 51, 61, 93, 32, 67, 97, 75, 64], "yaxis": "y"}, {"hovertemplate": "isbn=345465083<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "345465083", "line": {"color": "#00cc96"}, "mode": "lines", "name": "345465083", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [38, 64, 39, 45, 100, 39, 75, 65, 43, 83, 75, 64, 56, 84, 58, 61, 76, 61, 98, 11, 92, 77, 58, 49, 72, 82, 38, 48, 25, 84, 95, 17, 35, 100, 91, 88, 101, 98, 86, 83, 93, 28, 93, 93, 30, 75, 54, 60, 97, 26, 44, 93], "yaxis": "y"}, {"hovertemplate": "isbn=446612545<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "446612545", "line": {"color": "#ab63fa"}, "mode": "lines", "name": "446612545", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [18, 40, 84, 59, 38, 88, 16, 90, 29, 28, 91, 29, 91, 32, 58, 77, 95, 95, 18, 73, 96, 55, 94, 32, 70, 49, 55, 66, 72, 29, 50, 45, 77, 100, 93, 76, 86, 62, 18, 36, 95, 98, 12, 99, 89, 100, 29, 94, 39, 67, 76, 62], "yaxis": "y"}, {"hovertemplate": "isbn=385509456<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "385509456", "line": {"color": "#FFA15A"}, "mode": "lines", "name": "385509456", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [43, 10, 79, 72, 29, 49, 54, 62, 53, 76, 47, 69, 26, 91, 27, 87, 65, 48, 89, 86, 65, 25, 96, 81, 75, 43, 91, 28, 80, 93, 30, 85, 87, 29, 48, 100, 17, 88, 68, 41, 96, 30, 96, 81, 69, 72, 101, 94, 61, 26, 25, 55], "yaxis": "y"}, {"hovertemplate": "isbn=749748001<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "749748001", "line": {"color": "#19d3f3"}, "mode": "lines", "name": "749748001", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [74, 79, 31, 13, 71, 36, 48, 48, 84, 79, 67, 82, 76, 29, 58, 61, 33, 101, 66, 65, 50, 75, 92, 29, 69, 99, 67, 57, 31, 30, 55, 91, 67, 31, 72, 43, 54, 77, 77, 71, 81, 50, 33, 93, 41, 43, 66, 57, 61, 79, 54, 91], "yaxis": "y"}, {"hovertemplate": "isbn=312995423<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "312995423", "line": {"color": "#FF6692"}, "mode": "lines", "name": "312995423", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [36, 49, 99, 99, 98, 34, 99, 45, 29, 78, 57, 51, 27, 101, 85, 60, 66, 93, 84, 101, 79, 52, 79, 89, 25, 13, 71, 17, 13, 86, 47, 73, 28, 86, 77, 29, 38, 61, 93, 37, 64, 87, 31, 66, 29, 67, 62, 32, 84, 49, 59, 78], "yaxis": "y"}, {"hovertemplate": "isbn=743470389<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "743470389", "line": {"color": "#B6E880"}, "mode": "lines", "name": "743470389", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [17, 83, 89, 69, 62, 21, 36, 76, 12, 37, 30, 62, 97, 42, 101, 38, 71, 72, 80, 96, 32, 95, 91, 83, 72, 83, 84, 55, 42, 45, 18, 42, 54, 100, 20, 100, 67, 61, 51, 64, 80, 98, 101, 101, 12, 67, 10, 101, 19, 71, 50, 16], "yaxis": "y"}, {"hovertemplate": "isbn=385495145<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "385495145", "line": {"color": "#FF97FF"}, "mode": "lines", "name": "385495145", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [91, 14, 41, 53, 21, 67, 84, 33, 101, 93, 64, 43, 85, 76, 17, 49, 82, 95, 59, 93, 79, 79, 97, 98, 57, 25, 11, 91, 97, 41, 95, 54, 25, 100, 49, 15, 54, 98, 73, 27, 98, 17, 94, 56, 97, 49, 66, 64, 65, 23, 56, 10], "yaxis": "y"}, {"hovertemplate": "isbn=618119760<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "618119760", "line": {"color": "#FECB52"}, "mode": "lines", "name": "618119760", "orientation": "v", "showlegend": true, "stackgroup": "1", "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [59, 77, 25, 53, 11, 98, 74, 85, 12, 42, 55, 65, 81, 94, 41, 93, 76, 51, 26, 100, 38, 79, 63, 95, 40, 75, 83, 73, 76, 17, 75, 55, 77, 73, 59, 95, 94, 96, 88, 76, 91, 55, 86, 94, 18, 71, 45, 52, 99, 45, 23, 95], "yaxis": "y"}],
                        {"legend": {"title": {"text": "isbn"}, "tracegroupgap": 0}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "borrowing trends for top 10 books"}, "xaxis": {"anchor": "y", "domain": [0.0, 1.0], "title": {"text": "week"}}, "yaxis": {"anchor": "x", "domain": [0.0, 1.0], "title": {"text": "count"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('d9fde76a-e70a-4258-b32f-a93209e7101c');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>



```python
# borrowing trends for one book, here the book with most total borrowings
fig = px.line(top10Info[top10Info['isbn'] == '425182908'], x="week", y="count", color="isbn")
fig.show()
```


<div>


            <div id="313362e4-0760-4dc2-a1d0-5fa2b3a3f494" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("313362e4-0760-4dc2-a1d0-5fa2b3a3f494")) {
                    Plotly.newPlot(
                        '313362e4-0760-4dc2-a1d0-5fa2b3a3f494',
                        [{"hovertemplate": "isbn=425182908<br>week=%{x}<br>count=%{y}<extra></extra>", "legendgroup": "425182908", "line": {"color": "#636efa", "dash": "solid"}, "mode": "lines", "name": "425182908", "orientation": "v", "showlegend": true, "type": "scatter", "x": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52], "xaxis": "x", "y": [21, 52, 39, 61, 79, 56, 14, 74, 85, 92, 28, 63, 74, 78, 40, 76, 73, 88, 57, 100, 46, 100, 55, 89, 40, 47, 22, 78, 51, 92, 28, 77, 72, 78, 39, 24, 50, 88, 25, 96, 83, 76, 60, 85, 100, 90, 31, 82, 40, 16, 66, 43], "yaxis": "y"}],
                        {"legend": {"title": {"text": "isbn"}, "tracegroupgap": 0}, "margin": {"t": 60}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "xaxis": {"anchor": "y", "domain": [0.0, 1.0], "title": {"text": "week"}}, "yaxis": {"anchor": "x", "domain": [0.0, 1.0], "title": {"text": "count"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('313362e4-0760-4dc2-a1d0-5fa2b3a3f494');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


### 2.2 Borrowing frequency by week
Next we can also look at the number of books borrowed per week:


```python
# extract number of books borrowed each week
weekOverview = topBooks.groupby(by=["week"])['count'].sum()
weekOverview[:3]
```




    week
    1    6151
    2    5838
    3    5727
    Name: count, dtype: int64




```python
# summary statistics of borrowings per week
weekOverview.describe()
```




    count      52.000000
    mean     5964.403846
    std       224.758542
    min      5484.000000
    25%      5830.500000
    50%      5969.500000
    75%      6105.750000
    max      6424.000000
    Name: count, dtype: float64




```python
# overview of books borrowed per week for top books
ax = weekOverview.plot(title='Overview of books borrowed per week in 2020')
ax.set_ylabel("number of books")
```




    Text(0, 0.5, 'number of books')




![png](output_21_1.png)


<a id='merge'></a>
## 3. Merging datasets 
Although our dataset with the information on the books that have been borrowed the most we only have information about their isbn and not much more. By merging the temporal borrowing information with the dataset containing information on all books in our library we can obtain a more complete picture of what books we are dealing with. 


```python
def getUniqueIsbns(df):
    """
    returns a list of unique isbns contained in the given dataframe 
    """
    isbns = df['isbn'].unique()
    isbnList = []
    for isbn in isbns:
        if len(isbn) < 10:
            zerosNeeded = 10 - len(isbn)
            isbn = '0'*zerosNeeded + isbn # add zeros in the front to get the isbn10
        isbnList.append(isbn)
    return isbnList
```


```python
# preprocess isbns for merging
uniqueIsbns = getUniqueIsbns(bookFreq)
uniqueIsbns[:5]
```




    ['0345465083', '0618119760', '0446612545', '0385509456', '0385495145']




```python
# check if all isbns were processed
len(uniqueIsbns)
```




    106




```python
# Create a dataframe copy and update the cleaned isbns 
bookFreq_clean = bookFreq.copy()
bookFreq_clean['isbn'] = uniqueIsbns
```


```python
# merge dataset 
popBooks = pd.merge(bookFreq_clean, books_df, on="isbn")
popBooks.head()
```




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
      <th>isbn</th>
      <th>count</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0345465083</td>
      <td>3440</td>
      <td>Seabiscuit</td>
      <td>LAURA HILLENBRAND</td>
      <td>2003</td>
      <td>Ballantine Books</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0618119760</td>
      <td>3419</td>
      <td>The Saints of Big Harbour: A Novel</td>
      <td>Lynn Coady</td>
      <td>2002</td>
      <td>Houghton Mifflin Company</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>6</td>
      <td>1</td>
      <td>0.166667</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0446612545</td>
      <td>3270</td>
      <td>The Beach House</td>
      <td>James Patterson</td>
      <td>2003</td>
      <td>Warner Books</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0385509456</td>
      <td>3238</td>
      <td>The Curious Incident of the Dog in the Night-T...</td>
      <td>MARK HADDON</td>
      <td>2003</td>
      <td>Doubleday</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0385495145</td>
      <td>3221</td>
      <td>The Messenger</td>
      <td>JOSEPH GIRZONE</td>
      <td>2002</td>
      <td>Image</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>2</td>
      <td>1</td>
      <td>0.500000</td>
      <td>2000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# check if merged all books successfully
popBooks.shape
```




    (106, 13)




```python
# extract the top hundred books and save to dataframe
top100 = popBooks.sort_values('count', ascending=False)[:100]
most_read = top100.copy().reset_index(drop=True)
most_read = most_read.drop(['count'],axis=1) # drop count column to be able to compare to books dataframe
most_read.head()
```




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
      <th>isbn</th>
      <th>count</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0345465083</td>
      <td>3440</td>
      <td>Seabiscuit</td>
      <td>LAURA HILLENBRAND</td>
      <td>2003</td>
      <td>Ballantine Books</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0618119760</td>
      <td>3419</td>
      <td>The Saints of Big Harbour: A Novel</td>
      <td>Lynn Coady</td>
      <td>2002</td>
      <td>Houghton Mifflin Company</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>6</td>
      <td>1</td>
      <td>0.166667</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0446612545</td>
      <td>3270</td>
      <td>The Beach House</td>
      <td>James Patterson</td>
      <td>2003</td>
      <td>Warner Books</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0385509456</td>
      <td>3238</td>
      <td>The Curious Incident of the Dog in the Night-T...</td>
      <td>MARK HADDON</td>
      <td>2003</td>
      <td>Doubleday</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0385495145</td>
      <td>3221</td>
      <td>The Messenger</td>
      <td>JOSEPH GIRZONE</td>
      <td>2002</td>
      <td>Image</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>2</td>
      <td>1</td>
      <td>0.500000</td>
      <td>2000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# extract all books that don't belong to top 100 list
bottom900 = pd.concat([books_df,most_read]).drop_duplicates(keep=False)
bottom900.head()
```




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
      <th>isbn</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>0002005018</td>
      <td>Clara Callan</td>
      <td>Richard Bruce Wright</td>
      <td>2001</td>
      <td>HarperFlamingo Canada</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0060973129</td>
      <td>Decision in Normandy</td>
      <td>Carlo D'Este</td>
      <td>1991</td>
      <td>HarperPerennial</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>9</td>
      <td>7</td>
      <td>0.777778</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0374157065</td>
      <td>Flu: The Story of the Great Influenza Pandemic...</td>
      <td>Gina Bari Kolata</td>
      <td>1999</td>
      <td>Farrar Straus Giroux</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0393045218</td>
      <td>The Mummies of Urumchi</td>
      <td>E. J. W. Barber</td>
      <td>1999</td>
      <td>W. W. Norton &amp;amp; Company</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>7</td>
      <td>4</td>
      <td>0.571429</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0399135782</td>
      <td>The Kitchen God's Wife</td>
      <td>Amy Tan</td>
      <td>1991</td>
      <td>Putnam Pub Group</td>
      <td>http://images.amazon.com/images/P/0399135782.0...</td>
      <td>http://images.amazon.com/images/P/0399135782.0...</td>
      <td>http://images.amazon.com/images/P/0399135782.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>1990</td>
    </tr>
  </tbody>
</table>
</div>



<a id='ttest'></a>
## 4. Hypothesis testing

Having broken down our datasets into two independent disjoint sets we can perform different hypothesis tests with regards to the books and the number of copies and other data contained in these datasets.



```python
import scipy.stats as stats
from plotly.figure_factory import create_table
```


```python
def performTTest(group1,group2,var=False):
    """
    performs a two sample t-test and returns the result
    """
    return stats.ttest_ind(group1, group2, 
                                nan_policy='omit', # omit nan values from the samples 
                                equal_var=var) # don't assume that variance is the same
```


```python
def showTTestResults(ttest):
    """
    given the results of a two-sample t-test creates a table that shows the results of the test
    """
    matrix_twosample = [
    ['', 'Test Statistic', 'p-value'],
    ['Sample Data', ttest[0], ttest[1]/2]] 
    
    test_table = create_table(matrix_twosample, index=True)
    test_table.show()
```

### 4.1 Hypothesis test regarding book copies

**Null Hypothesis:** The average number of available copies for books in the top 100 books and bottomw 900 books is same.

**Alternative Hypothesis:** The number of copies available for books in the top 100 books is larger.

**Significance Level**: 95% (p-value needs to be smaller than 0.05 to reject the null hypothesis)


```python
# summary statistics of book copies of bottom 900 books
bottom900['copies'].describe()
```




    count    899.000000
    mean       4.936596
    std        2.585161
    min        1.000000
    25%        3.000000
    50%        5.000000
    75%        7.000000
    max        9.000000
    Name: copies, dtype: float64




```python
# summary statistics of book copies of top 100 books
top100['copies'].describe()
```




    count    100.000000
    mean       4.920000
    std        2.751235
    min        1.000000
    25%        2.000000
    50%        5.000000
    75%        8.000000
    max        9.000000
    Name: copies, dtype: float64




```python
# t-test of average number of copies of books 
showTTestResults(performTTest(top100['copies'],bottom900['copies']))
```


<div>


            <div id="bf765fd8-cdc4-4a95-94bb-bf427272aacb" class="plotly-graph-div" style="height:110px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("bf765fd8-cdc4-4a95-94bb-bf427272aacb")) {
                    Plotly.newPlot(
                        'bf765fd8-cdc4-4a95-94bb-bf427272aacb',
                        [{"colorscale": [[0, "#00083e"], [0.5, "#ededee"], [1, "#ffffff"]], "hoverinfo": "none", "opacity": 0.75, "showscale": false, "type": "heatmap", "z": [[0, 0, 0], [0, 0.5, 0.5]]}],
                        {"annotations": [{"align": "left", "font": {"color": "#ffffff"}, "showarrow": false, "text": "<b></b>", "x": -0.45, "xanchor": "left", "xref": "x", "y": 0, "yref": "y"}, {"align": "left", "font": {"color": "#ffffff"}, "showarrow": false, "text": "<b>Test Statistic</b>", "x": 0.55, "xanchor": "left", "xref": "x", "y": 0, "yref": "y"}, {"align": "left", "font": {"color": "#ffffff"}, "showarrow": false, "text": "<b>p-value</b>", "x": 1.55, "xanchor": "left", "xref": "x", "y": 0, "yref": "y"}, {"align": "left", "font": {"color": "#ffffff"}, "showarrow": false, "text": "<b>Sample Data</b>", "x": -0.45, "xanchor": "left", "xref": "x", "y": 1, "yref": "y"}, {"align": "left", "font": {"color": "#000000"}, "showarrow": false, "text": "-0.057562358049431266", "x": 0.55, "xanchor": "left", "xref": "x", "y": 1, "yref": "y"}, {"align": "left", "font": {"color": "#000000"}, "showarrow": false, "text": "0.47709677836146247", "x": 1.55, "xanchor": "left", "xref": "x", "y": 1, "yref": "y"}], "height": 110, "margin": {"b": 0, "l": 0, "r": 0, "t": 0}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "xaxis": {"dtick": 1, "gridwidth": 2, "showticklabels": false, "tick0": -0.5, "ticks": "", "zeroline": false}, "yaxis": {"autorange": "reversed", "dtick": 1, "gridwidth": 2, "showticklabels": false, "tick0": 0.5, "ticks": "", "zeroline": false}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('bf765fd8-cdc4-4a95-94bb-bf427272aacb');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


### 4.2 Hypothesis test regarding availability ratio 

**Null Hypothesis:** The average book ratio for books published in the top 100 and bottom 900 books datasets is the same.

**Alternative Hypothesis:** The book ratio value for books published in the top 100s larger.

**Significance Level**: 95% (p-value needs to be smaller than 0.05 to reject the null hypothesis)


```python
showTTestResults(performTTest(top100['ratio'],bottom900['ratio']))
```


<div>


            <div id="ddd7be11-8aa9-4de7-b113-946c38a0b96b" class="plotly-graph-div" style="height:110px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("ddd7be11-8aa9-4de7-b113-946c38a0b96b")) {
                    Plotly.newPlot(
                        'ddd7be11-8aa9-4de7-b113-946c38a0b96b',
                        [{"colorscale": [[0, "#00083e"], [0.5, "#ededee"], [1, "#ffffff"]], "hoverinfo": "none", "opacity": 0.75, "showscale": false, "type": "heatmap", "z": [[0, 0, 0], [0, 0.5, 0.5]]}],
                        {"annotations": [{"align": "left", "font": {"color": "#ffffff"}, "showarrow": false, "text": "<b></b>", "x": -0.45, "xanchor": "left", "xref": "x", "y": 0, "yref": "y"}, {"align": "left", "font": {"color": "#ffffff"}, "showarrow": false, "text": "<b>Test Statistic</b>", "x": 0.55, "xanchor": "left", "xref": "x", "y": 0, "yref": "y"}, {"align": "left", "font": {"color": "#ffffff"}, "showarrow": false, "text": "<b>p-value</b>", "x": 1.55, "xanchor": "left", "xref": "x", "y": 0, "yref": "y"}, {"align": "left", "font": {"color": "#ffffff"}, "showarrow": false, "text": "<b>Sample Data</b>", "x": -0.45, "xanchor": "left", "xref": "x", "y": 1, "yref": "y"}, {"align": "left", "font": {"color": "#000000"}, "showarrow": false, "text": "-1.486048843456184", "x": 0.55, "xanchor": "left", "xref": "x", "y": 1, "yref": "y"}, {"align": "left", "font": {"color": "#000000"}, "showarrow": false, "text": "0.06993128493139807", "x": 1.55, "xanchor": "left", "xref": "x", "y": 1, "yref": "y"}], "height": 110, "margin": {"b": 0, "l": 0, "r": 0, "t": 0}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "xaxis": {"dtick": 1, "gridwidth": 2, "showticklabels": false, "tick0": -0.5, "ticks": "", "zeroline": false}, "yaxis": {"autorange": "reversed", "dtick": 1, "gridwidth": 2, "showticklabels": false, "tick0": 0.5, "ticks": "", "zeroline": false}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('ddd7be11-8aa9-4de7-b113-946c38a0b96b');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


In neither case of looking at the ratio as well as the number of copies of books available did we find a significant difference in the average copies and copy to available copies ratio.

### 4.3 Examining authors 

Now that we have a merged dataset we can extract the names of the authors of the most popular books and compare those to the bottom 900 books and the authors of these books. 


```python
# authors and number of times they appear in top 100 books
top100['author'].value_counts()
```




    James Patterson         3
    Terry Pratchett         2
    LAURA HILLENBRAND       2
    John Grisham            2
    Enid Blyton             2
                           ..
    ARTHUR PHILLIPS         1
    Stuart Woods            1
    Richard Russo           1
    Jonathan Safran Foer    1
    Sheila Heti             1
    Name: author, Length: 91, dtype: int64




```python
# authors and number of times they appear in bottom 900 books
bottom900['author'].value_counts()
```




    Ray Bradbury          11
    Mary Higgins Clark     9
    John Grisham           9
    PHILIP PULLMAN         7
    Douglas Adams          6
                          ..
    Willa Cather           1
    Dan Quayle             1
    Nicola Griffith        1
    Sandra Levy Ceren      1
    LEE SMITH              1
    Name: author, Length: 702, dtype: int64



If we look at the top authors listed in both groups, and see that the top ones are different, this cannot be interpreted as a sign that there is a significant difference in which authors are popular. Instead the books part of the top 100 list are from a wide array of different authors and are not clustered around a handful of authors.

<a id="feedback"></a>
## 5. Suggestions to the library administrators

Although we have determined which books are in the group of most borrowed books, but we are also asked to make a recommendation of how many more copies of each particular book are need as well as which books should be removed from the library. 

### 5.1 Examining top books borrowing 
To calculate the need of additional books, lets again take a closer look at the number of total copies and available copies exist for the books that we know are part of the most popular books list.


```python
top100.sort_values('copies')
```




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
      <th>isbn</th>
      <th>count</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>90</th>
      <td>0385511612</td>
      <td>2720</td>
      <td>Bleachers</td>
      <td>John Grisham</td>
      <td>2003</td>
      <td>Doubleday</td>
      <td>http://images.amazon.com/images/P/0385511612.0...</td>
      <td>http://images.amazon.com/images/P/0385511612.0...</td>
      <td>http://images.amazon.com/images/P/0385511612.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>41</th>
      <td>0375726403</td>
      <td>2998</td>
      <td>Empire Falls</td>
      <td>Richard Russo</td>
      <td>2002</td>
      <td>Vintage Books USA</td>
      <td>http://images.amazon.com/images/P/0375726403.0...</td>
      <td>http://images.amazon.com/images/P/0375726403.0...</td>
      <td>http://images.amazon.com/images/P/0375726403.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0446612545</td>
      <td>3270</td>
      <td>The Beach House</td>
      <td>James Patterson</td>
      <td>2003</td>
      <td>Warner Books</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0385509456</td>
      <td>3238</td>
      <td>The Curious Incident of the Dog in the Night-T...</td>
      <td>MARK HADDON</td>
      <td>2003</td>
      <td>Doubleday</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0425182908</td>
      <td>3219</td>
      <td>Isle of Dogs</td>
      <td>Patricia Cornwell</td>
      <td>2002</td>
      <td>Berkley Publishing Group</td>
      <td>http://images.amazon.com/images/P/0425182908.0...</td>
      <td>http://images.amazon.com/images/P/0425182908.0...</td>
      <td>http://images.amazon.com/images/P/0425182908.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>56</th>
      <td>0451208080</td>
      <td>2925</td>
      <td>The Short Forever</td>
      <td>Stuart Woods</td>
      <td>2003</td>
      <td>Signet Book</td>
      <td>http://images.amazon.com/images/P/0451208080.0...</td>
      <td>http://images.amazon.com/images/P/0451208080.0...</td>
      <td>http://images.amazon.com/images/P/0451208080.0...</td>
      <td>9</td>
      <td>4</td>
      <td>0.444444</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>65</th>
      <td>0316666343</td>
      <td>2845</td>
      <td>The Lovely Bones: A Novel</td>
      <td>Alice Sebold</td>
      <td>2002</td>
      <td>Little, Brown</td>
      <td>http://images.amazon.com/images/P/0316666343.0...</td>
      <td>http://images.amazon.com/images/P/0316666343.0...</td>
      <td>http://images.amazon.com/images/P/0316666343.0...</td>
      <td>9</td>
      <td>5</td>
      <td>0.555556</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>73</th>
      <td>0786015276</td>
      <td>2832</td>
      <td>11th Hour</td>
      <td>Bradley Warshauer</td>
      <td>2003</td>
      <td>Pinnacle Books</td>
      <td>http://images.amazon.com/images/P/0786015276.0...</td>
      <td>http://images.amazon.com/images/P/0786015276.0...</td>
      <td>http://images.amazon.com/images/P/0786015276.0...</td>
      <td>9</td>
      <td>2</td>
      <td>0.222222</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>59</th>
      <td>0449005615</td>
      <td>2909</td>
      <td>Seabiscuit: An American Legend</td>
      <td>LAURA HILLENBRAND</td>
      <td>2002</td>
      <td>Ballantine Books</td>
      <td>http://images.amazon.com/images/P/0449005615.0...</td>
      <td>http://images.amazon.com/images/P/0449005615.0...</td>
      <td>http://images.amazon.com/images/P/0449005615.0...</td>
      <td>9</td>
      <td>6</td>
      <td>0.666667</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>54</th>
      <td>0964778319</td>
      <td>2928</td>
      <td>An Atmosphere of Eternity: Stories of India</td>
      <td>David Iglehart</td>
      <td>2002</td>
      <td>Sunflower Press</td>
      <td>http://images.amazon.com/images/P/0964778319.0...</td>
      <td>http://images.amazon.com/images/P/0964778319.0...</td>
      <td>http://images.amazon.com/images/P/0964778319.0...</td>
      <td>9</td>
      <td>1</td>
      <td>0.111111</td>
      <td>2000</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 13 columns</p>
</div>



To set up a formula for calculating the need for more copies for particular books, let's first take a closer look at the borrowing history for one of the top books. Here we take the first book listed in our dataframe, titled "Bleachers" by John Grisham. 


```python
# examine borrowing history of a particular book over the year
topBooks[topBooks['isbn'] == '385511612'].head(10)
```




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
      <th>week</th>
      <th>isbn</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>45</th>
      <td>1</td>
      <td>385511612</td>
      <td>10</td>
    </tr>
    <tr>
      <th>151</th>
      <td>2</td>
      <td>385511612</td>
      <td>23</td>
    </tr>
    <tr>
      <th>257</th>
      <td>3</td>
      <td>385511612</td>
      <td>54</td>
    </tr>
    <tr>
      <th>363</th>
      <td>4</td>
      <td>385511612</td>
      <td>72</td>
    </tr>
    <tr>
      <th>469</th>
      <td>5</td>
      <td>385511612</td>
      <td>27</td>
    </tr>
    <tr>
      <th>575</th>
      <td>6</td>
      <td>385511612</td>
      <td>23</td>
    </tr>
    <tr>
      <th>681</th>
      <td>7</td>
      <td>385511612</td>
      <td>15</td>
    </tr>
    <tr>
      <th>787</th>
      <td>8</td>
      <td>385511612</td>
      <td>71</td>
    </tr>
    <tr>
      <th>893</th>
      <td>9</td>
      <td>385511612</td>
      <td>92</td>
    </tr>
    <tr>
      <th>999</th>
      <td>10</td>
      <td>385511612</td>
      <td>54</td>
    </tr>
  </tbody>
</table>
</div>




```python
# look at borrowing statistics for the year
topBooks[topBooks['isbn'] == '385511612']['count'].describe()
```




    count    52.000000
    mean     52.307692
    std      27.878264
    min      10.000000
    25%      28.500000
    50%      54.000000
    75%      77.000000
    max      94.000000
    Name: count, dtype: float64



### 5.2 Developing a measure to determine extra need

If we take the mean number of times the book was borrowed per week by looking at the total number of borrowings divided by the number of weeks in a year (52 weeks) we get that the book was borrowed on average 52 times per week. That is a very large number (and also unrealistic) for a book where there is only one copy. 

If we assume that the numbers are indeed correct, this would mean that there would be 7 borrows for a that particular book per day (highly unlikely!). Most individuals might take at least a week for a book. But under the assumption that this libary is an outlier and the readers are in fact very fast and take only a day that would mean that there is a need for around seven more book copies of that particular book to meet demand. 

Using this reasoning we can then deduce a formula to calculate how many books are needed:
The result we want is that there is that the ratio of one borrow per copy per day for a given book title (at minimum).
First calculate the mean number of borrows per day. Then since for reasons of simplicity we assume that everyone using the library can finish a book within a day we want to have a book available to be borrowed at least once every day, then to achieve this ratio we take the average number of books borrowed on any given day and subtract the number of copies available to get the further quantity of books needed.


```python
import math
```


```python
def booksNeeded(numCopies,totalCount):
    """
    given the number of copies of a book that exist and the total number of books that were borrowed throughout 
    the years returns the number of additional books that need to be purchased to reach the ratio of having 
    a copy per customer per day 
    """
    daily_ratio = totalCount/52/7 # calculates on average how often a book gets borrowed per day
    additional_need = math.ceil(daily_ratio) - numCopies
    if (daily_ratio > 1) and additional_need >0: # since one is our cutoff
        return additional_need
    return 0 # no additional book copies needed
```


```python
# save the number of additional copies needed in new column
top100['additional_need'] = top100.apply(lambda val: booksNeeded(val['copies'], val['count']), axis=1)
```


```python
# summary statistics on how many additional books are needed
top100['additional_need'].describe()
```




    count    100.000000
    mean       3.770000
    std        2.740843
    min        0.000000
    25%        1.000000
    50%        4.000000
    75%        6.000000
    max        8.000000
    Name: additional_need, dtype: float64




```python
# visualize additional book needs
pl = top100['additional_need'].plot.hist(bins=len(top100['additional_need'].unique()), 
                                         title='distribution of number of additional books needed')
pl.set_xlabel("number of additional books")
```




    Text(0.5, 0, 'number of additional books')




![png](output_57_1.png)



```python
# books where we need more copies 
top100[top100['additional_need'] > 0]
```




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
      <th>isbn</th>
      <th>count</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
      <th>additional_need</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0345465083</td>
      <td>3440</td>
      <td>Seabiscuit</td>
      <td>LAURA HILLENBRAND</td>
      <td>2003</td>
      <td>Ballantine Books</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
      <td>8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0618119760</td>
      <td>3419</td>
      <td>The Saints of Big Harbour: A Novel</td>
      <td>Lynn Coady</td>
      <td>2002</td>
      <td>Houghton Mifflin Company</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>6</td>
      <td>1</td>
      <td>0.166667</td>
      <td>2000</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0446612545</td>
      <td>3270</td>
      <td>The Beach House</td>
      <td>James Patterson</td>
      <td>2003</td>
      <td>Warner Books</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0385509456</td>
      <td>3238</td>
      <td>The Curious Incident of the Dog in the Night-T...</td>
      <td>MARK HADDON</td>
      <td>2003</td>
      <td>Doubleday</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
      <td>8</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0385495145</td>
      <td>3221</td>
      <td>The Messenger</td>
      <td>JOSEPH GIRZONE</td>
      <td>2002</td>
      <td>Image</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>2</td>
      <td>1</td>
      <td>0.500000</td>
      <td>2000</td>
      <td>7</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>95</th>
      <td>0743418190</td>
      <td>2682</td>
      <td>In Her Shoes : A Novel</td>
      <td>Jennifer Weiner</td>
      <td>2002</td>
      <td>Atria Books</td>
      <td>http://images.amazon.com/images/P/0743418190.0...</td>
      <td>http://images.amazon.com/images/P/0743418190.0...</td>
      <td>http://images.amazon.com/images/P/0743418190.0...</td>
      <td>5</td>
      <td>1</td>
      <td>0.200000</td>
      <td>2000</td>
      <td>3</td>
    </tr>
    <tr>
      <th>96</th>
      <td>0142001740</td>
      <td>2679</td>
      <td>The Secret Life of Bees</td>
      <td>Sue Monk Kidd</td>
      <td>2003</td>
      <td>Penguin Books</td>
      <td>http://images.amazon.com/images/P/0142001740.0...</td>
      <td>http://images.amazon.com/images/P/0142001740.0...</td>
      <td>http://images.amazon.com/images/P/0142001740.0...</td>
      <td>5</td>
      <td>1</td>
      <td>0.200000</td>
      <td>2000</td>
      <td>3</td>
    </tr>
    <tr>
      <th>97</th>
      <td>051513290X</td>
      <td>2675</td>
      <td>Summer of Storms</td>
      <td>Judith Kelman</td>
      <td>2002</td>
      <td>Jove Books</td>
      <td>http://images.amazon.com/images/P/051513290X.0...</td>
      <td>http://images.amazon.com/images/P/051513290X.0...</td>
      <td>http://images.amazon.com/images/P/051513290X.0...</td>
      <td>3</td>
      <td>2</td>
      <td>0.666667</td>
      <td>2000</td>
      <td>5</td>
    </tr>
    <tr>
      <th>98</th>
      <td>0385508042</td>
      <td>2663</td>
      <td>The King of Torts</td>
      <td>John Grisham</td>
      <td>2003</td>
      <td>Doubleday Books</td>
      <td>http://images.amazon.com/images/P/0385508042.0...</td>
      <td>http://images.amazon.com/images/P/0385508042.0...</td>
      <td>http://images.amazon.com/images/P/0385508042.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
      <td>7</td>
    </tr>
    <tr>
      <th>99</th>
      <td>0771076002</td>
      <td>2635</td>
      <td>Remembering Peter Gzowski : A Book of Tributes</td>
      <td>EDNA BARKER</td>
      <td>2002</td>
      <td>Douglas Gibson Books</td>
      <td>http://images.amazon.com/images/P/0771076002.0...</td>
      <td>http://images.amazon.com/images/P/0771076002.0...</td>
      <td>http://images.amazon.com/images/P/0771076002.0...</td>
      <td>4</td>
      <td>1</td>
      <td>0.250000</td>
      <td>2000</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
<p>83 rows × 14 columns</p>
</div>



In total there are thus 83 of the top 100 books of which we need additional copies. The column additional_copies details the exact need of how many copies are needed in addition to satisfy the demand according to the assumptions mentioned above.

## 5.3 Determing which books to remove
Next we also want to look at the worst 100 books. Now we have a list of books that do not belong to the 106 most popular books.


```python
# extract the top 106 books
allTop = popBooks.sort_values('count', ascending=False)
most_readAll = allTop.copy().reset_index(drop=True)
most_readAll = most_readAll.drop(['count'],axis=1) # drop count column to be able to compare to books dataframe
most_readAll.head()
```




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
      <th>isbn</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0345465083</td>
      <td>Seabiscuit</td>
      <td>LAURA HILLENBRAND</td>
      <td>2003</td>
      <td>Ballantine Books</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>http://images.amazon.com/images/P/0345465083.0...</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0618119760</td>
      <td>The Saints of Big Harbour: A Novel</td>
      <td>Lynn Coady</td>
      <td>2002</td>
      <td>Houghton Mifflin Company</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>http://images.amazon.com/images/P/0618119760.0...</td>
      <td>6</td>
      <td>1</td>
      <td>0.166667</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0446612545</td>
      <td>The Beach House</td>
      <td>James Patterson</td>
      <td>2003</td>
      <td>Warner Books</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>http://images.amazon.com/images/P/0446612545.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0385509456</td>
      <td>The Curious Incident of the Dog in the Night-T...</td>
      <td>MARK HADDON</td>
      <td>2003</td>
      <td>Doubleday</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>http://images.amazon.com/images/P/0385509456.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0385495145</td>
      <td>The Messenger</td>
      <td>JOSEPH GIRZONE</td>
      <td>2002</td>
      <td>Image</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>http://images.amazon.com/images/P/0385495145.0...</td>
      <td>2</td>
      <td>1</td>
      <td>0.500000</td>
      <td>2000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# extract the books that do not belong to the top 106 books
lessPop = pd.concat([books_df,most_readAll]).drop_duplicates(keep=False)
lessPop.head()
```




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
      <th>isbn</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>0002005018</td>
      <td>Clara Callan</td>
      <td>Richard Bruce Wright</td>
      <td>2001</td>
      <td>HarperFlamingo Canada</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>http://images.amazon.com/images/P/0002005018.0...</td>
      <td>2</td>
      <td>0</td>
      <td>0.000000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0060973129</td>
      <td>Decision in Normandy</td>
      <td>Carlo D'Este</td>
      <td>1991</td>
      <td>HarperPerennial</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>http://images.amazon.com/images/P/0060973129.0...</td>
      <td>9</td>
      <td>7</td>
      <td>0.777778</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0374157065</td>
      <td>Flu: The Story of the Great Influenza Pandemic...</td>
      <td>Gina Bari Kolata</td>
      <td>1999</td>
      <td>Farrar Straus Giroux</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>http://images.amazon.com/images/P/0374157065.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0393045218</td>
      <td>The Mummies of Urumchi</td>
      <td>E. J. W. Barber</td>
      <td>1999</td>
      <td>W. W. Norton &amp;amp; Company</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>http://images.amazon.com/images/P/0393045218.0...</td>
      <td>7</td>
      <td>4</td>
      <td>0.571429</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0399135782</td>
      <td>The Kitchen God's Wife</td>
      <td>Amy Tan</td>
      <td>1991</td>
      <td>Putnam Pub Group</td>
      <td>http://images.amazon.com/images/P/0399135782.0...</td>
      <td>http://images.amazon.com/images/P/0399135782.0...</td>
      <td>http://images.amazon.com/images/P/0399135782.0...</td>
      <td>1</td>
      <td>0</td>
      <td>0.000000</td>
      <td>1990</td>
    </tr>
  </tbody>
</table>
</div>




```python
# total of 893 books that are not part of popular books list
lessPop.shape
```




    (893, 12)



Since there is no data on borrowing history for these book, again we have to come up with a simplistic metric that would help us determine which books have had the worse performance. We make the strongly simplifiying assumption that the current borrowing rate is reflective of the borrowing rate throughout the whole year. Therefore by taking the borrowing ratio as the metric to determine which 100 books the library should get rid of we sort our dataset by ratio and take the books with the highest ratio (those with the least borrowings). These are the books that should be removed from our library as they are not very popular and occupy space. A good way to deal with these books is to sell them or donate them to library patrons.


```python
# the hundred books that we won't be needing 
lessPop.sort_values('ratio',ascending=False)[:100]
```




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
      <th>isbn</th>
      <th>title</th>
      <th>author</th>
      <th>publication_year</th>
      <th>publisher</th>
      <th>image_url_s</th>
      <th>image_url_m</th>
      <th>image_url_l</th>
      <th>copies</th>
      <th>available</th>
      <th>ratio</th>
      <th>decade</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>744</th>
      <td>3746670055</td>
      <td>Women are the Niggers of the World: über Fraue...</td>
      <td>Peter B Heim</td>
      <td>1994</td>
      <td>Aufbau Taschenbuch Verlag</td>
      <td>http://images.amazon.com/images/P/3746670055.0...</td>
      <td>http://images.amazon.com/images/P/3746670055.0...</td>
      <td>http://images.amazon.com/images/P/3746670055.0...</td>
      <td>9</td>
      <td>8</td>
      <td>0.888889</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>521</th>
      <td>0192815318</td>
      <td>Cranford (The World's Classics)</td>
      <td>Elizabeth Gaskell</td>
      <td>1982</td>
      <td>Oxford University Press</td>
      <td>http://images.amazon.com/images/P/0192815318.0...</td>
      <td>http://images.amazon.com/images/P/0192815318.0...</td>
      <td>http://images.amazon.com/images/P/0192815318.0...</td>
      <td>9</td>
      <td>8</td>
      <td>0.888889</td>
      <td>1980</td>
    </tr>
    <tr>
      <th>818</th>
      <td>1842151053</td>
      <td>Cooking for One (Cook's Essentials)</td>
      <td>Valerie Ferguson</td>
      <td>2000</td>
      <td>Southwater Publishing</td>
      <td>http://images.amazon.com/images/P/1842151053.0...</td>
      <td>http://images.amazon.com/images/P/1842151053.0...</td>
      <td>http://images.amazon.com/images/P/1842151053.0...</td>
      <td>9</td>
      <td>8</td>
      <td>0.888889</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>728</th>
      <td>2070362388</td>
      <td>Ravage</td>
      <td>Rene Barjavel</td>
      <td>0</td>
      <td>Gallimard French</td>
      <td>http://images.amazon.com/images/P/2070362388.0...</td>
      <td>http://images.amazon.com/images/P/2070362388.0...</td>
      <td>http://images.amazon.com/images/P/2070362388.0...</td>
      <td>9</td>
      <td>8</td>
      <td>0.888889</td>
      <td>0</td>
    </tr>
    <tr>
      <th>956</th>
      <td>0793133955</td>
      <td>Wall Street's Picks for 2000: An Insider's Gui...</td>
      <td>Kirk Kazanjian</td>
      <td>1999</td>
      <td>Dearborn Financial Publishing</td>
      <td>http://images.amazon.com/images/P/0793133955.0...</td>
      <td>http://images.amazon.com/images/P/0793133955.0...</td>
      <td>http://images.amazon.com/images/P/0793133955.0...</td>
      <td>9</td>
      <td>8</td>
      <td>0.888889</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>353</th>
      <td>0671847546</td>
      <td>REAL GUIDE: CALIFORNIA AND THE WEST COAST (The...</td>
      <td>LTD</td>
      <td>1993</td>
      <td>John Wiley &amp;amp; Sons Inc</td>
      <td>http://images.amazon.com/images/P/0671847546.0...</td>
      <td>http://images.amazon.com/images/P/0671847546.0...</td>
      <td>http://images.amazon.com/images/P/0671847546.0...</td>
      <td>4</td>
      <td>3</td>
      <td>0.750000</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>169</th>
      <td>0671047612</td>
      <td>Skin And Bones</td>
      <td>Franklin W. Dixon</td>
      <td>2000</td>
      <td>Aladdin</td>
      <td>http://images.amazon.com/images/P/0671047612.0...</td>
      <td>http://images.amazon.com/images/P/0671047612.0...</td>
      <td>http://images.amazon.com/images/P/0671047612.0...</td>
      <td>8</td>
      <td>6</td>
      <td>0.750000</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>216</th>
      <td>0380807866</td>
      <td>The Elusive Flame</td>
      <td>Kathleen E. Woodiwiss</td>
      <td>1999</td>
      <td>Avon</td>
      <td>http://images.amazon.com/images/P/0380807866.0...</td>
      <td>http://images.amazon.com/images/P/0380807866.0...</td>
      <td>http://images.amazon.com/images/P/0380807866.0...</td>
      <td>4</td>
      <td>3</td>
      <td>0.750000</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>847</th>
      <td>1563411148</td>
      <td>The Second Coming of Curly Red</td>
      <td>Jody Seay</td>
      <td>1999</td>
      <td>Firebrand Books</td>
      <td>http://images.amazon.com/images/P/1563411148.0...</td>
      <td>http://images.amazon.com/images/P/1563411148.0...</td>
      <td>http://images.amazon.com/images/P/1563411148.0...</td>
      <td>8</td>
      <td>6</td>
      <td>0.750000</td>
      <td>1990</td>
    </tr>
    <tr>
      <th>991</th>
      <td>0380817144</td>
      <td>Lord of the Silent: A Novel of Suspense</td>
      <td>Elizabeth Peters</td>
      <td>2002</td>
      <td>Avon</td>
      <td>http://images.amazon.com/images/P/0380817144.0...</td>
      <td>http://images.amazon.com/images/P/0380817144.0...</td>
      <td>http://images.amazon.com/images/P/0380817144.0...</td>
      <td>4</td>
      <td>3</td>
      <td>0.750000</td>
      <td>2000</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 12 columns</p>
</div>




```python

```
