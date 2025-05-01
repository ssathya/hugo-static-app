+++
date = '2025-05-01T12:40:09-04:00'
draft = false
title = 'Simplified Stock Portfolio Tracker and Analyzer'
tags = ['Programming']
categories = ['docs', 'Stock Market']
+++


## Preamble
*This high-level requirement was generated using my prompts, with the AI providing its interpretations of my bullet points. My implementation of the Portfolio Tracker may diverge from the original specifications. However, I must establish a target, and I hope that the final product will incorporate most of the defined objectives. Once I execute my project, I will enumerate the changes and articulate how they differ from the initial goals.*

## Detailed Specification

### 1. Project Overview

The goal of this project is to build a web application using C# and Blazor that allows users to easily track the performance of their stock portfolio. Users will be able to input their stock holdings and purchase history, and the application will then fetch real-time (or near real-time) stock data to calculate portfolio value, returns, and present insightful visualizations. The application aims to provide a clear and straightforward view of investment performance without overwhelming the user with excessive complexity.

### 2. Technical Details (C# with Blazor)

You've chosen a fantastic tech stack! Here's how the application could be structured:

* **Blazor Frontend (Client-Side or Server-Side):**
    * **Components:** You'll likely create several Blazor components for different parts of the application:
        * `PortfolioInput.razor`: For users to add or edit their stock holdings and purchase details.
        * `PortfolioDisplay.razor`: To show the current portfolio holdings with real-time data.
        * `PerformanceCharts.razor`: To render visualizations of portfolio performance over time.
        * `TransactionHistory.razor`: To display a log of the user's buy and sell transactions.
        * `StockDetails.razor`: Potentially to show more detailed information about individual stocks.
        * Layout components for the overall structure of the application.
    * **State Management:** Consider how you'll manage the application's state, especially the user's portfolio data and fetched stock quotes. Options include:
        * **Simple State Management:** For a medium-complexity project, you might start with passing state down through component parameters and using `EventCallback` for child-to-parent communication.
        * **Flux/Redux-like patterns:** Libraries like Fluxor or Blazor State offer more structured approaches for larger applications but might be overkill initially.
        * **Scoped Services:** Using scoped services to hold and manage user-specific data can be a good middle ground.
    * **UI Framework:** Blazor leverages standard HTML and CSS. You can enhance the styling and responsiveness with CSS frameworks like Bootstrap or Tailwind CSS, or Blazor UI component libraries like MudBlazor or Radzen Blazor Components.
* **Backend (ASP.NET Core Web API):**
    * **Controllers:** You'll need API controllers to handle requests from the Blazor frontend:
        * `PortfolioController`: Endpoints for adding, retrieving, updating, and deleting portfolio holdings.
        * `StockDataController`: Endpoint(s) to fetch real-time stock quotes and potentially historical data.
    * **Services:** Create services to encapsulate business logic:
        * `PortfolioService`: To manage portfolio data (calculations, persistence).
        * `StockDataService`: To interact with the chosen stock data API.
    * **Data Access:** Use Entity Framework Core (EF Core) to interact with your database. Define your data models (e.g., `PortfolioItem`, `Transaction`).
    * **Authentication and Authorization:** Implement user authentication to secure user data. ASP.NET Core Identity is a robust choice.
* **Database:**
    * Choose a relational database like PostgreSQL, MySQL, or SQL Server to store user data, portfolio holdings, and transaction history.

### 3. Data Source

For fetching real-time or near real-time stock data, you'll need to integrate with a financial data API. Here are some popular options:

* **Free/Limited Free Options:**
    * **Yahoo Finance API (Unofficial Libraries):** While Yahoo Finance doesn't have an official public API, there are several community-maintained libraries in C# that can scrape or access its data (be mindful of terms of service and potential instability).
    * **Alpha Vantage:** Offers a free tier with limitations on API calls and data frequency. Provides real-time and historical stock data.
    * **Financial Modeling Prep:** Offers a free tier with limited API calls and data.
* **Paid Options (More Reliable and Feature-Rich):**
    * **IEX Cloud:** A popular option with various data plans and a well-documented API.
    * **Polygon.io:** Offers real-time and historical market data with different pricing tiers.
    * **Refinitiv Eikon API:** A professional-grade API with comprehensive financial data (typically higher cost).
    * **Bloomberg API:** Another professional-grade option (also typically higher cost).

**Considerations for Data Source:**

