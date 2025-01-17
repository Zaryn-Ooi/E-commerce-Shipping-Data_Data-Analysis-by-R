# E-commerce-Shipping-Data_Data-Analysis-with-R

## Backgroud of the Company 
An international e-commerce company based wants to discover key insights from their customer database. 
They want to use some of the most advanced machine learning techniques to study their customers. The company sells electronic products.


## Task
To identify: 
  1)  Relation Between Customer Rating vs Product Arrived on Time 
  2)  Relation Between Customer Care Calls vs Cost of The Product
  3)  Discount Offered over Product Importance 
  4)  Which Mode of Shipment is the Most Desirable?
  5)  Which Gender Made the Most Prior Purchase? 
   
   
## Data Source: 
E-Commerce Shipping Data (Version 1)

<https://www.kaggle.com/prachi13/customer-analytics/code>

This dataset is used for model building contained 10999 observations of 12 variables. The data contains the following information:

  - ID: ID Number of customers.
  - Warehouse block: The company have big warehouse which is divided in to block such as A,B,C,D,E.
  - Mode of shipment:The company ships the products in multiple way such as Ship, Flight and Road.
  - Customer care calls: The number of calls made from enquiry for enquiry of the shipment.
  - Customer rating: The company has rated from every customer. 1 is the lowest (Worst), 5 is the highest (Best).
  - Cost of the product: Cost of the product (USD).
  - Prior purchases: Number of prior purchase.
  - Product importance: The company has categorized the importance of product in various parameter such as low, medium, high.
  - Gender: Male and Female.
  - Discount offered: Discount offered on that specific product.
  - Weight in gms: Product weight in grams.
  - Reached on time: It is the target variable,where 1 indicates that product has NOT reached on time and 0 indicates that products has reached on time.
  

## Install and Load Packages
```{r}
install.packages('tidyverse')
install.packages('dplyr')
install.packages('ggplot2')
install.packages('lubridate')
install.packages('skimr')
library(tidyverse)
library(dplyr)
library(ggplot2)
library(lubridate)
library(skimr)
```

```{r}
install.packages('caret')
install.packages('rpart')
install.packages('randomForest')
install.packages('matrixStats')
install.packages('gbm')
install.packages('data.table')
install.packages('gridExtra')
install.packages('corrplot')
install.packages('rpart.plot')
library(caret)
library(rpart)
library(randomForest)
library(matrixStats)
library(gbm)
library(data.table)
library(gridExtra)
library(corrplot)
library(rpart.plot)
```

## Importing Data 
```{r}
Train <- read_csv("Train.csv")
```


## Explore the Imported Data
Let's use the `head()` function and `skim_without_charts()` function to display the data. 

```{r}
head(Train)
```

```{r}
skim_without_charts(Train)
```

Now, let’s check whether the 'Train' data is a NULL object with the `is.null()` function.  
```{r}
is.null(Train)
```
The `is.null()` function returns FALSE, which means that the data drame is not a NULL object. 


Let's see if there's any duplicates in the 'Train' data frame. 
```{r}
sum(duplicated(Train))
```
There is no duplicated data in the data frame. We can now dove into the analysing the data. 

## Summary Statistics of 'Train' Data:

```{r}
Train %>% 
  select(Warehouse_block,
         Mode_of_Shipment,
         Customer_care_calls, 
         Customer_rating,
         Cost_of_the_Product,
         Prior_purchases,
         Gender,
         Discount_offered) %>% 
  summary()
```

## Exploratory Visualizations

I'll first rename the 'Reached.on.Time_Y.N' column to 'Reached_On_Time' to make it easier when plotting the visualizations. 
```{r}
Train <- 
  Train %>%
rename(Reached_On_Time = Reached.on.Time_Y.N)
```


Then, I used the `mutate()` function to create a new column 'Received_On_Time' to modify the 'Reached_On_Time' column, where 1 is No and 0 is Yes. 
```{r}
mutate(Train, Received_On_Time = ifelse(Train$Reached_On_Time == "0","Yes","No"))
Train <- Train %>%
  mutate(Train, Received_On_Time = ifelse(Train$Reached_On_Time == "0","Yes","No"))
```


### 1) Relation Between Customer Rating over Product Arrived on Time 

I would like to identify if the product will reached on time for customers who gave the company good ratings than those who gave poor ratings. 
In this case, 1 is the lowest, while 5 is the highest. Hence, I will create a new summarized table where I will classify the 
Customer Rating into "Good Rating" and "Poor Rating" for the analysis. 

```{r}
min(Train$Customer_rating)
max(Train$Customer_rating)
```
```{r}
Rating <- 
  Train %>% 
   summarize(
    Customer_Rating = factor(case_when(
      Customer_rating >= 3 ~ "Good Rating",
      Customer_rating <3 ~ "Poor Rating",
      ), levels = c("Good Rating", "Poor Rating")), 
      Received_On_Time, .groups="drop") %>% 
      drop_na()
head(Rating)
```


#### Visualization

I will create a barchart that represents the Product Reached on Time based on the two categories of Customer Rating. 
It is faceted by two categories, where 'No' indicates Product Did Not Reached on Time and 'Yes' indicates Product Reached on Time.
  
