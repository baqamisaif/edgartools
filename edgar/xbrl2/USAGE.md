# XBRL2 Standardization Package - Usage Guide

This package provides sector-aware XBRL concept mapping and standardized financial statement extraction.

## Installation

The `xbrl2` package is part of EdgarTools. No additional installation required.

```python
from edgar.xbrl2 import (
    SECTORS, SECTOR_PRIORITY,
    get_sector_by_sic, get_sector_info, get_all_sector_keys,
    get_package_dir, get_map_dir
)
```

## Quick Start

### Filing Integration (Recommended)

The easiest way to use xbrl2 is through the `Filing.standardized_financials()` method:

```python
from edgar import Company

# Get a filing
company = Company("AAPL")
filing = company.get_filings(form="10-K").latest()

# Extract standardized financials (all 3 statements at once)
data = filing.standardized_financials()

# Access income statement fields
print(f"Revenue: ${data['income_statement']['revenue']:,.0f}")
print(f"Net Income: ${data['income_statement']['netIncome']:,.0f}")
print(f"EPS: ${data['income_statement']['earningsPerShareBasic']:.2f}")

# Access balance sheet fields
print(f"Total Assets: ${data['balance_sheet']['totalAssets']:,.0f}")
print(f"Total Equity: ${data['balance_sheet']['totalEquity']:,.0f}")

# Access cash flow fields
print(f"Operating Cash Flow: ${data['cash_flow']['operatingCashFlow']:,.0f}")
print(f"Free Cash Flow: ${data['cash_flow']['freeCashFlow']:,.0f}")

# Check extraction metadata
print(f"Extraction Rate: {data['meta']['extraction_rate']}")
```

**With Industry Hints (for banks, insurance, utilities):**

```python
# For banks - uses interest income/expense rules
filing = Company("JPM").get_filings(form="10-K").latest()
data = filing.standardized_financials(industry="Bank")

# For insurance companies
filing = Company("MET").get_filings(form="10-K").latest()
data = filing.standardized_financials(industry="Insurance")
```

**Output Structure:**

```python
{
    "income_statement": {
        "revenue": 391035000000.0,
        "netIncome": 96995000000.0,
        "earningsPerShareBasic": 6.16,
        # ... 25 fields total
    },
    "balance_sheet": {
        "totalAssets": 352583000000.0,
        "totalLiabilities": 290437000000.0,
        "totalEquity": 62146000000.0,
        # ... 39 fields total
    },
    "cash_flow": {
        "operatingCashFlow": 118254000000.0,
        "freeCashFlow": 99584000000.0,
        # ... 33 fields total
    },
    "meta": {
        "form": "10-K",
        "period": "2024-09-28",
        "accession": "0000320193-24-000123",
        "extraction_rate": "64.9%"
    }
}
```

---

### 1. Sector Lookup

```python
from edgar.xbrl2 import get_sector_by_sic, get_sector_info

# Look up sector by SIC code
sector = get_sector_by_sic(6021)  # Commercial bank
print(sector)  # 'financials_banking'

# Get sector details
info = get_sector_info(sector)
print(info['name'])  # 'Financials - Banking'
print(info['sic_ranges'])  # [(6020, 6029), (6030, 6036), (6061, 6062)]
```

**Supported Sectors:**

| Sector Key | Name | SIC Ranges |
|------------|------|------------|
| `financials_banking` | Financials - Banking | 6020-6029, 6030-6036, 6061-6062 |
| `financials_securities` | Financials - Securities & Investment | 6200-6289, 6720-6799 |
| `financials_insurance` | Financials - Insurance | 6300-6399, 6411 |
| `financials_realestate` | Financials - Real Estate & REITs | 6500-6553, 6798-6799 |
| `energy_utilities` | Energy - Utilities | 4910-4941 |
| `energy_oilgas` | Energy - Oil & Gas | 1300-1399, 2911 |
| `industrials_general` | Industrials - General Manufacturing | 2000-3999 |
| `technology` | Technology - Software & Hardware | 3570-3579, 7370-7379 |
| `healthcare` | Healthcare - Pharma & Services | 2833-2836, 8000-8099 |
| `consumer` | Consumer - Retail & Goods | 5000-5999, 2000-2399 |

### 2. Extract Income Statement

Command-line extraction for any company:

