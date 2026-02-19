<img width="1065" height="711" alt="Screenshot 2026-02-19 185634" src="https://github.com/user-attachments/assets/9362bb68-856d-4db0-88ad-a537977f2b14" />

## Project Overview

### Objective
To create a comprehensive analytical framework for examining global development indicators across multiple domains, enabling data-driven insights into economic growth, labor markets, trade, poverty, environment, health, and technology adoption patterns worldwide.

### Data Source
- **Primary Source**: World Bank Open Data API
- **Base URL**: `https://api.worldbank.org`
- **Data Format**: JSON
- **Coverage**: Global (296 countries/territories)
- **Time Period**: 2016 onwards (post-2015 data)

### Technology Stack
- **Data Extraction**: Python `requests` library
- **Data Processing**: Pandas, NumPy
- **Visualization**: Matplotlib, Seaborn
- **Dashboard**: Power BI (.pbix)
- **Data Storage**: CSV files for Power BI integration

##  Data Extraction Flow

### Phase 1: Country Metadata Extraction

**Process:**
1. **API Endpoint**: `/countries?format=json&per_page=500`
2. **Method**: Single API call to retrieve all 296 countries
3. **Data Retrieved**:
   - Country ID (ISO3 code)
   - ISO2 Code
   - Country Name
   - Region
   - Income Level
   - Capital City
   - Geographic Coordinates (Longitude, Latitude)

**Feature Engineering:**
- Extracted nested dictionary values using lambda functions:
  - `region.value` → `region`
  - `incomeLevel.value` → `incomeLevel`
  - `lendingType.value` → `lendingType`
- Column renaming: `iso2Code` → `country_id` for join operations
- Data cleaning: Removed `adminregion` and `lendingType` columns
- Filtered out aggregate regions (kept only individual countries)

**Output**: Cleaned countries DataFrame with 8 columns

### Phase 2: Indicator Catalog Extraction

**Process:**
1. **API Endpoint**: `/v2/indicator?format=json`
2. **Method**: Paginated extraction across 587 pages
3. **Pagination Strategy**: 
   - 500 indicators per page
   - Loop through pages 1-587
   - Error handling for failed requests
4. **Data Retrieved**:
   - Indicator ID (e.g., `NY.GDP.MKTP.KD.ZG`)
   - Indicator Name (descriptive text)

**Output**: Complete indicator catalog with 29,323 indicators saved to `final_df.csv`

### Phase 3: Domain-Specific Indicator Data Extraction

**Process:**
1. **Categorization**: Indicators grouped into 7 domains
2. **API Endpoint Pattern**: `/countries/all/indicators/{INDICATOR_CODE}?format=json&per_page=1000&page={PAGE}`
3. **Extraction Strategy**:
   - Iterate through each category
   - For each indicator in category, paginate through all available pages
   - Rate limiting: 0.3 second delay between requests
   - Time filter: Only data from 2016 onwards (`year > 2015`)

**Data Structure Extracted:**
- `country.id`: Country identifier
- `country.value`: Country name
- `indicator.id`: Indicator code
- `indicator.value`: Indicator description
- `date`: Year
- `value`: Actual metric value

**Data Volume Collected:**
- Economic Activity & Growth: 4,788 rows
- Labour Market Indicators: 7,182 rows
- Trade & Globalization: 4,788 rows
- Poverty & Inequality: 4,788 rows
- Environmental Indicators: 4,788 rows
- Health Indicators: 31,122 rows (largest dataset)
- Technology Indicators: ~4,788 rows (estimated)

**Total Records**: ~62,244 data points

## Features Considered and Engineered

### Original Features from API

**Country-Level Features:**
- `country_id`: ISO2 country code (join key)
- `country_value`: Country name
- `region`: Geographic region classification
- `incomeLevel`: Economic classification (High income, Upper middle income, Lower middle income, Low income)
- `capitalCity`: Capital city name
- `longitude`: Geographic coordinate
- `latitude`: Geographic coordinate

**Indicator-Level Features:**
- `indicator_id`: World Bank indicator code
- `indicator_name`: Full descriptive name
- `year`: Time dimension (2016+)
- `value`: Actual metric measurement

### Feature Engineering Operations

**1. Nested Dictionary Extraction**
- **Method**: Lambda functions with dictionary access
- **Purpose**: Flatten nested JSON structures
- **Example**: `countries["region"].apply(lambda x: x["value"])`

**2. Data Type Conversion**
- **Year Filtering**: Convert `date` to integer for comparison (`df["year"].astype(int) > 2015`)
- **Purpose**: Ensure proper temporal filtering

