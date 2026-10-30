# REQUIREMENTS

Owns WHAT the system must do: functional and non-functional requirements, including externally observable validation, authorization, and failure behavior. Design: `DESIGN.md`. Architecture: `ARCHITECTURE.md`. Security controls: `SECURITY.md`.

Source of truth: the project notes and the owner interview. The requirement text uses ASD-STE100 Simplified Technical English.

## Required Requirement Inputs

- Project purpose: A stock market application for the United States (US) stock markets. It uses machine learning analysis to help the user invest like Warren Buffett. It makes a trade model from the good trades of Warren Buffett in all of history. It examines the trades of the user and changes the strategy and the trade model from the results.
- Primary users / actors: One investor (the user). "We" and "our" in the notes refer to this user. There are no other users.
- Core workflows: (1) Examine the good trades of Warren Buffett in all of history. (2) Make a trade model from this strategy. (3) Apply the trade model to new market data to help the user invest. (4) Examine the trades of the user for success. (5) Change the strategy and the trade model from the results, continuously.
- Business objects / data entities: US stock; end-of-day market data; Warren Buffett historical trade; good trade; strategy; trade model; model version; backtest result; user trade; trade success result.
- External integrations: SEC EDGAR (Berkshire Hathaway Form 13F filings); one end-of-day US stock price provider (free or low cost; selected in `ARCHITECTURE.md`). The system does not connect to a broker.
- Authentication / roles: One user account with password login. One role (owner). The owner creates the account on the host. There is no web sign-up, no admin, and no social function.
- Regulatory or privacy constraints: Personal use only. The output is not investment advice. The system must obey the terms of use of each data provider. User trade data is private.

## Functional Requirements

### 1. US Market Coverage

- **FR-1.1** The system MUST supply market data for common stocks on US stock exchanges (NYSE, NASDAQ, NYSE American) only. The system MUST NOT supply data for ETFs, bonds, commodities, or non-US markets.
- **FR-1.2** The system MUST show all prices in US dollars (USD).
- **FR-1.3** The system MUST get end-of-day market data one time each trading day, after the market closes.
- **FR-1.4** The system MUST keep the daily price history (with dividends and splits) for each stock from the date of the first recorded Warren Buffett trade to the current date.
- **FR-1.5** The system MUST use data from all supported stocks in its machine learning analysis.

### 2. Warren Buffett Trade History

- **FR-2.1** The system MUST keep a record of the Berkshire Hathaway US stock trades that are in all available Form 13F filings on SEC EDGAR.
- **FR-2.2** The system MUST derive each trade (buy, add, reduce, sell) from the change in holdings between two sequential 13F filings.
- **FR-2.3** The system MUST get each new 13F filing when it becomes available on SEC EDGAR (each quarter).
- **FR-2.4** The system MUST identify each recorded Warren Buffett position as a good trade if its total return (price change plus dividends) during the hold period is more than the total return of the S&P 500 index during the same period. For a position that Berkshire Hathaway still holds, the hold period ends at the current date.
- **FR-2.5** The system MUST examine the Warren Buffett good trades with machine learning analysis.

### 3. Strategy and Trade Model

- **FR-3.1** The system MUST keep the strategy and the trade model as two different objects.
- **FR-3.2** The strategy MUST contain the investment rules and the weight of each stock feature (for example price-to-earnings ratio, return on equity, debt level, earnings stability), in a form that the user can read.
- **FR-3.3** The trade model MUST be the machine learning model that the system trains from the Warren Buffett good trades and the strategy.
- **FR-3.4** Before the system shows results from a trade model, the system MUST test the model against historical data that the system did not use to train the model (backtest).
- **FR-3.4.1** The backtest MUST use a rolling walk-forward test: train on all years before a test year, test on that year, then move forward one year. The first training period MUST be 10 years or more.
- **FR-3.5** The system MUST show results from a trade model only if the backtest total return for all test years together is more than the S&P 500 total return for the same test years.
- **FR-3.6** The system MUST apply the trade model to new market data each trading day.
- **FR-3.7** For each stock, the system MUST show the user: a score from 0 to 100, a recommendation (buy, hold, or sell), and the reason (the strategy features with the largest effect on the score).
- **FR-3.7.1** A score from 70 to 100 MUST give buy. A score from 31 to 69 MUST give hold. A score from 0 to 30 MUST give sell. The reason MUST show the 3 features with the largest effect.
- **FR-3.8** When the model cannot give a result for a stock (for example, because data is missing), the system MUST show "No result" and the cause.
- **FR-3.9** The system MUST NOT execute trades.
- **FR-3.10** The system MUST show price history and backtest results as line charts. Each chart MUST show the S&P 500 line for comparison.
- **FR-3.11** The system MUST state on the results page (OQ-14) that it uses the total return of SPY as the S&P 500 baseline, and that the SPY fee makes this baseline slightly easier to beat.

