# REQUIREMENTS

Source of truth: the project notes only. The requirement text uses ASD-STE100 Simplified Technical English.

## Required Requirement Inputs

- Project purpose: A stock market application for all markets in the world. It uses machine learning analysis to help the user invest like Warren Buffett. It makes a trade model from the good trades of Warren Buffett in all of history. It examines the trades of the user and changes the strategy and the trade model from the results.
- Primary users / actors: The investor ("me" in the notes). The notes also use "we" and "our", but they do not identify these persons.
- Core workflows: (1) Examine the good trades of Warren Buffett in all of history. (2) Make a trade model from this strategy. (3) Apply the trade model to future market data to help the user invest. (4) Examine the trades of the user for success. (5) Change the strategy and the trade model from the results, continuously.
- Business objects / data entities: Stock market; market data; Warren Buffett historical trade; good trade; strategy; trade model; own trade (user trade); trade success result.
- External integrations: UNKNOWN
- Authentication / roles: UNKNOWN
- Regulatory or privacy constraints: UNKNOWN

## Functional Requirements

### 1. Global Market Coverage

- **FR-1.1** The system MUST supply stock market data for all stock markets in the world. The list of markets is in OQ-1.
- **FR-1.2** The system MUST use data from all supported stock markets in its machine learning analysis.

### 2. Warren Buffett Trade History

- **FR-2.1** The system MUST keep a record of the trades that Warren Buffett made in all of history. The scope of this record is in OQ-3.
- **FR-2.2** The system MUST identify each recorded Warren Buffett trade as a good trade or as not a good trade. The criteria are in OQ-3.
- **FR-2.3** The system MUST examine the Warren Buffett good trades with machine learning analysis.

### 3. Warren Buffett Strategy Model

- **FR-3.1** The system MUST make a trade model from the Warren Buffett good trades.
- **FR-3.2** The trade model MUST use the investment strategy that the Warren Buffett good trades show.
- **FR-3.3** The system MUST apply the trade model to new market data after the system makes the model.
- **FR-3.4** The system MUST show the results of the trade model to the user to help the user invest. The type of result is in OQ-4.

### 4. Own Trade Evaluation

- **FR-4.1** The system MUST keep a record of the trades of the user. The source of these trades is in OQ-5.
- **FR-4.2** The system MUST examine each recorded user trade and find if the trade is a success. The criteria are in OQ-6.
- **FR-4.3** The system MUST keep a record of the success result for each examined user trade.

### 5. Strategy and Model Adaptation

- **FR-5.1** The system MUST use the success results of the user trades to change the strategy.
- **FR-5.2** The system MUST use the success results of the user trades to change the trade model.
- **FR-5.3** The system MUST do FR-5.1 and FR-5.2 again when new success results become available. The frequency is in OQ-7.
- **FR-5.4** After the system changes the trade model, the system MUST use the changed trade model for all subsequent results.

## Non-Functional Requirements

None stated

## Open Questions

- **OQ-1** Which markets does "all markets globally" include? Does the scope include only stocks, or also other instruments (for example ETFs, bonds, commodities)? How must the system show prices in different currencies?
- **OQ-2** Which data sources supply market data and the Warren Buffett trade history? Must market data be real-time, delayed, or end-of-day? How much historical data must the system keep?
- **OQ-3** Which trades are "Warren Buffett trades" (personal, Berkshire Hathaway, Buffett Partnership, other)? Which period is "all of history"? What criteria make a trade a "good trade"?
- **OQ-4** What output must the system give to "help me invest" (for example buy/sell/hold recommendation, score, alert, portfolio allocation)? Must the output include a reason? What must the system show when the model cannot give a result for a stock?
- **OQ-5** Does the system execute trades through a broker, or does it only record trades? If it only records, how do trades enter the system (manual entry, file import, broker connection)?
- **OQ-6** What makes a user trade a "success" (metric, time period, benchmark)? When must the system examine a trade (at close, at fixed intervals)?
- **OQ-7** How often, and on which trigger, must the system change the strategy and the trade model? Must the change be automatic, or must the user approve it? Must the system keep previous versions and let the user go back to a previous version?
- **OQ-8** Are "strategy" and "trade model" two different objects, or one object? If different, what does each contain?
- **OQ-9** Must the system test the trade model against historical data before it shows results to the user? What minimum result must the test give?
- **OQ-10** Who are "we" in the notes? Is the system for one user or for many users? Do users share trades, strategies, or models? What authentication and roles are necessary?
- **OQ-11** Which regulatory constraints apply (for example investment-advice rules, market-data licenses, jurisdictions)? Which privacy rules apply to user trade data?
- **OQ-12** Which non-functional requirements apply (performance, availability, data latency, capacity, security)?
- **OQ-13** Which platforms must the application use (web, mobile, desktop, API)?
