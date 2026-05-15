
# My Dashboard - Project Brief

## Summary
A single-page analytics dashboard showing monthly business metrics. 
Primary stakeholder is VP of operations who needs consolidated, at-a-glance view 
Think Shopify admin or a simple Google Analytics view.

## Data
- Generate a fake dataset as a JSON file (src/data/metrics.json).
- 12 months of data (Jan-Dec 2025) that can be filtered down to last 30 days, 60 days, 6 months, and last year
- Data is by Region (North, West, South, East) and should include:
- Shipment volume
    - Total shipments (daily, weekly and monthly)
    - Shipments in transit vs. delivered vs. pending
    - Volume trends over time (rolling 30, 60, 90 days)
- On-time delivery
    - On-time delivery rate (%) by time period
    - Late shipment count and percentage
    - Trend line comparing current vs. prior period
- Regional Performance
    - Shipment volume by region
    - On-time rate by region
    - Region-level exception counts
- Open Exceptions
    - Total count of open exceptions
    - Exception type breakdown (damaged, delayed, lost, customs hold)

## Layout (Vuetify)
- v-app-bar at the top with the dashboard title "FastForward Logistics" and filters
- Filters should be drop down menus
    - Region filter: Default to All. Region filter should include All, North, South, East, West
    - Time Period Filter: Default to Last Year. Filter should include Last 30 Days, Last 60 Days, Last 6 Months, and Last Year
- When a time period is selected, all cards and charts filter to that range.
- Use a top-down hierarchy, prioritizing the most critical KPIs at the top and drilling into details as the user scrolls or interacts
- Below the app bar: a row of summary cards (v-card) showing the key metrics
- Use v-container, v-row, v-col for responsive grid layout

## Interactions
- Time period filter - dropdown menu that filters by time period. User can select one at a time.
- Region filter - dropdown menu that filters by region. User can select one at a time.
- Hovering over data points reveals exact values, date, and percentage changes
- Cards should show a small up/down arrow or color indicating change from previous month

## Style
- Clean, minimal, lots of whitespace
- Charts should use a cohesive color palette - not rainbow
- Mobile responsive - cards stack on small screens

## Tech
- Vue 3 + TypeScript + Vuetify 3
- Chart.js via vue-chartjs for all charts
- Fake data from a local JSON file (no API calls)
- Single page - no routing needed for this app