### 4. User Trade Evaluation

- **FR-4.1** The system MUST keep a record of the trades of the user. Each trade MUST contain: stock symbol, buy or sell, date, quantity, and price.
- **FR-4.2** The user MUST be able to enter a trade manually and import trades from a CSV file.
- **FR-4.2.1** The CSV file MUST have one header row and these columns: symbol, side (buy or sell), date (YYYY-MM-DD), quantity, price (USD). If a row is not valid, the system MUST reject the full file and show the row number and the cause.
- **FR-4.3** The system MUST identify a user trade as a success if its total return is more than the total return of the S&P 500 index during the same hold period.
- **FR-4.4** The system MUST examine a user trade when the user closes the position, and each quarter for each open position.
- **FR-4.5** The system MUST keep a record of the success result for each examined user trade.

### 5. Strategy and Model Adaptation

- **FR-5.1** The system MUST use the success results of the user trades and new 13F data to propose a change to the strategy and the trade model.
- **FR-5.2** The system MUST make a proposal each quarter after it gets a new 13F filing, and when the user requests one.
- **FR-5.3** Each proposal MUST include the backtest result of the changed trade model, compared to the current trade model.
- **FR-5.4** The system MUST NOT apply a proposal until the user approves it.
- **FR-5.5** The system MUST keep all previous versions of the strategy and the trade model.
- **FR-5.6** The user MUST be able to go back to a previous version of the strategy and the trade model.
- **FR-5.7** After the system changes the trade model, the system MUST use the changed trade model for all subsequent results.

### 6. Users and Access

- **FR-6.1** The system MUST support one user account.
- **FR-6.2** The user MUST log in with a password before the user can use the system.
- **FR-6.3** The system MUST NOT let a person create an account from the web interface. The owner MUST create the one account with a command on the host.
- **FR-6.4** The user MUST be able to change the password. The system MUST ask for the current password before it changes the password.
- **FR-6.5** The user MUST be able to log out. Logout MUST end the session immediately.

### 7. Legal

- **FR-7.1** The system MUST show a statement that its output is not investment advice.
- **FR-7.2** The system MUST obey the terms of use and request limits of each data provider.

## Non-Functional Requirements

- **NFR-1 Platform:** The system MUST be a web application that operates in a current desktop web browser.
- **NFR-2 Data latency:** The system MUST complete the daily data update and model results not more than 2 hours after the end-of-day data becomes available.
- **NFR-3 Performance:** Each page MUST load in not more than 3 seconds.
- **NFR-4 Availability:** No high-availability requirement. The system MUST continue the daily update after a restart without loss of data.
- **NFR-5 Security:** The system MUST use HTTPS for all connections. The system MUST use TLS on each network connection, including connections between internal components. The system MUST NOT accept plain HTTP. The system MUST encrypt user trade data at rest. The system MUST keep passwords as salted hashes with a password hashing algorithm.
- **NFR-6 Privacy:** The system MUST NOT send user trade data to external services.
- **NFR-7 Display:** The system MUST operate in a browser window 1024 pixels wide or more. The system MUST use a light theme only.
- **NFR-8 Fonts:** The system MUST use the built-in fonts of the operating system. The system MUST NOT get fonts from external services.
- **NFR-9 Name:** The name of the system is "Oracle Buffet". The name is for personal use only. The system MUST NOT show photos of Warren Buffett or Berkshire Hathaway logos. The system MUST NOT state or suggest that Warren Buffett or Berkshire Hathaway approves the system.

## Open Questions

- **OQ-14** FR-3.11 says "the results page", but `DESIGN.md` defines no view with that name. Alternatives: (a) the Trade Results view only; (b) each view that shows a comparison with the S&P 500. TO BE DECIDED.