```bash
# Extract from 10-K
python edgar/xbrl2/extractors/ic.py --symbol AAPL --form 10-K --identity "Your Name your@email.com"

# Extract from 10-Q
python edgar/xbrl2/extractors/ic.py --symbol MSFT --form 10-Q --identity "Your Name your@email.com"

# With industry hint for banks
python edgar/xbrl2/extractors/ic.py --symbol JPM --form 10-K --industry "Bank" --identity "Your Name your@email.com"
```

**Output Example (AAPL 10-K):**

```json
{
  "symbol": "AAPL",
  "financials": [
    {
      "revenue": 391035000000.0,
      "costOfGoodsSold": 214137000000.0,
      "grossIncome": 176898000000.0,
      "researchDevelopment": 29915000000.0,
      "sgaExpense": 24932000000.0,
      "operatingIncome": 122034000000.0,
      "netIncome": 96995000000.0,
      "earningsPerShareBasic": 6.16,
      "earningsPerShareDiluted": 6.13,
      "period": "2024-09-28",
      "year": 2024
    }
  ],
  "meta": {
    "form": "10-K",
    "periodEnd": "2024-09-28",
    "extraction": "statement_df",
    "extractionRate": "68.0%"
  }
}
```

### 3. Extract Balance Sheet

```bash
python edgar/xbrl2/extractors/bs.py --symbol AAPL --form 10-K --identity "Your Name your@email.com"
```

**Fields Extracted:**

| Field | Description |
|-------|-------------|
| `cash` | Cash and cash equivalents |
| `shortTermInvestments` | Marketable securities (current) |
| `accountsReceivable` | Trade receivables |
| `inventory` | Inventory |
| `totalCurrentAssets` | Total current assets |
| `propertyPlantEquipment` | PP&E, net |
| `totalAssets` | Total assets |
| `accountsPayable` | Trade payables |
| `shortTermDebt` | Short-term borrowings |
| `totalCurrentLiabilities` | Total current liabilities |
| `longTermDebt` | Long-term debt |
| `totalLiabilities` | Total liabilities |
| `retainedEarnings` | Retained earnings |
| `totalEquity` | Total stockholders' equity |
| `totalLiabilitiesAndEquity` | Total liabilities and equity |

### 4. Extract Cash Flow Statement

```bash
python edgar/xbrl2/extractors/cf.py --symbol AAPL --form 10-K --identity "Your Name your@email.com"
```

**Fields Extracted:**

| Field | Description |
|-------|-------------|
| `netIncome` | Net income (starting point) |
| `depreciationAndAmortization` | D&A add-back |
| `operatingCashFlow` | Net cash from operations |
| `capitalExpenditures` | CapEx |
| `investingCashFlow` | Net cash from investing |
| `stockRepurchases` | Share buybacks |
| `dividendsPaid` | Dividends paid |
| `financingCashFlow` | Net cash from financing |
| `netChangeInCash` | Net change in cash |
| `freeCashFlow` | Operating cash flow - CapEx |

## Programmatic Usage

### Using the Evaluator Directly

```python
import json
from pathlib import Path
from edgar.xbrl2.extractors.ic import Evaluator

# Load schema
schema_path = Path('edgar/xbrl2/schemas/income-statement.json')
with open(schema_path) as f:
    mapping = json.load(f)

# Your XBRL facts dict (concept -> value)
facts = {
    'us-gaap:RevenueFromContractWithCustomerExcludingAssessedTax': 391035000000,
    'us-gaap:CostOfGoodsAndServicesSold': 214137000000,
    'us-gaap:GrossProfit': 176898000000,
    'us-gaap:ResearchAndDevelopmentExpense': 29915000000,
    'us-gaap:SellingGeneralAndAdministrativeExpense': 24932000000,
    'us-gaap:OperatingIncomeLoss': 122034000000,
    'us-gaap:NetIncomeLoss': 96995000000,
    'us-gaap:EarningsPerShareBasic': 6.16,
    'us-gaap:EarningsPerShareDiluted': 6.13,
}

# Extract standardized values
evaluator = Evaluator(mapping=mapping, facts=facts, industry=None, normalize_abs=True)
standardized = evaluator.standardize()

print(f"Revenue: ${standardized['revenue']:,.0f}")
print(f"Net Income: ${standardized['netIncome']:,.0f}")
print(f"EPS Basic: ${standardized['earningsPerShareBasic']:.2f}")
```

### Getting Facts from Edgar Company