**3. Column Standardization**
- **Renaming**: Standardized column names for consistency
  - `country.id` → `country_id`
  - `country.value` → `country_value`
  - `indicator.id` → `indicator_id`
  - `indicator.value` → `indicator_name`
  - `date` → `year`

** Data Merging**
- **Method**: Inner joins on `country_id`
- **Purpose**: Enrich indicator data with country metadata
- **Result**: Each dataset now includes region, income level, and geographic data

** Column Cleanup**
- **Removed**: Redundant columns (`indicator_id`, `name`, `id`)
- **Purpose**: Streamline datasets for analysis and visualization

**6. Data Pivoting**
- **Method**: `pivot()` function for health indicators
- **Purpose**: Transform long format to wide format for correlation analysis
- **Structure**: Rows = (country, year), Columns = indicator names, Values = metric values

### Final Feature Sets by Domain

**Economic Activity & Growth:**
- GDP growth (annual %)
- GDP per capita (current US$)
- Region, Income Level, Geographic coordinates

**Labour Market:**
- Unemployment total (%)
- Unemployment youth total (ages 15–24) (%)
- Labour force, total
- Region, Income Level, Geographic coordinates

**Trade & Globalization:**
- Exports of goods and services (current US$)
- Imports of goods and services (current US$)
- Region, Income Level, Geographic coordinates

**Poverty & Inequality:**
- Poverty headcount ratio at national poverty lines (% of population)
- Gini index (measure of income inequality)
- Region, Income Level, Geographic coordinates

**Environmental:**
- Renewable energy consumption (% of total final energy consumption)
- Forest area (% of land area)
- Region, Income Level, Geographic coordinates

**Health:**
- Life expectancy at birth (years)
- Infant mortality rate
- Access to at least basic water services (% of population)
- Current health expenditure (% of GDP)
- Immunization, DPT (% of children ages 12–23 months)
- Immunization, measles (% of children ages 12–23 months)
- Risk of maternal death
- Deaths from communicable diseases (% of total)
- Tuberculosis incidence (per 100,000 people)
- Births attended by skilled health staff (%)
- Maternal mortality ratio (per 100,000 live births)
- Population ages 65 and above (% of total population)
- HIV incidence rate (per 1,000 uninfected population ages 15–49)
- Region, Income Level, Geographic coordinates

**Technology:**
- Individuals using the Internet (% of population)
- Mobile cellular subscriptions (per 100 people)
- Region, Income Level, Geographic coordinates

---

## Methods and Techniques Applied

### Data Extraction Methods

**1. RESTful API Consumption**
- HTTP GET requests using `requests` library
- JSON response parsing
- Status code validation (200 = success)

**2. Pagination Handling**
- Dynamic page iteration
- Total pages detection from API response metadata
- Break conditions: status code != 200 or no data returned

**3. Rate Limiting**
- `time.sleep(0.3)` between requests
- Purpose: Prevent API throttling and ensure reliable data retrieval

**4. Error Handling**
- Status code checking
- Empty data validation
- Graceful failure with informative messages

### Data Processing Methods

**1. Data Normalization**
- `pd.json_normalize()`: Flatten nested JSON structures
- Column selection and renaming in single operation

**2. Data Filtering**
- Temporal filtering: `df[df["year"].astype(int) > 2015]`
- Purpose: Focus on recent data (2016+)

**3. Data Aggregation**
- `pd.concat()`: Combine multiple DataFrames
- `ignore_index=True`: Reset index after concatenation

**4. Data Merging**
- `pd.merge()` with `how="inner"` join
- Join key: `country_id`
- Purpose: Enrich indicator data with country metadata

**5. Data Transformation**
- `pivot()`: Reshape from long to wide format
- Index: `["country_value", "year"]`
- Columns: `indicator_name`
- Values: `value`

### Statistical Analysis Methods

**1. Correlation Analysis**
- **Method**: `df.corr()` - Pearson correlation coefficient
- **Purpose**: Identify relationships between health indicators
- **Output**: Correlation matrix

**2. Regression Analysis**
- **Method**: `sns.regplot()` - Linear regression with scatter plot
- **Purpose**: Examine relationship between health expenditure and life expectancy
- **Features**: 
  - Scatter plot with transparency (`alpha=0.5`)
  - Regression line overlay
  - Grid for readability

### Data Export Methods

**1. CSV Export**
- `df.to_csv()` for each domain dataset
- Files created:
  - `economic.csv`
  - `labour_market.csv`
  - `trade.csv`
  - `poverty.csv`
  - `environment.csv`
  - `health.csv`
  - `technology.csv`
  - `final_df.csv` (indicator catalog)

