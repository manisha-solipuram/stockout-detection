# Stockout Detection Tool

An **unsupervised** algorithm that analyzes historical sales data to identify past periods of low stock or stockouts for scenarios where inventory data is not available. This method requires **no training data**. It flags anomalies directly from sales patterns and only using historical sales data. 

## Key Features  
- 🚨 **Detects Past Stockouts**: Identifies periods where sales drops suggest insufficient inventory (no external stock data needed).  
- ⚡ **Future-Proof Logic**: The same rules can be adapted to build real-time alerts for stockouts as they happen.  
- 📊 **Input Agnostic**: Works with just sales/units data (`product_id`, `store_id`, `date`, `gross_sales`, `gross_units`).
 
<img width="816" alt="Screenshot 2025-04-03 at 8 21 09 AM" src="https://github.com/user-attachments/assets/98cd3ec8-620a-4e59-902e-cd01a4fdbc0f" />

### How The Algorithm Works  
The algorithm marks time windows with abnormal sales dips as potential low stock or stockout periods. 

#### Definitions
A stockout or period of low stock is defined as a continuous period of low or no gross units sold as compared to a calculated baseline value of gross units sold during periods of active sales.  

#### Input Data
The input file sales.csv contains the following fields for a set of 5 products, 1500 stores, and dates ranging from 2024-06-01 to 2024-12-31:

| product_id | store_id | metric_date | gross_sales | gross_units |
|----------|----------|----------|----------|----------|
| product_0 | store_1 | 2024-06-01 | 3.50 | 1 |

#### Algorithm 
1. Calculate a rolling average value for each data record.
3. Calculate a baseline sales level from days where gross units sold are non zero.
4. Create a low sales flag for each data record. The flag is marked as true if the rolling average is less than (baseline x threshold ratio) or gross units sold is 0.
5. Group consecutive low-sales days.
6. If there are periods of consecutive low-sales flagged days of 10 or more days, the start and end date are logged and included in the final dataset which outputs datasets of periods that the product was low stock or missing from a product’s shelf.

<pre>
# Flag low sales periods
ts['low_sales_flag'] = (ts['rolling_avg'] < baseline * threshold_ratio) | \(ts['gross_units'] == 0)
    
# Group consecutive low-sales days
ts['period_id'] = (~ts['low_sales_flag']).cumsum() * ts['low_sales_flag']
</pre>

#### Output
The program generates product_missing_dataset.csv, which contains the start and end dates of stockout periods for a given store and product.

| product_id | store_id | missing_start_date | missing_end_date | 
|----------|----------|----------|----------|
| product_0 | store_1 | 2024-07-01 | 2024-07-10 | 

### Use Cases  
- **Postmortem Analysis**: Understand past inventory shortages.  
- **Real-Time Potential**: Logic can be repurposed to trigger alerts in live systems (e.g., "Stockout likely TODAY at Store X").

### Deployments 
This notebook can be converted into a .py script for production deployment. A configuration file (e.g., params.yaml) can be introduced to manage key parameters such as:

🚨**Threshold ratios** for inventory alerts

🚨**Number of consecutive low-sales days** to trigger stock warnings

### Additional Considerations

Some periods of low or no sales can be due to seasonality of a product or other external factors. This algorithm flags any potential low stock or stockout periods. In theory, it would be good to give a retailer a flag regardless so they can check and replenish stock if needed. If there is information on seasonality or natural sales trends of a specific product, that can be factored into this algorithm for further tuning and reduction of false flags. 