* **Real-time vs. Delayed Data:** Decide if you need truly real-time data or if a slight delay (e.g., 15-minute delayed data offered by some free APIs) is acceptable for your "simplified" tracker.
* **API Limits:** Be aware of rate limits on free APIs and design your application to handle them gracefully (e.g., implement caching, backoff strategies).
* **Data Coverage:** Ensure the API covers the stock exchanges and instruments your target users are likely to trade.
* **Historical Data:** If you plan to implement charting of past performance, you'll need an API that provides historical data.

### 4. Portfolio Details

#### a. User-Provided Details (Input)

When a user adds a stock to their portfolio, they will likely need to provide the following information:

* **Instrument Details:**
    * **Ticker Symbol (Required):** The unique identifier of the stock (e.g., AAPL, MSFT, GOOGL).
    * **Company Name (Optional, but helpful for display):** The full name of the company. You might consider fetching this automatically based on the ticker symbol using the data API.
    * **Exchange (Optional, but can be important for disambiguation):** The stock exchange where the stock is traded (e.g., NASDAQ, NYSE). Some APIs might require this for accurate data retrieval.
* **Procurement Details (Transaction History):**
    * **Transaction Type (Buy/Sell):** To track purchases and sales.
    * **Purchase Date (for Buy transactions):** The date the stock was acquired.
    * **Purchase Price (per share, for Buy transactions):** The price at which the stock was bought.
    * **Quantity (Number of shares):** The number of shares bought or sold.
    * **Transaction Costs/Fees (Optional):** Any brokerage fees associated with the transaction.

You'll need a user interface that allows users to input this information easily, potentially with validation to ensure data integrity. You might also consider allowing users to edit or delete their transactions.

#### b. Details to be Presented to the User (Output)

Based on the input data and fetched stock information, here are some details you can present to the user:

* **Current Holdings:**
    * **Ticker Symbol:** The stock ticker.
    * **Company Name:** The name of the company.
    * **Current Price:** The latest stock price fetched from the data API.
    * **Quantity Held:** The total number of shares currently owned.
    * **Current Value:** The current price multiplied by the quantity held.
    * **Average Cost Basis (per share):** The average price paid for all shares of that stock (total cost of purchases divided by total shares bought).
    * **Total Cost Basis:** The total amount invested in that stock.
    * **Unrealized Gain/Loss (Dollar Amount):** Current Value minus Total Cost Basis.
    * **Unrealized Gain/Loss (Percentage):** ((Current Value - Total Cost Basis) / Total Cost Basis) \* 100.

* **Portfolio Summary:**
    * **Total Portfolio Value:** The sum of the current value of all holdings.
    * **Total Investment:** The sum of the cost basis of all holdings.
    * **Overall Gain/Loss (Dollar Amount):** Total Portfolio Value minus Total Investment.
    * **Overall Gain/Loss (Percentage):** ((Total Portfolio Value - Total Investment) / Total Investment) \* 100.

* **Charts:**
    * **Portfolio Value Over Time:** A line chart showing how the total portfolio value has changed over a selected period (e.g., 1 month, 6 months, 1 year, YTD, All Time). This will require fetching historical stock data.
    * **Asset Allocation (Optional):** A pie chart showing the percentage of the portfolio allocated to different stocks.

* **Investment Returns:**
    * **Daily/Weekly/Monthly Returns (Percentage or Dollar Amount):** Showing the change in portfolio value over specific periods.
    * **Time-Weighted Return (Optional, more complex):** A more accurate measure of investment performance that removes the impact of cash flows (deposits and withdrawals). This is generally more advanced.

* **Analyst Estimates (Optional, adds complexity):**
    * **Target Price:** Analyst consensus on the expected future price of the stock.
    * **Buy/Hold/Sell Ratings:** Summary of analyst recommendations.
    * **Note:** This data is often available through paid APIs and can be less reliable or consistent across different sources. Consider if this is crucial for your "simplified" version.

* **Transaction History:**
    * A table displaying all buy and sell transactions with details like date, ticker, quantity, price, and total cost/proceeds.

**Further Considerations:**

* **User Interface (UI) and User Experience (UX):** Focus on creating a clean, intuitive, and responsive interface using Blazor components and styling.
* **Error Handling:** Implement proper error handling for API calls, data parsing, and user input.
* **Data Persistence:** Use EF Core to save and retrieve user portfolio data from your chosen database.
* **Asynchronous Operations:** Use `async` and `await` effectively in Blazor to handle API calls and prevent blocking the UI thread.
* **Code Organization:** Structure your C# code logically with services, controllers, and data models to maintain readability and scalability.
