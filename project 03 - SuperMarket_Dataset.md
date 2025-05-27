# Supermarket Data Insights: A Comprehensive Analytics Project
This project involves data analysis using the supermarket_data dataset from Kaggle. 
The goal is to summarize findings and perform deep-dive analysis across various aspects, including sales, 
customer behavior, and shipping. For this analysis, R programming language will be used in RStudio, 
and the results will be presented as an interactive dashboard using Tableau.

## RStudio code
```R
##import library
library(tidyverse)
library(readxl)
library(ggplot2)
library(stringr)
library(lubridate)

##load data
data <- read_xlsx("Supermarket_Dataset.xlsx")
glimpse(data
```
### Clean data
``` R
#remove $ from sales & Profit + Convert Postal to Character
df_cleaned <-data %>%
  mutate(
  sales_numeric = as.numeric(str_replace_all(Sales,"\\$|,","")),
  profit_numeric = as.numeric(str_replace_all(Profit,"\\$|,","")),
  postal_code = as.character(`Postal Code`)
)
```
### Create New Column
``` R
df_add_col <- df_cleaned %>%
  mutate(
    order_day_of_week = wday(`Order Date`, label = TRUE, abbr = FALSE, locale = "en_US"),
    ship_day_of_week = wday(`Ship Date`, label = TRUE, abbr = FALSE, local = "en_US")
  )

df_final <- df_add_col %>%
  mutate( net_revenue = sales_numeric * (1 - Discount),
          rev_per_unit = round((net_revenue / Quantity),2),
          sales_per_unit = round((sales_numeric / Quantity),2),
          profit_per_unit = round((profit_numeric / Quantity),2),
        ) %>%
  mutate(
    order_year = year(`Order Date`),
    shipping_year = year(`Ship Date`)
  ) %>%
  mutate(
    shipping_duration = as.numeric(difftime(`Ship Date`,`Order Date`, units = "days"))
  ) %>%
  select(-Sales, -Profit, -`Number of Records`)
glimpse(df_final)
```
#### Result
![image](https://github.com/user-attachments/assets/dcd2ffdd-c20d-4912-84fd-af8e53db3cf2)
![image](https://github.com/user-attachments/assets/1e976004-aa26-49a6-b363-292dbe8efef3)

### Analyzed
``` R
##Overall performance
Overall_Perform <-df_final %>%
  summarise( total_sales = sum(sales_numeric),
             total_discount = sum(Discount * sales_numeric),
             total_net_revenue = sum(net_revenue))

##Overall by Year
A2 <- df_final %>%
  group_by(order_year) %>%
  summarise(Total_Sales = sum(sales_numeric),
            Total_Discount = sum(Discount * sales_numeric),
            Total_Revenue = sum(net_revenue)) %>%
  arrange(order_year)

##Revenue by Category
sales_by_category <- df_final %>%
  group_by(Category) %>%
  summarise( Count_Order = n(),
             Quantity = sum(Quantity),
             Total_Revenue = sum(net_revenue))

##Revenue by Product name
product_performance_ranking <- df_final %>%
  group_by(`Product Name`) %>%
  summarise( Quantity = sum(Quantity),
             Total_Revenue = sum(net_revenue),
             Avg_Rev_Unit = mean(rev_per_unit) ) %>%
  arrange(desc(Total_Revenue)) %>%
  head(10)

##Total Loss by Sub-cat
loss_by_subcategory <- df_final %>%
  filter(profit_numeric < 0) %>%
  group_by(`Sub-Category`) %>%
  summarise( Quantity = sum(Quantity),
             Revenue_Loss = sum(profit_numeric)) %>%
  arrange(Revenue_Loss)

##Top 10 Customers by Purchases
top_10_customers_by_purchase <- df_final%>%
  group_by(`Customer Name`) %>%
  summarise( Count_Order = n(),
             Total_Purchase = sum(sales_numeric)) %>%
  arrange(desc(Total_Purchase)) %>%
  head(10)

##Top 10 Total Revenue by State
df_final %>%
  filter(order_year == 2016) %>%
  group_by(State) %>%
  summarise( Total_Transactions = n(),
             Total_Revenue = sum(net_revenue)) %>%
  arrange(desc(Total_Revenue)) %>%
  head(10)

## Total Revenue by Region
df_final %>%
  group_by(Region) %>%
  summarise(Total_Order = n(),
            Total_Revenue = sum(net_revenue)) %>%
  arrange(desc(Total_Revenue))

##Count Order in Day of week 
daily_performance_2016 <- df_final %>%
  filter(order_year == 2016) %>%
  group_by(order_year,order_day_of_week) %>%
  summarise(Count_Order = n(),
            Total_Revenue = sum(net_revenue)) %>%
  arrange(order_year,desc(Count_Order))

##Customer Segment
customer_purchase_ranking <- df_final%>%
  group_by(`Customer Name`) %>%
  summarise( Count_Order = n(),
             Total_Purchase = sum(sales_numeric)) %>%
  arrange(desc(Total_Purchase))

customer_segments_classified <- customer_purchase_ranking %>%
  mutate( 
    customer_segment = case_when(
    Total_Purchase <= 1000 ~ "Low Value Customer",
    Total_Purchase > 1000 & Total_Purchase < 5000 ~ "Medium Value Customer",
    TRUE ~ "High Value Customer(VIP)")
  ) %>%
  arrange(`Customer Name`)

#Count Customer Segment & Avg Purchases
segment_summary_performance <- customer_segments_classified %>%
  group_by(customer_segment) %>%
  summarise( Count_segment = n(),
             Avg_purchase = mean(Total_Purchase)) %>%
  arrange(desc(Avg_purchase))

##Total sales per segment
df_final %>%
  group_by(Segment) %>%
  summarise( count = n(),
             Total_Revenue = sum(net_revenue),
             Avg_Revenue = round(mean(net_revenue),2)) %>%
  arrange(desc(Total_Revenue))

## About Shipping
shipping_mode_performance <- df_final %>%
  group_by(`Ship Mode`) %>%
  summarise( Count = n(),
             Total_Value_Order = sum(net_revenue),
             Fastest_Ship = min(shipping_duration),
             Slowest_Ship = max(shipping_duration),
             Average_Ship = mean(shipping_duration))

shipments_by_day_of_week <- df_final %>%
  group_by(ship_day_of_week) %>%
  summarise(Count_Order = n()) %>%
  arrange(desc(Count_Order))

## About Manufacturer
top_10_manufacturers_low_profit_ratio <- df_final %>%
  group_by(Manufacturer) %>%
  summarise( Quantity = sum(Quantity),
             Total_Sales = sum(sales_numeric),
             Avg_Discount = round(mean(Discount),2),
             Avg_ProfitRatio = round(mean(`Profit Ratio`),2),
             Total_Profit = sum(profit_numeric)
             ) %>%
  arrange(Avg_ProfitRatio) %>%
  head(10)

top_10_manufacturers_by_profit <- df_final %>%
  group_by(Manufacturer) %>%
  summarise( Quantity = sum(Quantity),
             Total_Sales = sum(sales_numeric),
             Avg_Discount = round(mean(Discount),2),
             Avg_ProfitRatio = round(mean(`Profit Ratio`),2),
             Total_Profit = sum(profit_numeric)
  ) %>%
  arrange(desc(Total_Profit)) %>%
  head(10)

#Top 5 Products by Quantity Sold, per Customer Segment. Little Confused ##
Top_5 <- df_final %>%
  group_by(`Sub-Category`,Segment) %>%
  summarise( Quantity = sum(Quantity),
             Avg_Rev_Per_Units = mean(rev_per_unit),
             Total_Revenue = sum(net_revenue)) %>%
  arrange(desc(Quantity)) %>%
  head(5)

##Export df_final
write.csv(df_final, "SuperMarket_Dataset_Export.csv", row.names = FALSE)
```

## Data Visualization
``` R
##Library
library(ggplot2)
library(tidyverse)

#Data set
dataset <- read.csv("SuperMarket_Dataset_Export.csv")
glimpse(dataset)
```

``` R
ggplot(data = dataset,
       mapping = aes(x = Sub.Category)) +
  geom_bar(mapping = aes(fill = Sub.Category), alpha = 0.8) +
  labs(title = "Number of Orders by Category",
       x = "Product Sub-Category",
       y = "Number of Orders",
       fill = "Sub-Category") +
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5)) +
  scale_fill_viridis_d(option = "G")

ggplot(data = dataset,
       mapping = aes(x = ship_day_of_week)) +
  geom_bar(aes(fill = ship_day_of_week)) + 
  labs(title = "Shipping by Day of the Week",
       x = "Day of Week",
       y = "Number of Oders",
       fill = "Day of Week") + 
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5)) +
  scale_fill_brewer(palette = "Pastel1")

options(scipen = 999)
Top_10_Rev <- dataset %>%
  group_by(State) %>%
  summarise( Total_Rev = sum(net_revenue)) %>%
  arrange(desc(Total_Rev)) %>%
  head(10)

ggplot(data = Top_10_Rev,
       mapping = aes(x = State, y = Total_Rev)) + 
  geom_col(aes(fill = State),alpha = 0.7) + 
  labs(title = "Top 10 US States by Revenue",
       y = "Total Revenue") + 
  theme(plot.title = element_text(hjust = 0.5)) + 
  theme_minimal() +
  scale_fill_viridis_d()

ggplot(dataset,
       aes(x = Ship.Mode, y = Quantity)) +
  geom_col(aes(fill = Category)) + 
  theme_minimal() + 
  labs(title = "Quantity of Products by Ship Mode and Product Category",
       x = "Ship mode") +
  scale_fill_carto_d() +
  theme(plot.title = element_text(hjust = 0.5))
```
## Dashboard
You can view the dashboard here. > https://public.tableau.com/app/profile/chitnupong.nakkong/viz/Dataviz_SuperMarket/Dashboard1
Thank you
