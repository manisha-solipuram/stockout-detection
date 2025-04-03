# Stockout Detection Tool

An **unsupervised** algorithm that analyzes historical sales data to identify past periods of low stock or stockouts. This method requires **no training data**—it flags anomalies directly from sales patterns. 

## Key Features  
- 🚨 **Detects Past Stockouts**: Identifies periods where sales drops suggest insufficient inventory (no external stock data needed).  
- ⚡ **Future-Proof Logic**: The same rules can be adapted to build real-time alerts for stockouts as they happen.  
- 📊 **Input Agnostic**: Works with just sales/units data (`product_id`, `store_id`, `date`, `gross_sales`, `gross_units`).  

### How The Algorithm Works  
The algorithm marks time windows with abnormal sales dips as potential stockouts. 

#### Definitions
A stockout or period of low stock is defined as a continuous period of low or no gross units sold as compared to a median baseline value of gross units sold during periods of active sales.  

#### Input Data
The input file sales.csv contains the following fields:
**product_id**: unique identifier for products (5 distinct IDs).
**store_id**: identifier for stores (1,500 distinct stores).
**metric_date**: date of recorded sales (ranging from 2024-06-01 to 2024-12-31).
**gross_sales & gross_units**: sales revenue and units sold for each product-store-date combination.
EVEN HERE CAN I INCLUDE A SMALL TABLE OF WHAT THE OUTPUT DATASET LOOKS LIKE/HEADER + OTHER INSTEAD OF WRITING IT OUT OR AS AN ADDITION TO WRITING IT OUT.

#### Algorithm 
1. Fill in dataset with 0’s for gross units and gross sales on dates not logged for a product and store to get a continuous dataset to work with.
2. Calculate a rolling average value for each data record.
3. Calculate a baseline sales level from days where gross units sold are non zero.
4. Create a low sales flag for each data record. The flag is marked as true if the rolling average is less than (baseline x threshold ratio) or gross units sold is 0.
5. Group consecutive low-sales days.
6. If there are periods of consecutive low-sales flagged days of 10 or more days, the start and end date are logged and included in the final dataset which outputs datasets of periods that the product was low stock or missing from a product’s shelf.

   SHOULD I MAKE THIS CODE SNIPPET INSTEAD AND JUST EXPLAIN SIMPLY INSTEAD OF WRITING A BIG PARAGRAPH??

#### Output
The program generates product_missing_dataset.csv, which contains the start and end dates of stockout periods for a given store and product.
**product_id**: product affected.
**store_id**: store with stock issues.
**missing_start_date**: start date of the low-stock/stockout period.
**missing_end_date**: end date of the low-stock/stockout period. 

### Use Cases  
- **Postmortem Analysis**: Understand past inventory shortages.  
- **Real-Time Potential**: Logic can be repurposed to trigger alerts in live systems (e.g., "Stockout likely TODAY at Store X").

### Deployments 

### Additional Considerations

Some periods of low or no sales can be due to seasonality of a product or other external factors. This algorithm flags any potential low stock or stockout periods. In theory, it would be good to give a retailer a flag regardless so they can check and replenish stock if needed. If there is information on seasonality or natural sales trends of a specific product, that can be factored into this algorithm for further tuning and reduction of false flags. 
