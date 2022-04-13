---
layout: post
title: "Open Problems in Crypto Infrastructure: Automated Accounting, Triple-Entry Bookkeeping, Streaming Financials"
tags: 
    - accounting
    - blockchains
usemathjax: true
---
This is blog post is a response to Balaji Srinivasan's Open Problems in Crypto Infrastructure [YouTube video](https://youtu.be/l2VQkbucXns?t=1077){:target="_blank"}. Particularly, we show a toy solution for the Automated Accounting, Triple-Entry Bookkeeping, Streaming Financials problem.

The code corresponding to this blog post is on Github [here](https://github.com/saiakilesh/automated-crypto-accounting){:target="_blank"}. To run the code, follow the instructions in the `README.md` file. The code uses Solidity, TypeScript, and the Hardhat Ethereum development environment. Since this is a toy solution, we will not optimize for gas usage, and instead optimize for code clarity. Also, to keep the blog post to a reasonable length, we will include GitHub links to relevant code snippets and pictures instead of directly including them here.

A quick summary of the blog post is as follows:
- We start with going over the three financial statements of accounting
- We then show how to implement a Solidity smart contract to that can conduct business tranasactions and do accounting
- Finally, we deploy this smart contract to a local Hardhat network and use it to do accounting for a hypothetical company over the course of a hypothetical fiscal year

## Basics of Accounting
We closely follow Thomas Ittelson's presentation of accounting in his book *Financial Accounting*. In accounting, there are three financial statements: 
- **Balance Sheet**: Contains information about what a company has (Assets), what it owes (Liabilities), and what it's worth (Equity) at a particular point in time. It always maintains the invariant Assets = Liabilities + Equity. The types of entries of a basic balance sheet can be seen in [this picture](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.35.31%20AM.png){:target="_blank"}.

- **Income Statement**: Contains information about the making and selling activities over a period of time. It documents the details of the equality Sales - Costs & Expenses = Income. The types of entries of a basic income statement can be seen in [this picture](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.35.42%20AM.png){:target="_blank"}.

- **Cash Flow Statement**: Tracks the movement of cash through a business over a period of time. The types of entries of a basic cash flow statement can be seen in [this picture](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.24.26%20AM.png){:target="_blank"}.

During the course of business, companies take various types of actions, which in turn change the entries in these statements. For instance, raising equity affects the balance sheet by increasing cash and increasing shareholder's equity. It also affects the cash flow statement by increasing the sale of capital stock entry. We will describe more actions and how they affect the various statements in the section below. 

## An Aside: Using an ERC20 Token Contract to Mock USD Coin (USDC)

USDC is our currency of choice for doing business and accounting. Since we will deploy to a local Hardhat network, we don't actually have access to USDC. So instead, we will use a generic ERC20 token contract from [OpenZeppelin](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20) to mock USDC. We call this contract `USDC.sol`.

For readers unfamiliar with ERC20 smart contracts, they maintain a mapping from addresses to balances, which keeps track of how many tokens a particular account owns. Tokens can be transferred between two accounts in the following two ways:
- The owner of the token calls the `transfer` function, which takes in 1) the amount to transfer and 2) the recipient to transfer to.
- The owner of the token calls the `approve` function which takes in a spender and an amount. The spender then has the ability to spend the owner's tokens up to the specified amount by calling the `transferFrom` function. The `transferFrom` function takes in 1) a sender, which is the account from which the tokens are being transferred, 2) an amount to transfer, and 3) a recipient to transfer to.

We will assume:
- When we pay USDC, the `transfer` method is used.
- When we are paid USDC, the `approve`, `transferFrom` method is used.

## Implementing an Accounting Smart Contract in Solidity

We now describe the main Solidity smart contract that companies can use to conduct businsess and do accounting. The smart contract can be found in the `Accounting.sol` file in the [Github repository](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol){:target="_blank"}. We will have three storage variable mappings corresponding to the three financial statements: `balanceSheet`, `incomeStatement`, and `cashFlowStatement`. Each mapping maps a `string` (such as `'Accounts Receivable'`) to a signed integer (such as `100`). We need signed integers because some values, like the net income can be negative. We also have a storage variable that we call `admins` which corresponds to the set of employees who are allowed to conduct business (make payments, receive payments, etc.) on behalf of the company. This variable is initialized in the constructor of the contract. See the code for the storage variables and the constructor [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L8-L25){:target="_blank"}.

Each type of action taken during the course of business can be represented as a function that usually changes at least one of the storage mappings. There are many possibilities for types of actions in the business world, but for this blog post we limit ourselves to the following 11 actions.

#### Action 1: Raising Equity

The `raiseEquity` function changes the three financial statements by:

- Increasing cash (balance sheet)
- Increasing capital stock (balance sheet)
- Increasing sale of capital stock (cash flow statement)