```{r}
ggplot(data = Rating)+
  geom_bar(mapping= aes(x= Customer_Rating, fill= Customer_Rating)) +
  facet_wrap(~ Received_On_Time) +
  theme_linedraw()+
  xlab ("Customer Rating") + 
  ylab ("Count of Shipments") +
  scale_fill_discrete(name = "Customer Rating", labels = c("Good Rating", "Poor Rating")) +  
labs(title = "Customer Rating over Product Reached on Time")
```

Analysis: 

From the bar chart, we can identify that most products did not reached on time. This barchart also shows that most customers who 
received their product on time are customers who gave good ratings. That being said, most customers who did not received their product 
on time are also those who gave good ratings. Hence, this reveals that there is no correlation between customer ratings and product reached on time.



### 2) Relation Between Customer Care Calls over Cost of The Product

To identify the relationship between Customer Care Calls and Cost of Product, I will create a new summarized table with the 
`summarise()` and `case_when()` function where I will classify the Cost of the Product into more easily interpretable categories for the analysis.

```{r}
min(Train$Cost_of_the_Product)
max(Train$Cost_of_the_Product)
```
```{r}
Cost <- Train %>% 
   summarise(
    Cost_of_the_Product = factor(case_when(
      Cost_of_the_Product >= 96 & Cost_of_the_Product <= 100 ~ "Below $100",
      Cost_of_the_Product >= 101 & Cost_of_the_Product <= 200 ~ "$101 > & < $200",
      Cost_of_the_Product >= 201 & Cost_of_the_Product <= 300 ~ "$201 > & < $300",
      Cost_of_the_Product >= 301 & Cost_of_the_Product <= 310 ~ "Above $300",
      ), levels = c("Below $100", "$101 > & < $200", "$201 > & < $300", "Above $300")), 
      Customer_care_calls, .groups="drop") %>% 
      drop_na()
head(Cost)
```


#### Visualization

For the visualization, I will create a barchart that represents the customer care calls based on the cost of the product and then 
it is faceted by the four categories of cost of the product. By doing this, I'm able to identify the relationship between the customer care calls
and the cost of the product.

```{r}
ggplot(data = Cost,aes(Cost_of_the_Product,Customer_care_calls,fill = Cost_of_the_Product)) +
    geom_col() +
    facet_wrap(~Customer_care_calls)+
    theme_linedraw()+
    xlab ("Product Cost") + 
    ylab ("Count of Calls") +
    scale_fill_discrete(name = "Product Cost", labels = c("Below $100", "$101 > & < $200", "$201 > & < $300", "Above $300")) +
    labs(title="Customer Care Calls over Cost of the Product") +
    theme(legend.position="right", text = element_text(size = 10), plot.title = element_text(hjust = 1), axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
```

Analysis: 

Here, we can see that customers who bought products that costs below $100 did not made calls to make enquiry for their shipment.

Customers who purchased products that costs between $100 to $300 start to made 2 to 7 calls to enquire their shipment, 
in which the average number of customer care calls made by these customers is 4. 
Interesting to note, the number of customer care calls made by these customers start to decrease after 4 calls.  

Customer who purchase products that costs above $300 made 6 to 7 customer care calls to make enquiry for their shipment. 
Reason being is that customer might worry that they will lost shipment that worth more than $300. 
In short, as the cost of the product increases, the number of customer care calls made by customer for shipment enquiry will also increase. 



### 3) Discount Offered over Product Importance 

I will create a new summarized table named "Discount" with the `summarise()` and `case_when()` function where I will classify the Discount Offered into more easily interpretable categories for the analysis.

```{r}
Discount <- Train %>% 
   summarise(
    Discount_Offered = factor(case_when(
      Discount_offered <= 9  ~ "Min $10 off",
      Discount_offered >= 10 & Discount_offered <= 19 ~ "$10 to $19 off",
      Discount_offered >= 20 & Discount_offered <= 29 ~ "$20 to $29 off",
      Discount_offered >= 30 & Discount_offered <= 39 ~ "$30 to $39 off",
      Discount_offered >= 40 & Discount_offered <= 49 ~ "$40 to $49 off",
      Discount_offered >= 50 & Discount_offered <= 59 ~ "$50 t0 $59 off",
      Discount_offered >= 60 ~ "Up to $60 off"
      ), levels = c("Min $10 off", "$10 to $19 off", "$20 to $29 off", "$30 to $39 off","$40 to $49 off","$50 to $59 off", "Up to $60 off")), 
      Product_importance, .groups="drop") %>% 
      drop_na()
```


```{r}
dsct <- table(Discount$Discount_Offered,Discount$Product_importance)
dsct
dsct <- as.data.frame(dsct)
```


#### Visualization

I will create three pie charts where I can identify the percentage of discount offered over three types of product importance. 
By doing this, I can find out the largest category of discount offered based on the importance of the product. 


