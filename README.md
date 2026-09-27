# Enterprise Inventory Architecture: Non-Additive Time-State Modeling
### Advanced DAX Context Transition, Star Schema Normalization & Inventory Asset Valuation

## 📊 Business Problem & Analytics Objective
In supply chain and corporate retail analytics, tracking inventory assets introduces a core architectural challenge: inventory levels are **semi-additive**. While stock units can be mathematically aggregated across physical locations or product categories, they **cannot be summed across the time dimension** (e.g., holding 50 units on Monday and 50 units on Tuesday represents a sustained balance of 50 units, not a cumulative sum of 100). 

Standard aggregate functions fail in this scenario, returning heavily distorted evaluations at summary levels. Furthermore, real-world operational realities—such as holiday closures or missing ledger entries—create data density gaps that break naive time-intelligence functions.

**The core architectural challenge solved:** Designing a resilient time-state analytics engine within Power BI that dynamically isolates the absolute latest valid closing balance for any user-selected time frame, evaluating asset value at the grain's lowest intersection without incurring storage engine latency.

---

## 🛠️ Data Engineering & Star Schema Architecture
To isolate computational logic and achieve rapid query execution over daily records, the raw flat ledger log was normalized into a production-grade **Star Schema** utilizing an optimized **Periodic Snapshot Fact Pattern**:

*   **Fact_Inventory_Snapshot:** Captures daily snapshot balances. The raw numerical arrays map physical counts and item cost baselines directly into the core matrix.
*   **Dim_Date (M-Code Engine):** A continuous, dynamic enterprise calendar generated via native Power Query M-code. Automatic time intelligence hierarchies were explicitly deactivated to reduce storage dictionary footprint and avoid unoptimized background scans.
*   **Dim_Product:** Inventory master listing mapping product identifiers and categories.
*   **Dim_Store:** Operational location tracking dimension to evaluate regional stocking density.

---

## 🧠 Advanced Analytical Logic (DAX Framework)

### 1. Resilient Non-Additive Time-State Isolation
To ensure absolute data density across operational gaps, the engine avoids unoptimized native closing functions. Instead, it utilizes `LASTNONBLANK` to force an explicit context transition over the date array, scanning backward to isolate the latest date containing an actual count:

```dax
Closing Stock Quantity = 
VAR LastDateWithData = 
    CALCULATE(
        MAX(Fact_Inventory_Snapshot[Date]),
        LASTNONBLANK(
            Dim_Date[Date],
            CALCULATE(COUNTROWS(Fact_Inventory_Snapshot))
        )
    )
RETURN
CALCULATE(
    SUM(Fact_Inventory_Snapshot[Inventory Level]),
    Dim_Date[Date] = LastDateWithData
)
```

### 2. Low-Grain Capital Asset Valuation ($)
Inventory capital valuation requires multiplying the closing physical volume by the localized unit cost. To prevent mathematical skew at aggregate row headers, this calculation is forced down to the lowest dimensional grain using a virtual iteration loop before compiling summary values upward:

```dax
Ending Inventory Value = 
SUMX(
    SUMMARIZE(
        Fact_Inventory_Snapshot, 
        Dim_Product[Product ID], 
        Dim_Store[Store ID]
    ),
    [Closing Stock Quantity] * CALCULATE(MAX(Fact_Inventory_Snapshot[Price]))
)
```

---

## 🎨 Executive UI/UX Design & Architecture
The presentation layer bypasses cluttered charts to deliver a single, high-density matrix visual tailored for operational leaders:
1.  **Hierarchical Time Nesting:** Integrates full calendar structures, nesting months chronologically directly beneath their parent fiscal years.
2.  **Context-Aware Aggregations:** Displays correct non-additive balances across time intervals while simultaneously computing clean, additive product category values.

---

## 🚀 Corporate Consulting & Inquiries
I specialize in engineering high-performance business intelligence architecture, optimizing complex data models, and establishing robust enterprise design patterns for scaling organizations.

*   **Connect on LinkedIn:** www.linkedin.com/in/guhamba-d-72788619
*   **Corporate Inquiries & Architecture Strategy:** guhamba.d@gmail.com