The `raiseEquity` function also receives the equity funds by calling the `transferFrom` function on `USDC.sol` with the relevant parameters. See the code for the `raiseEquity` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L33-L47){:target="_blank"}.

#### Action 2: Paying Salaries

The `paySalary` function changes the three financial statements by:

- Decreasing cash on the balance sheet
- Decreasing retained earnings (balance sheet)
- Increasing general and administrative expenses (income statement)
- Increasing cash disbursemenets (cash flow statement)

The `paySalary` function also pays the salary to the specified employee via the `transfer` functoin on `USDC.sol`. See the code for the `paySalary` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L51-L67){:target="_blank"}.

#### Action 3: Taking on Long Term Debt

The `takeLongTermDebt` function changes the three financial statements by:

- Increasing cash by the loan amount (balance sheet)
- Increasing current portion of the debt by interest rate * loan amount (balance sheet)
- Increasing long term debt by loan amount - interest rate * loan amount (balance sheet)
- Increasing net borrowings by the loan amount (cash flow statement)

The `takeLongTermDebt` function also receives the loan funds by calling the `transferFrom` function on `USDC.sol` with the relevant parameters. See the code for the `takeLongTermDebt` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L71-L88){:target="_blank"}.

#### Action 4: Paying Interest on Long Term Debt

The `payInterestOnLongTermDebt` function changes the three financial statements by:

- Decreasing cash by the interest amount (balance sheet)
- Decreasing retained earnings by the interest amount (balance sheet)
- Decreasing net interest income by the interest amount (income statement)
- Increasing cash disbursements by the interest amount (cash flow statement)

The `payInterestOnLongTermDebt` function also pays out the interest to the loaner by calling the `transfer` function on `USDC.sol` with the relevant parameters. See the code for the `payInterestOnLongTermDebt` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L92-L108){:target="_blank"}.

#### Action 5: Buying Fixed Assets

When we buy a fixed asset we may be able to buy on credit, meaning we pay some amount of the cost now and pay the rest later. The `buyFixedAssset` function changes the three financial statements by:

- Increasing fixed asset at cost (balance sheet)
- Decreasing cash by the amount paid now (balance sheet)
- Increasing accounts payable by the amount to be paid later (balance sheet)
- Increasing PP&E purchase by the amount paid now (cash flow statement)

The `buyFixedAssset` function also pays out the amount to be paid now to the seller by calling the `transfer` function on `USDC.sol`. See the code for the `buyFixedAssset` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L113-L127){:target="_blank"}.

#### Action 6: Accumulating Depreciation on Fixed Assets

The `depreciateFixedAsset` function changes the three financial statements by:

- Increasing accumulated depreciation by the depreciation amount (balance sheet)
- Increasing inventories by the depreciation amount (balance sheet)

See the code for the `depreciateFixedAsset` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L131-L139){:target="_blank"}.

#### Action 7: Buying Inventory (for instance, raw materials)

Similar to buying a fixed asset, when we buy inventory we may be able to buy on credit, meaning we pay some amount of the cost now and pay the rest later. The `buyInventory` function changes the three financial statements by:

- Decreasing cash by the amount paid now (balance sheet)
- Increasing inventories by the total amount of inventories bought (balance sheet)
- Increasing accounts payable by total amount of inventories - amount paid now (balance sheet)
- Increasing cash disbursements by the amount paid now (cash flow statement)

The `buyInventory` function also pays out the amount to be paid now to the seller by using the `transfer` function on `USDC.sol`. See the code for the `buyInventory` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L143-L157){:target="_blank"}.

#### Action 8: Selling Inventory

When we sell inventory, we will receive some cash now and the rest will be booked as accounts receivables. The `sellInventory` function changes the three financial statements by:

- Increasing cash by amount received now (balance sheet)
- Increasing accounts receivable by amount sold - amount of cash received now (balance sheet).
- Decreasing inventories by cost of goods sold (balance sheet)
- Increasing retained earnings by amount sold - cost of goods sold (balance sheet)
- Increasing net sales by amount sold (income statement)
- Increasing cost of good sold (income statement)
- Increasing cash receipts by amount received now (cash flow statement)

The `sellInventory` function also receives payment from the buyer using the `transferFrom` function on `USDC.sol`. See the code for the `sellInventory` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L161-L181){:target="_blank"}.

#### Action 9: Paying down Accounts Payable

The `payAccountsPayable` function changes the three financial statements by:

- Decreasing cash (balance sheet)
- Decreasing accounts payable (balance sheet)
- Increase cash disbursements (cash flow statement)

The `payAccountsPayable` function also pays out the relevant amount to the owed party using the `transfer` function on `USDC.sol`. See the code for the `payAccountsPayable` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L185-L198){:target="_blank"}.