**2. Power BI Integration**
- CSV files imported into Power BI
- Enables interactive dashboard creation
- Supports drill-down, filtering, and cross-domain analysis

---

## Analysis Actions Performed

### 5.1 Data Quality Checks

**1. Data Completeness Validation**
- Verified API response status codes
- Checked for empty data arrays
- Validated pagination completeness

**2. Data Structure Validation**
- Confirmed column existence
- Verified data types
- Checked for null values

**3. Data Consistency Checks**
- Validated country_id format
- Ensured year format consistency
- Verified indicator code structure

### Exploratory Data Analysis

**1. Health Indicators Correlation Analysis**
- **Action**: Created correlation matrix for all health indicators
- **Method**: Pivot table → correlation matrix → heatmap visualization
- **Insight**: Identified relationships between health metrics (e.g., life expectancy, mortality rates, health expenditure)

**2. Health Expenditure vs. Life Expectancy Analysis**
- **Action**: Regression analysis between health expenditure (% of GDP) and life expectancy
- **Method**: Scatter plot with regression line
- **Insight**: Examined whether increased health spending correlates with higher life expectancy

### 5.3 Data Preparation for Visualization

**1. Data Reshaping**
- Converted long format to wide format for correlation analysis
- Created pivot tables for multi-dimensional analysis

**2. Data Cleaning**
- Removed redundant columns
- Standardized column names
- Filtered temporal data

**3. Data Enrichment**
- Merged country metadata with indicator data
- Added geographic and economic classification context

## Visualizations Created

### Python-Based Visualizations (Jupyter Notebook)

#### Visualization 1: Health Indicators Correlation Heatmap

**Type**: Correlation Heatmap
**Library**: Seaborn
**Size**: 16x16 inches
**Features**:
- Color scheme: `coolwarm` colormap
- Annotations: Correlation coefficients displayed
- Font size: 14pt for axis labels, bold annotations
- Color bar: Labeled "Correlation"
- Title: "Relationship between various health indicators"

**Purpose**: 
- Identify strong positive/negative correlations between health metrics
- Discover hidden relationships in health data
- Guide feature selection for predictive modeling

**Metrics Analyzed**:
- Life expectancy at birth
- Infant mortality rate
- Health expenditure
- Immunization rates (DPT, Measles)
- Maternal health indicators
- Disease incidence rates
- Water access
- Population demographics

#### Visualization 2: Health Expenditure vs. Life Expectancy Regression Plot

**Type**: Scatter Plot with Regression Line
**Library**: Seaborn (`regplot`)
**Size**: 16x16 inches
**Features**:
- X-axis: Current health expenditure (% of GDP)
- Y-axis: Life expectancy at birth (years)
- Scatter points: Alpha transparency (0.5), size 40
- Regression line: Red color, line width 2
- Grid: Enabled for readability
- Title: "Relationship between Expenditure and Life Expectancy"

**Purpose**:
- Examine causal relationship between health spending and outcomes
- Identify outliers (countries with high spending but low life expectancy, or vice versa)
- Quantify the strength of relationship through regression line

**Insights Potential**:
- Positive correlation suggests investment in health improves outcomes
- Outliers may indicate inefficiencies or other factors at play
- Regional patterns may emerge when colored by region/income level

### Power BI Dashboard Visualizations

**Note**: The Power BI file (.pbix) contains interactive dashboards created from the exported CSV files. While the exact visualizations cannot be directly extracted from the binary file, based on the data structure and common Power BI practices, the dashboard likely includes:

#### Expected Dashboard Components:

**1. Economic Indicators Dashboard**
- GDP growth trends over time (line chart)
- GDP per capita by country/region (bar chart or map)
- Income level distribution (pie chart)
- Time series analysis (area chart)

**2. Labour Market Dashboard**
- Unemployment rates by region (comparative bar chart)
- Youth unemployment trends (line chart)
- Labour force size visualization (treemap or bar chart)
- Regional comparisons (clustered column chart)

**3. Trade & Globalization Dashboard**
- Export/Import trends (dual-axis line chart)
- Trade balance by country (waterfall chart)
- Regional trade patterns (map visualization)
- Time series comparison (area chart)

**4. Poverty & Inequality Dashboard**
- Poverty headcount by region (bar chart)
- Gini index distribution (histogram or box plot)
- Income inequality trends (line chart)
- Regional poverty comparison (clustered bar chart)

**5. Environmental Indicators Dashboard**
- Renewable energy consumption trends (line chart)
- Forest area coverage by region (map or bar chart)
- Environmental sustainability index (gauge or KPI)
- Regional comparisons (stacked bar chart)

