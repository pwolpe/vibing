# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-page HTML application for **Total Return Analysis** of stocks. It calculates and visualizes the difference between simple price appreciation and dividend-reinvested returns using the Tiingo financial data API.

## Architecture

### Single-File Application Structure
The entire application is contained in `tra-app.html`, organized as follows:

1. **External Dependencies** (CDN-loaded):
   - Chart.js v4.4.1 for data visualization
   - chartjs-adapter-date-fns v3.0.0 for time-series support
   - Google Fonts (JetBrains Mono, IBM Plex Sans)

2. **CSS Architecture** (lines 12-460):
   - CSS custom properties for theming (dark mode color scheme)
   - Component-based styles: header, input panel, results table, chart section
   - Responsive design with mobile breakpoint at 768px

3. **HTML Structure** (lines 462-566):
   - Input panel: ticker, date range, share count
   - Results panel: summary header, comparison table, performance chart, dividend events
   - Loading and error states

4. **JavaScript Application Logic** (lines 568-1053):
   - **Configuration**: Tiingo API key (line 572)
   - **Data Fetching**: Functions to retrieve price and dividend data from Tiingo API
   - **Return Calculations**: Two strategies compared
     - Simple Price Appreciation: Uses raw closing prices
     - Dividends Reinvested: Compounds shares purchased from dividend reinvestment
   - **Chart Rendering**: Dual-axis chart showing portfolio value and total return percentage
   - **Annualization Formula**: `(1 + total return)^(1/years) - 1`

### Key Calculation Logic

**Dividend Reinvestment** (lines 777-808):
- Iterates through daily price data
- When dividend detected (`divCash > 0`), calculates dividend payout based on current shares
- Reinvests dividend at ex-date closing price to purchase additional shares
- Accumulates shares throughout the period for compounding effect

**Return Metrics**:
- Total Return %: `((final_value - initial_investment) / initial_investment) * 100`
- Annualized Return: Geometric mean accounting for time period
- Gain/Loss: Dollar amount difference from initial investment

## Development Commands

Since this is a standalone HTML file with no build process:

```bash
# Run a local development server (Python 3)
python3 -m http.server 8000

# Run a local development server (Python 2)
python -m SimpleHTTPServer 8000

# Run using Node.js http-server (if installed)
npx http-server -p 8000
```

Then open `http://localhost:8000/tra-app.html` in a browser.

## API Configuration

**Tiingo API Key** is hardcoded at line 572:
```javascript
const TIINGO_API_KEY = '6523b8c90ee93b29737a1fd0c953e2b603b49d0b';
```

**API Endpoints Used**:
- Daily prices: `https://api.tiingo.com/tiingo/daily/{ticker}/prices?startDate={date}&endDate={date}&token={key}`
- Returns both price data and dividend events (`divCash` field)

## Important Implementation Details

1. **Price Data Handling**:
   - Uses `close` (raw price) for simple price appreciation calculations
   - Uses `adjClose` for adjusted prices (though primarily relies on raw prices + dividend tracking)
   - Dividend detection: `day.divCash && day.divCash > 0`

2. **Chart Dual Y-Axes**:
   - Left axis (y): Portfolio value in dollars
   - Right axis (y1): Total return percentage
   - Three datasets: Price Only, Dividend Adjusted, Total Return %

3. **Date Handling**:
   - Input format: `YYYY-MM-DD` (HTML5 date input)
   - Display format: `MM/DD/YYYY`
   - API returns ISO 8601 timestamps, split on 'T' to extract date

4. **Error Handling**:
   - 404 responses show ticker not found
   - Empty data arrays trigger "no data available" message
   - Date validation ensures start < end

## Making Changes

When modifying this application:

- **Style changes**: Edit the `:root` CSS variables (lines 13-26) for theming
- **Calculation changes**: Focus on `calculateReturns()` function (lines 750-848)
- **Chart configuration**: Modify Chart.js options in `updateChart()` (lines 884-1034)
- **API changes**: Update fetch functions (lines 651-680) and API key constant (line 572)
- **UI components**: All DOM elements are vanilla JS, no framework

## Testing

No automated tests exist. Manual testing checklist:
- Valid ticker with dividends (e.g., AAPL, MSFT)
- Valid ticker without dividends
- Invalid ticker (should show 404 error)
- Date range validation (start >= end should error)
- Mobile responsive layout at < 768px width