```python
from edgar import Company

company = Company("AAPL")
filing = company.get_filings(form="10-K").latest()
xbrl = filing.xbrl()

# Get income statement as DataFrame
stmt_df = xbrl.current_period.income_statement(as_statement=False, include_dimensions=False)

# Convert to facts dict
facts = {}
for _, row in stmt_df.iterrows():
    concept = row.get('concept', '')
    value = row.get('value')
    if concept and value is not None:
        facts[concept] = float(value)
```

## Schema Structure

Schemas define standardized fields with fallback rules:

```json
{
  "fields": {
    "revenue": {
      "standardLabel": "revenue",
      "rules": [
        {
          "name": "Banks / lenders",
          "priority": 150,
          "industryHints": ["Bank", "Banks", "Diversified Banks"],
          "selectAny": [
            "us-gaap:RevenuesNetOfInterestExpense",
            "us-gaap:Revenues"
          ],
          "computeAny": [
            {
              "op": "add",
              "terms": [
                {"conceptAny": ["us-gaap:InterestIncomeExpenseNet"]},
                {"conceptAny": ["us-gaap:NoninterestIncome"]}
              ]
            }
          ]
        },
        {
          "name": "General (GAAP primary)",
          "priority": 100,
          "selectAny": [
            "us-gaap:RevenueFromContractWithCustomerExcludingAssessedTax",
            "us-gaap:Revenues",
            "us-gaap:SalesRevenueNet"
          ]
        }
      ]
    }
  }
}
```

**Rule Evaluation:**
1. Rules are sorted by `priority` (descending)
2. `industryHints` filters rules by company industry
3. `selectAny` tries each concept, returns first non-null value
4. `computeAny` tries each expression, returns first fully-resolved value
5. Expressions support: `add`, `sub`, `mul`, `div`, `id` operations

## ML Data Files

The `ml_data/` directory contains machine-learned concept mappings:

| File Pattern | Description |
|--------------|-------------|
| `virtual_trees_*.json` | Hierarchical concept structures |
| `canonical_structures_*.json` | Standard statement templates |
| `learned_mappings_*.json` | Concept-to-field mappings |
| `learning_statistics_*.json` | Training statistics |
| `statement_mappings_v1_*.json` | Version 1 statement mappings |

Sector suffixes: `_global`, `_banking`, `_insurance`, `_utilities`

## Sector Overlays

Sector-specific overrides in `overlays/`:

- `banking.json` - Interest income/expense handling
- `insurance.json` - Premium/claims model
- `utilities.json` - Rate-regulated revenue

## Testing

Run the test extraction scripts:

```bash
# Test with sample companies
cd edgar/xbrl2/tests
python extract_financials.py --symbol AAPL --form 10-K

# Compare with actual financials
python compare_financials.py --symbol AAPL
```

## Extraction Rates

Typical extraction rates by company type:

| Company Type | 10-K Rate | 10-Q Rate |
|--------------|-----------|-----------|
| Technology (AAPL, MSFT) | 65-70% | 65-70% |
| Banks (JPM, BAC) | 55-65% | 55-65% |
| Industrials (GE, CAT) | 60-70% | 60-70% |
| Insurance (BRK, MET) | 50-60% | 50-60% |

Lower rates indicate:
- Company uses non-standard XBRL concepts
- Fields not applicable to company type (e.g., COGS for banks)
- Missing detailed breakdowns in filing

## Troubleshooting

### Empty or Null Fields

1. Check if field is applicable to company type
2. Verify company uses standard XBRL taxonomy
3. Try adding `--industry` hint for banks/insurance

### Low Extraction Rate

1. Check filing has XBRL data: `filing.xbrl()` should not be None
2. Verify period matches: use `--period-end YYYY-MM-DD`
3. Check for company-specific extension concepts

### Import Errors

Ensure EdgarTools is installed:
```bash
pip install edgartools
```

## API Reference

### `edgar.xbrl2` Module

| Function | Description |
|----------|-------------|
| `get_sector_by_sic(sic: int)` | Returns sector key for SIC code |
| `get_sector_info(sector: str)` | Returns sector configuration dict |
| `get_all_sector_keys()` | Returns list of all sector keys |
| `get_package_dir()` | Returns package directory Path |
| `get_map_dir()` | Returns map directory Path |

### Constants

| Constant | Description |
|----------|-------------|
| `SECTORS` | Dict of all sector configurations |
| `SECTOR_PRIORITY` | List of sectors in priority order |
| `DEFAULT_GLOBAL_THRESHOLD` | Default occurrence threshold (0.10) |
| `DEFAULT_SECTOR_THRESHOLD` | Default sector threshold (0.10) |