**6. Health Indicators Dashboard**
- Life expectancy by country/region (map or bar chart)
- Infant mortality trends (line chart)
- Health expenditure analysis (scatter plot)
- Immunization coverage (gauge charts)
- Maternal health indicators (comparative visuals)
- Disease incidence trends (area chart)
- Cross-indicator analysis (matrix visual)

**7. Technology Indicators Dashboard**
- Internet penetration by region (bar chart or map)
- Mobile subscription trends (line chart)
- Digital divide analysis (comparative visuals)
- Technology adoption over time (area chart)

**8. Cross-Domain Analysis**
- Multi-KPI dashboard with key metrics
- Drill-through capabilities between domains
- Slicers for filtering by region, income level, year
- Interactive tooltips with detailed information

**Power BI Features Likely Utilized**:
- Slicers for filtering (Region, Income Level, Year)
- Drill-through pages for detailed analysis
- Tooltips with additional context
- Bookmarks for navigation
- Custom visuals for enhanced presentation
- Responsive layout for different screen sizes

## Data Flow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    World Bank API                            │
│  (RESTful JSON Endpoints)                                    │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Python Data Extraction Layer                    │
│  • Country Metadata Extraction                               │
│  • Indicator Catalog Extraction                              │
│  • Domain-Specific Data Extraction                           │
│  • Pagination & Rate Limiting                                │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Data Processing & Engineering                    │
│  • JSON Normalization                                        │
│  • Feature Extraction (Lambda Functions)                     │
│  • Data Filtering (Year > 2015)                              │
│  • Column Standardization                                    │
│  • Data Merging (Country Metadata)                           │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Analysis & Visualization                        │
│  • Correlation Analysis                                      │
│  • Regression Analysis                                       │
│  • Statistical Visualizations (Python)                       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Data Export Layer                               │
│  • CSV Export (7 Domain Files)                               │
│  • Indicator Catalog Export                                  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Power BI Dashboard                              │
│  • Data Import from CSV                                      │
│  • Interactive Visualizations                                │
│  • Cross-Domain Analysis                                     │
│  • Drill-Down Capabilities                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Insights and Findings Framework

### Data Coverage
- **Countries**: 296 countries/territories
- **Indicators**: 29,323 total indicators cataloged
- **Selected Indicators**: 25 indicators across 7 domains
- **Time Period**: 2016 onwards
- **Total Data Points**: ~62,244 records

### Analytical Capabilities Enabled

**1. Temporal Analysis**
- Year-over-year trends
- Multi-year comparisons
- Time series forecasting potential

**2. Comparative Analysis**
- Cross-country comparisons
- Regional benchmarking
- Income level analysis

**3. Correlation Analysis**
- Inter-indicator relationships
- Domain cross-connections
- Predictive modeling foundation

**4. Geographic Analysis**
- Regional patterns
- Country-level insights
- Spatial distribution (via coordinates)

### Business Value

**1. Policy Making**
- Evidence-based decision support
- Benchmarking against global standards
- Identifying improvement areas

**2. Research & Development**
- Academic research support
- Trend identification
- Hypothesis generation

**3. Strategic Planning**
- Resource allocation insights
- Priority setting
- Goal tracking

**4. Performance Monitoring**
- KPI tracking
- Progress measurement
- Gap analysis

---

## Technical Implementation Details

### API Integration Specifications

**Base URLs:**
- Countries: `https://api.worldbank.org/countries`
- Indicators: `https://api.worldbank.org/v2/indicator`
- Indicator Data: `https://api.worldbank.org/countries/all/indicators/{INDICATOR_CODE}`

**Parameters:**
- `format=json`: Response format
- `per_page={N}`: Records per page (50-1000)
- `page={N}`: Page number for pagination

**Response Structure:**
```json
[
  {
    "page": 1,
    "pages": N,
    "per_page": "500",
    "total": N
  },
  [
    {
      "id": "...",
      "value": "...",
      ...
    }
  ]
]
```

### Data Processing Pipeline

**Step 1: Extraction**
- API calls with error handling
- Pagination management
- Rate limiting (0.3s delay)

**Step 2: Transformation**
- JSON normalization
- Column extraction and renaming
- Data type conversion
- Temporal filtering

**Step 3: Enrichment**
- Country metadata merge
- Feature engineering
- Column cleanup

**Step 4: Analysis**
- Statistical computations
- Correlation analysis
- Regression analysis

**Step 5: Visualization**
- Python-based static charts
- Power BI interactive dashboards

**Step 6: Export**
- CSV file generation
- Power BI import preparation
