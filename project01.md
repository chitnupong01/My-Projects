# Project01: Supplement Sales Analysis: Trends and Insights (2020-2025)
Analyzed Kaggle's "Supplement Sales Analysis (2020-2025)" dataset with Python (Pandas, Matplotlib) to identify trends, product performance, and return impact for strategic business insights, etc. This personal portfolio projec using free Kaggle data for skill enhancement

Skills Demonstrated:
- Data manipulation and cleaning with Pandas
- Basic data analysis and insight generation
- Data visualization for presentation using Matplotlib

## Data Preparation & Cleaning
```python
##import library
import pandas as pd
import matplotlib.pyplot as plt
##load data
df = pd.read_csv('Supplement_Sales_Weekly_Expanded.csv')

##convert date (object) to date
df['Date'] = pd.to_datetime(df['Date'], format='%d/%m/%Y', errors='coerce')
df['Year'] = df['Date'].dt.year
df['Month'] = df['Date'].dt.month
df['Day'] = df['Date'].dt.day
df['Quarter'] = 'Q' + df['Date'].dt.quarter.astype(str)

##Create a calculated column named Net_sales
df['Net_sales'] = df['Revenue'] - (df['Revenue'] * df['Discount'])
##Create a calculated column named "Total Return Value" from the sum of "Value Units Returned" (latest updated).
df['Total Return Value'] = df['Price'] * (1 - df['Discount']) * df['Units Returned']
##Reorder Column
df = df[['Date','Product Name','Category','Units Sold','Price','Revenue','Discount','Net_sales','Units Returned','Total Return Value','Location','Platform','Year']]
```
![image](https://github.com/user-attachments/assets/fad6060f-6fbb-434f-81e8-812249a2fdce)
## Exploratory Data Analysis (EDA)
```python
##Top10 Products by Net Sales per Year
sales_group_year = df.groupby(['Year','Category','Product Name'])[['Units Sold','Net_sales']].sum().reset_index()
sort_salesby_year = sales_group_year.sort_values(['Net_sales'], ascending=[False]).head(10)

##Top5 Best seller in 2025
top5_2025_sales = df.groupby(['Year','Category','Product Name'])['Net_sales'].sum().reset_index()
top5_2025_sales = top5_2025_sales[top5_2025_sales['Year'] == 2025].head(5).sort_values(['Net_sales'],ascending=False)

##Sales Groupby Location, Platform & sort net high to low
sales_by_location = df.groupby(['Location','Platform'])['Net_sales'].sum().reset_index().sort_values(['Net_sales'], ascending=[False])
sales_by_location = sales_by_location.style.format({'Net_sales': '{:.2f}'})

##Top 3 per Platform / apply() - for performing an action on grouped data.
top5_per_platform = df.groupby('Platform',group_keys= False).apply(lambda x: x.sort_values(by='Units Sold', ascending=False).head(5))
result = top5_per_platform[['Platform','Product Name','Units Sold']]

##Top 3 Product per Year
top3_by_year = df.groupby('Year', group_keys=False).apply(lambda x: x.sort_values(by='Units Sold', ascending=False).head(3))
result_top3_by_year = top3_by_year[['Year','Category','Units Sold']]

##Return Value by Product Category (Descending)
amt_unit_group = df.groupby(['Category'])[['Units Returned','Total Return Value']].sum().reset_index()
result = amt_unit_group.sort_values(by = 'Units Returned', ascending = False)

##Check Units Returned
sum_return = df.groupby('Category')['Units Returned'].sum().sort_values( ascending= False)
```
## Data Visualization
```python
##Bar chart net sales by category
category_net = df.groupby('Category')['Net_sales'].sum().sort_values(ascending=False)
category_net.plot(kind = 'bar', title = 'Net Sales by Category', color = 'salmon', alpha = 0.7)
plt.xlabel('Category')
plt.ylabel('Net Sales')
plt.ticklabel_format(style='plain', axis='y') ##Show numbers in a normal
plt.show()
```
![image](https://github.com/user-attachments/assets/b2ec9c77-d918-4700-8eca-ae5c91a8993c)

```python
##Price Distribution
df['Price'].plot(kind = 'hist', bins = 20, title = 'Distribution of Prices', color = '#bb69f5')
plt.xlabel('Price')
plt.ylabel('Count')
plt.show()
```
![image](https://github.com/user-attachments/assets/01d262f1-07ad-4bfb-b664-2cee52497672)

```python
##Pie chart by Category
colors = ['#FAD0C9','#FFFACD','#CCE2CB', '#ADD8E6', '#D1BCE3','#FFDAB9','#F0E68C','#E0FFFF','#FAF0E6', '#FFB347']
sum_by_category = df.groupby('Category')['Net_sales'].sum()
sum_by_category.plot(kind='pie', title = 'Pie Chart by Category', autopct='%.1f%%', colors = colors) ##legend=True
plt.ylabel('') ##hide y label
plt.show()
```
![image](https://github.com/user-attachments/assets/08abd67e-3e8a-4c19-a7d6-c98889bd2bd3)

```python
##Product Category Return Value
bar_value_returned = df.groupby('Category')['Total Return Value'].sum().sort_values(ascending= False)
bar_value_returned.plot(kind = 'bar', title = 'Product Category Return Value', color = 'salmon', alpha = 0.5)
plt.show()
```
![image](https://github.com/user-attachments/assets/829ac8b0-942e-449c-9439-ac28bcf7f72c)
