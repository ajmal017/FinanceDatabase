# Finance Database
As a private investor, the sheer amount of information that can be found on the internet is rather daunting. Trying to 
understand what type of companies or ETFs are available is incredibly challenging with there being millions of
companies amd derivatives available on the market. Sure, the most traded companies and ETFs can quickly be found
simply because they are known to the public (for example, Microsoft, Tesla, S&P500 ETF or an All-World ETF). However, 
what else is out there is often unknown.

**This database tries to solve that**. It features 180.000+ symbols containing Equities, ETFs, Funds, Indices, Futures, 
Options, Currencies, Cryptocurrencies and Money Markets. It therefore allows you to obtain a broad overview of sectors,
industries, types of investments and much more.

The aim of this database is explicitly _not_ to provide up-to-date fundamentals or stock data as those can be obtained 
with ease (with the help of this database) by using [FundamentalAnalysis](https://github.com/JerBouma/FundamentalAnalysis), 
[yfinance](https://github.com/ranaroussi/yfinance) or [ThePassiveInvestor](https://github.com/JerBouma/ThePassiveInvestor). 
Instead, it gives insights into the products that exist in each country, industry and sector and gives the most essential information about each product. With this information, you 
can analyse specific areas of the financial world and/or find a product that is hard to find. See for examples
on how you can combine this database, and the earlier mentioned packages the section [Examples](#Examples).

Some key statistics of the database:

| Product           | Quantity  | Sectors   | Industries    | Countries | Exchanges |
| ----------------- | --------- | --------- | ------------- | --------- | --------- |
| Equities          | 84.091    | 16        | 262           | 109       | 79        |
| ETFs              | 15.892    | 268*      | 88*           | 100**     | 44        |
| Funds             | 34.947    | 857*      | 416*          | 100**     | 25        |

| Product           | Quantity  | Exchanges |
| ----------------- | --------- | --------- |
| Indices           | 24.548    | 62        |
| Currencies        | 2.529     | 2         |
| Cryptocurrencies  | 3.624     | 1         |
| Options           | 13.819    | 1         |
| Futures           | 1.173     | 7         |
| Money Markets     | 1.384     | 2         |

\* These numbers refer to families (iShares, Vanguard) and categories (World Stock, Real Estate) respectively.  
\** This is an estimation. Obtaining the country distribution can only be done by collecting data on the underlying 
or by manual search.

## Usage
To access the database you can download the entire repository, but I strongly recommend making use of the package 
closely attached to the database. It allows you to select specific json files as well as search through collected
data with a specific query.

You can install the package with the following steps:
1. `pip install FinanceDatabase`
    - Alternatively, download the 'Searcher' directory.
2. (within Python) `import FinanceDatabase as fd`

The package has the following functions:
- `show_options(product, equities_selection=None)` - gives all available options from the functions below per 
  product (i.e. Equities, Funds) which then can be used to collect data. You can select a sub selection of 
  equities by entering 'countries', 'sectors' or 'industries' for equities_selection.
- `select_cryptocurrencies(cryptocurrency=None)` - with no input gives all cryptocurrencies, with input gives 
the cryptocurrency of choice.
- `select_currencies(currency=None)` - with no input gives all currencies, with input gives 
the currency of choice.
- `select_etfs(category=None)` - with no input gives all etfs, with input gives all etfs of a 
specific category.
- `select_equities(country=None, sector=None, industry=None)` - with no input gives all equities, with input 
gives all equities of a country, sector, industry or a combination of the three.
- `select_funds(category=None)` - with no input gives all funds, with input gives all funds of a 
specific category.
- `select_indices(market=None)` - with no input gives all indices, with input gives all indices of a 
specific market which usually refers to indices in a specific country (like de_market gives DAX).
- `select_other(product)` - gives either all Futures, all Moneymarkets or all Options.
- `search_products(database, query, search='summary', case_sensitive=False, new_database=None)` - with input 
  from the above functions, this function searches for specific values (i.e. the query 'sustainable') in 
  one of the keys of the dictionary (which is by default the summary). It also has the option to enable 
  case-sensitive searching which is off by default.
  
For additional information about each function you can use the build-in help function of Python. For 
example `help(show_options)` returns a general description, the possible input parameters and what is returned 
as output.

For users of the broker **DeGiro**, you are able to find data on the tickers found in the 
[Commission Free ETFs](https://www.degiro.ie/data/pdf/ie/commission-free-etfs-list.pdf) list by selecting either 
`core_selection_degiro_filled` (all data) or `core_selection_degiro_filtered` (filtered by summary) as category 
when using the function `select_etfs`.

## Examples
This section gives a few examples of the possibilities with this package. These are merely a few of the things you
can do with the package, and it only uses yfinance. **As you can obtain a wide range of symbols, pretty much any 
package that requires symbols should work.**

### United States' Airlines
If I wish to obtain all companies within the United States listed under 'Airlines' I can write the 
following code:
````
import FinanceDatabase as fd

airlines_us = fd.select_equities(country='United States', industry='Airlines')
````
Then, I can use packages like [yfinance](https://github.com/ranaroussi/yfinance) to quickly collect data from 
Yahoo Finance for each symbol in the industry like this:
````
from yfinance.utils import get_json
from yfinance import download

airlines_us_fundamentals = {}
for symbol in airlines_us:
    airlines_us_fundamentals[symbol] = get_json("https://finance.yahoo.com/quote/" + symbol)

airlines_us_stock_data = download(list(airlines_us))
```` 
With a few lines of code, I have collected all data from a specific industry within the United States. From here on 
you can compare pretty much any key statistic, fundamental- and stock data. For example, let's plot a simple bar 
chart that gives insights in the Quick Ratios (indicator of the overall financial strength or weakness of a company):
````
import matplotlib.pyplot as plt

for symbol in airlines_us_fundamentals:
    quick_ratio = airlines_us_fundamentals[symbol]['financialData']['quickRatio']
    long_name = airlines_us_fundamentals[symbol]['quoteType']['longName']

    if quick_ratio is None:
        continue

    plt.barh(long_name, quick_ratio)

plt.tight_layout()
plt.show()
``````
Which results in the graph displayed below (as of the 3rd of February 2021). From this graph you can identify 
companies that currently lack enough assets to cover their liabilities (quick ratio < 1), and those that do have 
enough assets (quick ratio > 1). Both too low and too high could make you wonder whether the company adequately 
manages its assets.

![FinanceDatabase](Examples/United_States_Airlines_QuickRatio.png)

### Semiconductors ETFs
If I wish to obtain all ETFs that have something to do with 'semiconductors' I can use the search function which can 
be used the following way:
````
import FinanceDatabase as fd

all_etfs = fd.select_etfs()
semiconductor_etfs = fd.search_products(all_etfs, 'semiconductor')
````
The variable semiconductor_etfs returns all etfs that have the word 'semiconductor' in their summary which usually 
also corresponds to the fact they are targeted around semiconductors. Next, I collect data:
````
semiconductor_etfs_fundamentals = {}
for symbol in semiconductor_etfs:
    semiconductor_etfs_fundamentals[symbol] = get_json("https://finance.yahoo.com/quote/" + symbol)
````
And lastly, I have a look at the YTD returns (as of the 3rd of February 2021) of each ETF to better understand the 
products that are available (for example levered products or specific bear and bull products):
````
for symbol in semiconductor_etfs_fundamentals:
    ytd_return = semiconductor_etfs_fundamentals[symbol]['fundPerformance']['trailingReturns']['ytd']
    long_name = semiconductor_etfs_fundamentals[symbol]['quoteType']['longName']

    if ytd_return is None:
        continue

    plt.barh(long_name, ytd_return)

plt.tight_layout()
plt.xticks([-1, -0.5, 0, 0.5, 1], ['-100%', '-50%', '0%', '50%', '100%'])
plt.show()
````
This results in the following graph which gives _some_ insights in the available semiconductor ETFs. Then with the 
large amount of fundamentals data you can figure out how each ETF differs and what might be worthwhile to invest in.

![FinanceDatabase](Examples/Semiconductors_ETFs_Returns.png)

### Using ThePassiveInvestor
Sometimes, Excel simply offers the best solution to quickly compare a range of ETFs. Therefore, another option is to 
use my program [ThePassiveInvestor](https://github.com/JerBouma/ThePassiveInvestor). The goal of this program is to 
quickly compare ETFs by comparing their most important attributies (i.e. holdings, return, volatility, tracking error). 
Thus what you can do is gather a set of symbols and run it through the program.

As I invest with DeGiro, a great start for me would be by collecting all ETFs that are listed within the Core 
Selection (commission free) of my broker with the following code (or manually obtain them from the json file):
````
import FinanceDatabase as fd

core_selection = fd.select_etfs("core_selection_degiro_filtered")
````
Then I convert the keys of the core_selection into a Series and send it to Excel without index and header.
````
import pandas as pd

tickers = pd.Series(core_selection.keys())
tickers.to_excel('core_selection_tickers.xlsx', index=None, header=None)
````
When you then open the Excel file created you see the following lay-out (which corresponds to the lay-out accepted 
by the program):

![ThePassiveInvestor](Examples/ThePassiveInvestor_Excel.PNG)

Then I can open ThePassiveInvestor program and use the Excel as input. The first input is the Excel that you want to 
be filled with input from your tickers (created by the program). The second input is the file you created above.

![ThePassiveInvestor](Examples/ThePassiveInvestor_Program.PNG)

When you run the program it starts collecting data on each ticker and fills the Excel with data. After the program is 
finished you are able to find an Excel that looks very much like the data you see below. With this data you can 
get an indication whether the ETF is what you are looking for.

![ThePassiveInvestor](Examples/ThePassiveInvestor_GIF.gif)

## Q&A

- **How did you get your data?**
    - Please check the [Methodology](https://github.com/JerBouma/FinanceDatabase/tree/master/Methodology).
- **Is there support for <insert_country>?**
    - Yes, most likely there is as the database includes 109 countries. Please check 
    [here](https://github.com/JerBouma/FinanceDatabase/tree/master/Database/Equities/Countries).
- **When I collect data via yfinance I notice that not all tickers return output, why is that?**
    - Some tickers are merely holdings of companies and therefore do not really have any data attached to them. 
      Therefore, it makes sense that not all tickers return data. If you are still in doubt, search the ticker on 
      Google to see if there is really no data available.
- **How frequently does the Database get updated?**
    - I aim at doing this every few months. The database does not have to get updated frequently because the data 
      collected is only general information. For example, a Sector name hardly changes and companies do not tend to 
      move to another country every few months. Therefore, the data should stay up to date for several months. 
      *If you wish to contribute to updating the database then that is much appreciated. Please check the 
      [Methodology](https://github.com/JerBouma/FinanceDatabase/tree/master/Methodology) for guidance on how.*
- **Do your sector and industry names use the same naming convention as GIC sector**?
    - Not entirely but very similar, it's based on Yahoo Finance's sectors and industries. See industries and 
      sectors. Perhaps a future adjustment could be to make them aligned with GICS.

## Contribution
Projects are bound to have (small) errors and can always be improved. Therefore, I highly encourage you to submit 
issues and create pull requests to improve the package.

The last update to the database is the 3rd of February 2021. I always accept Pull Requests every few months 
to keep the database up to date. Extending the amount of tickers and data is also much appreciated.