#### Action 10: Receiving Payments for Accounts Receivable

The `receiveAccountsReceivable` function changes the three financial statements by:

- Increasing cash (balance sheet)
- Decreasing accounts receivable (balance sheet)
- Increasing cash receipts (cash flow statement)

The `receiveAccountsReceivable` function also receives the funds from the ower using the `transferFrom` function on `USDC.sol`. See the code for the `receiveAccountsReceivable` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L202-L215){:target="_blank"}.

#### Action 11: Pay Taxes

The `payTaxes` function changes the three financial statements by:

- Decreasing cash by amount of taxes paid (balance sheet)
- Decreasing retained earnings by amount of taxes paid (balance sheet)
- Increasing income taxes by amount of taxes paid (income statement)
- Increasing income taxes paid by amount of taxes paid (cash flow statement)

The `payTaxes` function pays the government using the `transfer` function on `USDC.sol`. See the code for the `payTaxes` function [here](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/contracts/Accounting.sol#L219-L238){:target="_blank"}.

## Financial Accounting for a Hypothetical Company

We now deploy the `Accounting.sol` smart contract to a local Hardhat blochchain and use it to maintain the books of a hypothetical company, which we will call Stacy's Lemonade Stand. Checkout the `deploy.ts` script in the [Github repository](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts){:target="_blank"} for more details.

#### Stacy Deploys the `Accounting.sol` Contract

([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L91-L93){:target="_blank"}). Initially all the entries in all the statements are 0.

#### Stacy Raises $1,000,000 of Equity from Investors
Stacy calls the `raiseEquity` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L98){:target="_blank"}).  View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.36.10%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.36.20%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.24.37%20AM.png){:target="_blank"} after taking this action.

#### Stacy Pays Herself $5,000 in Salary
Stacy calls the `paySalary` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L102){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.36.42%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.36.51%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.24.47%20AM.png){:target="_blank"} after taking this action.

#### Stacy Takes a 10 Year Loan and Buys a Plot of Land for the Lemonade Stand for $50,000 at a 10% Yearly Interest Rate
Stacy calls the `takeLongTermDebt` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L107){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.37.12%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.37.21%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.24.55%20AM.png){:target="_blank"} after taking this action.

#### Stacy Buys a Jug to Make Her Lemonade In. The Total Cost is $10,000, but She Only Pays for Half Now
Stacy calls the `buyFixedAsset` ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L111){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.37.39%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.37.49%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.25.04%20AM.png){:target="_blank"} after taking this action.

#### Stacy Buys Lemonade Powder and Water, the Raw Materials Needed to Make Lemonade for $500
Stacy calls the `buyInventory` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L115){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.38.17%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.38.26%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.25.11%20AM.png){:target="_blank"} after taking this action.

#### Stacy Makes Lemonade and Sells All of it to Alice $20,000. She Receives Cash for Half of the Lemonade
Stacy calls the `sellInventory` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L120){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.38.47%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.38.55%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.25.22%20AM.png){:target="_blank"} after taking this action.

#### Stacy Pays Off the Other $5,000 She Owes for the Jug
Stacy calls the `payAccountsPayable` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L124){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.39.13%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.39.20%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.25.36%20AM.png){:target="_blank"} after taking this action.

#### Stacy Receives the Other $10,000 She is Owed From Her Customers
Stacy calls the `receiveAccountsReceivable` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L129){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.39.37%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.39.44%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.25.44%20AM.png){:target="_blank"} after taking this action.

#### It's the End of the Fiscal Year. Stacy Pays the Interest She Owes on the Loan She Used to Buy the Plot of Land
Stacy calls the `payInterestOnLongTermDebt` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L133){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.40.03%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.40.11%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.25.50%20AM.png){:target="_blank"} after taking this action.

#### Stacy Accumulates Depreciation on Her Jug
Stacy calls the `depreciateFixedAsset` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L137){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.40.46%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.40.54%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.25.59%20AM.png){:target="_blank"} after taking this action.

#### Stacy Pays Income Taxes at a Rate of 30%
Stacy calls the `payTaxes` function ([code](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/scripts/deploy.ts#L141){:target="_blank"}). View the [balance sheet](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.41.11%20AM.png){:target="_blank"}, [income statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%205.41.17%20AM.png){:target="blank"}, and [cash flow statement](https://github.com/saiakilesh/automated-crypto-accounting/blob/main/pics/Screen%20Shot%202022-04-13%20at%207.26.07%20AM.png){:target="_blank"} after taking this action.

Note the following facts:

- Retained earnings on the balance sheet equals net income on the income statement.
- Cash on the balance sheet equals ending cash balance on the cash flow statement.

This implies that our code works as desired.