```{r Discount offered for High Importance Product }
discount_high <- c(613, 105, 55, 47, 49, 0, 34)
labels <- c("Min $10 off", "$10 to $19 off", "$20 to $29 off", "$30 to $39 off","$40 to $49 off","$50 to $59 off", "Up to $60 off")
piepercent_high <- paste0(round( discount_high / sum(discount_high) * 100, 1), "%")
pie(discount_high, piepercent_high, cex = 0.8, 
main =" Discount offered for High Importance Product",
      col = rainbow(length(discount_high)))
      legend("left", c("Min $10 off", "$10 to $19 off", "$20 to $29 off", "$30 to $39 off","$40 to $49 off","$50 to $59 off", "Up to $60 off"), cex = 0.7,
      fill = rainbow(length(discount_high))
      ) 
      
```

Analysis:

The top 3 categories of Discount Offered for "High" importance product is :

1) Min $10 off (67.9%)
2) $10 to $19 off (11.6%) 
3) $20 to $29 off (6.1%) 

Besides, the pie chart also shows that there is no seller who offered $50 to $59 off (0%) for "High" importance product. 


```{r Discount offered for Low Importance Product}
discount_low <- c(3638, 636, 219, 215, 242, 0, 125)
labels <- c("Min $10 off", "$10 to $19 off", "$20 to $29 off", "$30 to $39 off","$40 to $49 off","$50 to $59 off", "Up to $60 off")
piepercent_low <- paste0(round( discount_low / sum(discount_low) * 100, 1), "%")
pie(discount_low, piepercent_low, cex = 0.8, 
main ="Discount offered for Low Importance Product",
      col = rainbow(length(discount_low)))
      legend("left", c("Min $10 off", "$10 to $19 off", "$20 to $29 off", "$30 to $39 off","$40 to $49 off","$50 to $59 off", "Up to $60 off"), cex = 0.7,
      fill = rainbow(length(discount_low))
      ) 
```

Analysis:

The top 3 categories of Discount Offered for "Low" importance product is :
1) Min $10 off (71.7%) 
2) $10 to $19 off (12.5%)
3) $40 to $49 off (4.8%) 

Similarly, there is no seller who offered $50 to $59 off (0%) for "Low" importance product.


```{r Discount offered for Medium Importance Product}
discount_medium <- c(3241, 557, 191, 202, 216, 0, 128)
labels <- c("Min $10 off", "$10 to $19 off", "$20 to $29 off", "$30 to $39 off","$40 to $49 off","$50 to $59 off", "Up to $60 off")
piepercent_medium <- paste0(round( discount_medium / sum(discount_medium) * 100, 1), "%")
pie(discount_medium, piepercent_medium, cex = 0.8, 
main ="Discount offered for Medium Importance Product",
      col = rainbow(length(discount_medium)))
      legend("left", c("Min $10 off", "$10 to $19 off", "$20 to $29 off", "$30 to $39 off","$40 to $49 off","$50 to $59 off", "Up to $60 off"), cex = 0.7,
      fill = rainbow(length(discount_medium))
      ) 
```

Analysis:

The top 3 categories of Discount Offered for "Medium" importance product is :
1) Min $10 off - 71.5% 
2) $10 to $19 off - 12.3% 
3) $40 to $49 off - 4.8% 

Just like "High" and "Low" importance product, sellers did not offer $50 to $59 off(0%) for "Medium" importance product.  



### 4)  Which Mode of Shipments is the most desirable?

#### Visualization

I will create a barchart that represents the Product Reached On Time based on Mode of Shipment. 

```{r}
ggplot(data = Train, aes(Received_On_Time)) + 
  geom_bar(aes(fill = Mode_of_Shipment)) +
  theme_linedraw() +
  xlab ("Reached on Time") + 
  ylab ("Count of Shipments") +
  scale_fill_discrete(name = "Mode of Shipment", labels = c("Flight", "Road", "Ship")) + labs(title = "Mode of Shipping Distribution over Reached in Time")
```

Analysis: 

According to the barchart above, we can see that most sellers prefer to ship their products to customers via ship shipment. 
Besides, by comparing whether the product has reached on time, it seems that there are more shipments did not reached on time that shipments that has reached on time. 
For shipment that reached on time, mostly were transported by ship. Hence, Ship appears to be the fastest and most desirable mode of transportation. 


### 5)  Which Gender Made the Most Prior Purchase? 

I will create a new table, "PriorP" for easier analysis. 

```{r}
PriorP <- table(Train$Gender,Train$Prior_purchases)
PriorP
PriorP <- as.data.frame(PriorP)
```
#### Visualization

I will create a barchart that represents the Prior Purchases based on Gender. 

```{r}
ggplot(data = PriorP, aes(Var1, Freq)) + 
  geom_col(aes(fill = Var2)) +
  theme_linedraw() +
  xlab ("Prior Purchases") + 
  ylab ("Count of Prior Purchases") +
  scale_fill_discrete(name = "Gender", labels = c("2","3","4","5","6","7","8","10")) + labs(title = "Prior Purchases by Gender")
```

Analysis: 

According to the graph, Female customers made more prior purchase than Male customer, but the prior purchases of both genders are almost the same.
Hence, we can say that gender does not influence the amount of prior purchase. 




