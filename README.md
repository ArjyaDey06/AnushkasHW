# Financial Consumer Complaints Dashboard
**Program & Course:** MBA, Business Intelligence and Data Visualization (Semester III)  
**Project:** Financial Consumer Complaints Interactive Dashboard  
**Tool Used:** Microsoft Power BI Desktop  
**Author:** Anushka Bendre  
**Primary Deliverables:** `AnushkaBendre.pbix` (Power BI Report), Final Dashboard Screenshot

---

## Executive Summary & Final KPI Results

This interactive Power BI dashboard analyzes **75,513 consumer complaints** from Cumulus Financial across products, channels, issues, geography, and response outcomes.

| Metric / KPI Card | Calculated Result | Formula / Calculation Logic |
|---|---|---|
| **Total Complaints** | **75,513** | Total row count across the dataset |
| **Timely Response Rate** | **98.05%** | Complaints with Timely Response = "Yes" divided by Total |
| **In Progress** | **283** | Complaints currently with status "In progress" |
| **Disputed Rate** | **19.09%** | Disputed "Yes" divided by total valid responses ("Yes" + "No") |
| **Resolved at No Cost** | **82.58%** | Complaints closed with explanation, non-monetary relief, or without relief divided by Total |

---

## 1. Datasets Used

1. **`Financial Consumer Complaints.csv`**
   - Contains 75,513 consumer complaint records.
   - Key attributes: `Complaint ID`, `Date Sumbited`, `Product`, `Issue`, `Company`, `State`, `Submitted via`, `Company response to consumer`, `Timely response?`, `Consumer disputed?`.

2. **`Hexagon.xlsx`**
   - Provides coordinate mapping for the 50 US States + DC.
   - Key attributes: `State` (full state name), `Abbreviation` (2-letter state code), `Row` (0 to 8), `Column` (0.5 to 12.0).
   - Used to generate the geographical tile map grid for US states.

---

## 2. Step-by-Step Implementation Guide

### Step 1: Data Ingestion & Preprocessing
1. **Load CSV:**
   - Go to **Home → Get Data → Text/CSV** and select `Financial Consumer Complaints.csv`.
   - Click **Transform Data** to open the Power Query Editor.
2. **Date Locale Handling (Crucial):**
   - The dates are formatted in US standard (`MM/DD/YYYY`, e.g., `2/24/18`).
   - Right-click the **`Date Sumbited`** column header $\rightarrow$ **Change Type → Using Locale...**
   - Select **Data Type: Date** and **Locale: English (United States)**.
   - Repeat for **`Date Received`**.
3. **Load Hexagon Data:**
   - In Power BI, go to **Home → Get Data → Excel workbook** $\rightarrow$ select `Hexagon.xlsx` $\rightarrow$ pick `Sheet1` (rename to `Hexagon`).
4. Click **Close & Apply**.

---

### Step 2: Data Modeling & Relationships
In **Model view** (left-hand sidebar, 3rd icon):
- Establish an active **One-to-Many (1:\*)** relationship between:
  - **`Hexagon[Abbreviation]`** (1 side) $\longleftrightarrow$ **`Complaints[State]`** (* side)
  - Cardinality: **Many to one (*:1)**
  - Cross-filter direction: **Single**

*(Note: Do not link `Hexagon[State]` to `Complaints[State]`, as Hexagon contains full state names while Complaints contains 2-letter codes).*

---

### Step 3: DAX Measures Creation
Create a dedicated measures container table (`_Measures` via **Home → Enter data**), then create the following DAX measures:

#### 1. Total Complaints
```DAX
Total Complaints = COUNTROWS(Complaints)
```
*Format: Whole number with comma `,` (e.g., 75,513)*

#### 2. Timely Response Rate
```DAX
Timely Response Rate = 
DIVIDE(
    CALCULATE([Total Complaints], Complaints[Timely response?] = "Yes"),
    [Total Complaints]
)
```
*Format: Percentage, 2 decimal places (`98.05%`)*

#### 3. In Progress
```DAX
In Progress = 
CALCULATE(
    [Total Complaints], 
    Complaints[Company response to consumer] = "In progress"
) + 0
```
*Format: Whole number (`283`)*

#### 4. Disputed Rate
```DAX
Disputed Rate = 
VAR DisputedYes = CALCULATE([Total Complaints], Complaints[Consumer disputed?] = "Yes")
VAR DisputedNo = CALCULATE([Total Complaints], Complaints[Consumer disputed?] = "No")
RETURN
DIVIDE(DisputedYes, DisputedYes + DisputedNo)
```
*Format: Percentage, 2 decimal places (`19.09%`)*

#### 5. Resolved at No Cost
```DAX
Resolved at No Cost = 
DIVIDE(
    CALCULATE(
        [Total Complaints],
        Complaints[Company response to consumer] IN {
            "Closed with explanation",
            "Closed with non-monetary relief",
            "Closed without relief"
        }
    ),
    [Total Complaints]
)
```
*Format: Percentage, 2 decimal places (`82.58%`)*

---

