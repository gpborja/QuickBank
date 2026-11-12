# Quick Bank

A single-file banking demo that runs entirely in the browser. No build step, no dependencies, no server.

## Run

Open `index.html` in any modern browser, or serve the folder:

```sh
python3 -m http.server 8080
```

## Features

- Checking and Savings accounts with a total balance; click an account card to filter history
- Deposit, withdraw, and transfer (between your accounts or to an external payee)
- Overdraft protection and strict amount validation (positive, up to 2 decimals)
- Searchable, filterable transaction history with running balance
- Amounts stored as integer cents to avoid floating-point rounding errors
- Data persists in `localStorage`; **Reset demo** restores the sample data

This is a front-end demo only: no authentication, no backend, no real money.
