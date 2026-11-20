# Swiggy CA1 — Set B Solution

**Subject:** Data Engineering and Predictive Analytics  
**Case:** Swiggy  
**Tools:** Microsoft SQL Server (MSSQL), Python, pandas  
**Dataset:** `CA1_Swiggy_Synthetic_Dataset.xlsx` (synthetic educational data)

This file documents the submitted solution and key outputs for all 10 questions. The original assessment uses four sheets: Customers, Restaurants, Delivery Partners and Orders.

## Dataset loading

The four sheets are loaded into pandas as `customers`, `restaurants`, `partners` and `orders`. The loaded shapes are:

| Table | Rows | Columns |
|---|---:|---:|
| Customers | 260 | 5 |
| Restaurants | 70 | 5 |
| Delivery Partners | 100 | 4 |
| Orders | 900 | 12 |

## Question 1 — MSSQL schema

Four tables are designed with primary keys:
- Customers: `CustomerID`
- Restaurants: `RestaurantID`
- DeliveryPartners: `PartnerID`
- Orders: `OrderID`

The Orders table has foreign keys to Customers, Restaurants and DeliveryPartners. Its `OrderStatus` CHECK constraint permits only `Delivered` and `Cancelled`. IDs use VARCHAR(10), dates use DATE, monetary values use DECIMAL(10,2), minutes and age use INT, and ratings use DECIMAL(2,1).

## Question 2 — Delivery performance by city

Delivered orders only; late means `ActualMinutes > PromisedMinutes`.

| City | Delivered orders | Avg promised min | Avg actual min | Late rate (%) |
|---|---:|---:|---:|---:|
| Chennai | 172 | 32.85 | 36.20 | 61.05 |
| Delhi NCR | 146 | 31.99 | 35.06 | 60.96 |
| Pune | 174 | 32.44 | 35.04 | 60.92 |
| Mumbai | 110 | 32.32 | 34.63 | 57.27 |
| Bengaluru | 119 | 32.82 | 35.59 | 56.30 |
| Hyderabad | 96 | 32.03 | 34.96 | 53.13 |

There are 817 delivered orders. Chennai has the highest late rate; Hyderabad has the lowest.

## Question 3 — Top five restaurants by platform commission

NetFoodValue = OrderValue − DiscountAmount. PlatformCommission = NetFoodValue × CommissionPct.

| Restaurant | City | Cuisine | Delivered orders | Net food value | Commission |
|---|---|---|---:|---:|---:|
| R050 | Bengaluru | Fast Food | 26 | 19,815 | 4,953.75 |
| R034 | Bengaluru | Chinese | 27 | 18,618 | 4,654.50 |
| R014 | Chennai | South Indian | 19 | 18,104 | 3,982.88 |
| R031 | Chennai | Chinese | 17 | 13,459 | 3,364.75 |
| R054 | Bengaluru | Chinese | 21 | 16,571 | 3,314.20 |

## Question 4 — Delivery partners above their city average

Delivered orders only; partners require at least five delivered orders. CityPartnerAverageMinutes is the average of eligible partner averages in that city.

- 84 partners have at least five delivered orders.
- 43 are above their city partner average.
- Largest excess: DP053 (Hyderabad), 42.20 minutes vs 34.80, an excess of 7.40 minutes.
- Next: DP058 (Chennai), excess 7.11 minutes; DP060 (Bengaluru), excess 5.54 minutes.

## Question 5 — Merge integrity

Orders are left-merged with Customers, Restaurants and DeliveryPartners using many-to-one validation.

- Orders before merges: 900.
- Row counts after each merge: 900 → 900 → 900.
- Unmatched CustomerID, RestaurantID and PartnerID: 0 each.
- Duplicate OrderID after merges: 0.
- All 900 orders have matching city values across the three master tables.

## Question 6 — Missing-value treatment

| Field | Missing before | Treatment | Missing after |
|---|---:|---|---:|
| CustomerRating | 127 | Keep numeric value missing; create “Not Rated” reporting category | 127 numeric; 0 category |
| Membership | 607 | Replace missing with “None” | 0 |
| Age | 100 | City median, then overall median fallback | 0 |

The overall median fallback was not needed because every city had a median.

## Question 7 — Feature engineering

Created:
- `DelayMinutes = max(ActualMinutes - PromisedMinutes, 0)`
- `NetFoodValue = OrderValue - DiscountAmount`
- `PlatformCommission = NetFoodValue × CommissionPct`
- `DeliveryPerformanceBand`: On Time, 1-10 Late, 11-20 Late, Over 20 Late.

Band counts across all 900 orders:

| Band | Orders |
|---|---:|
| 1-10 Late | 372 |
| On Time | 364 |
| 11-20 Late | 140 |
| Over 20 Late | 24 |

## Question 8 — Restaurant performance

54 restaurants have at least eight delivered orders. The top 10 are ranked by commission descending, then average customer rating descending, then RestaurantID ascending.

The leading restaurant is R050 (Bengaluru, Fast Food): 26 delivered orders, average rating 3.39, late rate 46.15%, and commission 4,953.75. R054 and R037 in the top 10 have high late rates (71.43% and 73.68%).

## Question 9 — City-cuisine intervention

Selected segment: **Delhi NCR — South Indian**.

- Delivered orders: 72
- Late deliveries: 46
- Late rate: 63.89%
- Average rating: 3.23
- Overall delivered-order late rate: 58.87%
- Overall average rating: 3.41

It has the highest number of late deliveries among eligible segments (at least 20 delivered orders), so it is selected under the stated rule.

## Question 10 — Data quality audit

All specified audit-rule violation counts are zero:
- Duplicate OrderID and missing order/master keys
- Unmatched foreign keys
- Negative OrderValue, DeliveryFee or DiscountAmount
- ActualMinutes or PromisedMinutes less than or equal to zero
- Ratings outside 1–5
- OrderStatus outside Delivered/Cancelled

Additional observations: 127 ratings are missing (14.11%); 71 of 83 cancelled orders still carry a rating. DiscountAmount greater than OrderValue: 0.

**Conclusion in the submitted analysis:** the dataset passes every specified audit rule and is suitable for operational analysis. Missing ratings and ratings attached to cancelled orders should be interpreted carefully.

---

*Academic case study using synthetic data only. This summary reflects the submitted solution; it is not a claim about Swiggy's actual operations. The source submission also contains the full MSSQL and Python/pandas code and detailed question-wise explanations.*