### Step 4: Dashboard Layout & Visualizations

The dashboard is structured on a single page following a clean, card-based UI grid:

#### Top Header & KPI Bar
- **Header Text Box:**
  - Title: **Financial Consumer Complaints Dashboard** (Segoe UI Bold, 20pt)
  - Subtitle: `Cumulus Financial | Complaints Overview` (10pt, slate grey)
- **5 KPI Cards (New Card / Card Visual):**
  - Arranged horizontally across the top.
  - Card 1: `Total Complaints` (76K / 75,513)
  - Card 2: `Timely Response Rate` (98.05%)
  - Card 3: `In Progress` (283)
  - Card 4: `Disputed Rate` (19.09%)
  - Card 5: `Resolved at No Cost` (82.58%)
  - Styled with soft drop shadows and clean white rounded card backgrounds.

#### Visual 1: Complaints by Issue (Top 15)
- **Visual Type:** Clustered horizontal bar chart
- **Y-axis:** `Complaints[Issue]`
- **X-axis:** `_Measures[Total Complaints]`
- **Filter Applied:** Filters pane $\rightarrow$ `Issue` $\rightarrow$ **Top N = 15** by `Total Complaints`.
- **Formatting:** Data labels enabled, horizontal X-axis hidden for a cleaner look.

#### Visual 2: Complaints by State (Hexagon Grid Map)
- **Visual Type:** Scatter Chart (utilizing `Hexagon.xlsx` coordinates)
- **Values:** `Hexagon[Abbreviation]`
- **X-axis:** `Average of Hexagon[Column]`
- **Y-axis:** `Average of Hexagon[Row]`
- **Formatting Details:**
  - `Y-axis` $\rightarrow$ **Inverted range = ON** (places Row 0 at top)
  - `X-axis` & `Y-axis` Values & Titles = **OFF**
  - `Gridlines` = **OFF**
  - `Category labels` = **ON** (displays state codes AK, CA, TX, NY, FL inside the map)
  - `Markers Color` = Conditional formatting gradient based on `_Measures[Total Complaints]` (light blue to deep navy `#1F3A93`).

#### Visual 3: Complaints by Media (Submission Channels)
- **Visual Type:** Clustered horizontal bar chart
- **Y-axis:** `Complaints[Submitted via]` (Web, Referral, Phone, Postal mail, Fax, Email)
- **X-axis:** `_Measures[Total Complaints]`
- **Formatting:** Sorted descending, data labels enabled.

#### Visual 4: Complaints by Products
- **Visual Type:** Clustered horizontal bar chart
- **Y-axis:** `Complaints[Product]`
- **X-axis:** `_Measures[Total Complaints]`
- **Formatting:** Sorted descending (Credit card, Checking/savings, Mortgage, etc.), data labels enabled.

#### Visual 5: Customer Disputed Analysis
- **Visual Type:** Donut chart
- **Legend:** `Complaints[Consumer disputed?]` (N/A, No, Yes)
- **Values:** `_Measures[Total Complaints]`
- **Center Overlay:** Total Complaints KPI card centered inside the donut ring.

#### Visual 6: Interactive Date Range Slicer
- **Visual Type:** Slicer
- **Field:** `Complaints[Date Sumbited]`
- **Type:** Date range slider (`Between` / range selector)
- **Interactivity:** Dynamically recalculates all 5 KPI cards and all charts upon date selection.

---

## 3. How to Open & View the `.pbix` File

### Why Windows / WhatsApp might show it as a `.zip` file:
A `.pbix` file is technically a compressed package container (containing `DataModel`, `Report`, and `Metadata`). Some messaging apps (like WhatsApp) or browsers inspect the binary structure and label it as a compressed zip folder.

### Instructions to open properly:
1. **Method 1 (Direct Open via Power BI):**
   - Open **Microsoft Power BI Desktop**.
   - Click **File → Open report → Browse reports**.
   - Select `AnushkaBendre.pbix`.
2. **Method 2 (Open With):**
   - Right-click `AnushkaBendre.pbix` on your computer.
   - Select **Open with... → Power BI Desktop**.
3. **If WhatsApp saved it with a `.zip` extension:**
   - Right-click the downloaded file $\rightarrow$ select **Rename**.
   - Change `.zip` at the end to **`.pbix`** (e.g., `AnushkaBendre.pbix`).
   - Confirm the extension change prompt, then double-click to open.

---

## 4. Submission Checklist

- [x] Accurate DAX calculations for all 5 required KPI measures.
- [x] Top 15 Complaints by Issue horizontal bar chart sorted descending.
- [x] Geographical US Hexagon map distribution using `Hexagon.xlsx`.
- [x] Complaints by Media (Submission channels) breakdown.
- [x] Complaints by Products sorted descending.
- [x] Customer Disputed donut chart breakdown.
- [x] Interactive Date range slicer linked to all visuals.
- [x] Single-page responsive layout matching assignment reference.
- [x] `.pbix` file saved and verified (`AnushkaBendre.pbix`).
- [x] High-resolution dashboard screenshot ready for submission.
