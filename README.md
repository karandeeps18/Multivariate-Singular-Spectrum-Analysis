# CDS Term Structure Data Completion

## Objective

This project addresses missing data in historical **Credit Default Swap (CDS) term structure spreads** across multiple maturities. This project addresses a critical challenge in credit risk analysis; handling incomplete Credit Default Swap (CDS) spread data. CDS spreads are essential market indicators that reflect the creditworthiness of corporate entities, but real-world datasets often contain missing values due to illiquid trading periods, data collection issues, or market disruptions.
The objective is to **fill missing CDS spreads in a disciplined and transparent manner**, while **preserving observed market data and economic structure**. 

The project is to understand how to deal with missing spread data and is borrowed from Stein, Harvey J. and Zhang, Yan, Big Data's Dirty Secret (June 29, 2018). Available at SSRN: https://ssrn.com/abstract=3205524 or http://dx.doi.org/10.2139/ssrn.3205524

---

## Data Overview

The dataset contain Citi CDS spreads across the following maturities from 2005 to 2025:

- 6M, 1Y, 2Y, 3Y, 4Y, 5Y, 7Y, 10Y

### Missing Data Summary

| Maturity | Missing Count | Missing % |
|--------|---------------|-----------|
| 6M     | 84            | 36.68%    |
| 1Y     | 37            | 16.16%    |
| 2Y     | 58            | 25.33%    |
| 3Y     | 35            | 15.28%    |
| 4Y     | 59            | 25.76%    |
| 5Y     | 0             | 0.00%     |
| 7Y     | 38            | 16.59%    |
| 10Y    | 36            | 15.72%    |

- Total missing values: **347**
- Overall completeness before filling: **81.06%**

Missing values are concentrated by maturity and time, indicating **structural gaps rather than random noise**.

---

## Methodology 

## Technical Methodology

To capture the time dependent structure in CDS spreads, each maturity’s time series is first transformed into Hankel matrix. A Hankel matrix is constructed so that each diagonal from left to right contains the same value, allowing overlapping segments of the time series to be viewed together. Then it is decomposed using Singular Value Decomposition (SVD). SVD factors the matrix into three components: left singular vectors, singular values, and right singular vectors. Intuitively, this separates the data into a set of orthogonal patterns ranked by importance. The largest singular values correspond to dominant, systematic structures in the data, while smaller singular values capture noise or idiosyncratic variation.

Reconstruction is performed by retaining only the top k singular components (in our dataset k=15). This produces a low-rank approximation of the original Hankel matrix that preserves the dominant dynamics while filtering out noise. By focusing on the most important components, the reconstructed series reflects the underlying term-structure behavior rather than short-term irregularities or missing observations. To ensure consistency with observed market data, the reconstructed values are anchored back to the original scale. This is done using either additive anchoring, which aligns mean levels, or multiplicative anchoring, which preserves proportional relationships. Anchoring ensures that filled values remain comparable to observed spreads and do not introduce level shifts.

Finally, the reconstruction is refined iteratively by increasing the number of retained components k. At each step, convergence is monitored using L1 and L2 norm differences between successive reconstructions. The process stops once improvements stabilize, indicating that the dominant structure has been captured and further components add limited value.

### Key Ideas
- The CDS dataset is treated as a **time × maturity matrix**.
- SVD extracts common patterns shared across maturities and time.
- Missing values are reconstructed using these shared patterns.
- **Observed market data is never modified**.

### Design Principles used for Robustness 
- No forward-filling or simple interpolation
- No averaging across time
- No distortion of observed data points
- Filling is driven by cross-maturity structure

This makes the approach conservative and market-consistent.
---

<img width="1324" height="851" alt="image" src="https://github.com/user-attachments/assets/42ffc6c4-6cec-4fc0-8d37-151fbe092833" />
This plot shows historical CDS spreads across maturities with visible gaps in the data.
Major stress events (2008–2009) appear clearly across all tenors, confirming strong cross-maturity co-movement.
However, missing points break curve continuity and prevent consistent term-structure analysis.
This heatmap highlights when and where CDS data is missing across maturities and time.
Missing values cluster by maturity and period rather than appearing randomly.
Some maturities are unavailable for extended intervals.

## Results
#### Convergence Plot for hole Filling 
<img width="1587" height="819" alt="image" src="https://github.com/user-attachments/assets/3911e2b7-d77e-4b3f-b3ad-02097c16fd8d" />
These plots show how reconstruction stabilizes as more structural components are added.
Error metrics decline and level off, indicating convergence.
No instability or explosive behavior is observed across maturities.

####  Preservation of Observed Data 
<img width="1324" height="851" alt="image" src="https://github.com/user-attachments/assets/692c907e-42df-4dfc-bf4f-c0ba69fd622c" />
This panel shows the same CDS series after filling missing values using the SVD-based method, filled points (highlighted in red) integrate smoothly with observed spreads and follow the surrounding term-structure behavior.
Crisis spikes, regime shifts, and relative levels across maturities are preserved and hence this can be used for downstream applications. 
The bottom pannel shows the difference between original and reconstructed values at points where data was originally observed, errors are effectively zero across all maturities and time periods.
Hence this confirms that observed market data is unchanged by the reconstruction.

---

### 3. Term Structure Stability

- Average CDS levels before and after filling are nearly identical
- Curve slope and shape are preserved
- Historical stress periods remain unchanged


## Conclusion

This project demonstrates that missing CDS term structure data can be filled reliably by leveraging the internal structure of the curve.

This approach:
- Improves data completeness
- Preserves observed market behavior
- Maintains economic realism
- Avoids aggressive assumptions

The result is a **clean and internally consistent CDS dataset** suitable for risk analysis, reporting, and historical studies.

This methodology should be viewed as a **data quality enhancement step**, not a forecasting or pricing model.

---

## Notes and Limitations

The filled values are estimates doesnot not replace real market observations. Additionally the method assumes stable cross-maturity relationships. Thus it is best suited for analytics and reporting, not trade execution

---

