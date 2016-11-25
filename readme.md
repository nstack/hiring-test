NStack Coding Test
------------------

The system is intended to simulate receiving and tracking trades in real-time financial markets. A selection of markets execute trades for simple financial instruments (e.g. bonds) in three different currencies: USD, EUR, and GBP. A market will notify us whenever we make a trade, indicating which currency we traded, the size of the trade (quantity), and whether we bought or sold.

We aim to build a simulator that can replay previously saved trade data, in real-time, for each market. These simulators can then be used to test an automated aggregation system system. The aggregator will merge invidual market output into a single multi-market output (i.e. show all trades from all markets), and then aggregate the history of individual trades to produce a running summary of total quantities traded.

We store previous market data in 3 CSV files, `market_a.csv`, `market_b.csv` and `market_c.csv` respectively. The CSV files store a list of trades, in the format:

  ```
  TIMESTAMP,CURRENCY,QTY,DIRECTION
  ```

where

* `TIMESTAMP` is elapsed milli-seconds since Market open (i.e. when the program begins)
* `CURRENCY` is one of USD | EUR | GBP, the three currencies we are trading
* `QTY` is an integer number of instruments, indicating how many we bought or sold
* `DIRECTION` is one of BUY | SELL

Each row corresponds to a single trade that occurred at the given timestamp. A trade shows that we have either *bought* that much qty of the instrument, or *sold* that much.

e.g.

  ```
  1003,USD,2000,BUY
  2422,EUR,1000,SELL
  etc...
  ```

As we buy or sell, our own stock of each instrument goes up or down. For example, if we sell 1000 USD and then sell 2000 more USD, we currently own 3000 USD instruments (we would say we are *long* 3000). If we then sell 2000 USD and then sell 2000 USD more, we are now at -1000 USD. A negative position is entirely valid (we would say we are *short* 1000 - effectively we owe 1000).

The task
--------

1. Using a Haskell iteratee library of your choice (e.g. Pipes, Conduit, Machines, etc.), write a function that given a CSV filename, returns an iteratee producer that replays data from that stored CSV in real-time and emits it downstream.

  e.g. using **Pipes**, you might end up with something like:

  ```haskell
  -- emits in real-time
  replayCsv :: FilePath -> Producer Order IO ()
  ```

2. Using the producers from (1), write a program that runs all the market simulators concurrently and combines the inputs from them into a combined 'multi-market' view, emitting all trades from the input markets in real-time. You should then transform the stream of combined orders into the aggregated state of the market given all orders up to that point,  e.g. total cumulative positions for all three currency symbols. Every 1 seconds the program should write the current aggregated state to stdout. A negative number indicates a 'short' position, e.g. we have sold more than we have bought. The output format should be `CURRENCY QTY` for each currency, comma separated, e.g.

  ```
  USD 2000, EUR -1000, GBP 3000
  USD 1000, EUR -1000, GBP 3000
  USD 1000, EUR 0, GBP 3000
  ```

  e.g. again using **Pipes**, you might end up with:

  ```haskell
  -- Prints to std out every 1s
  aggregateAndLog :: [Producer Order IO ()] -> IO ()
  ```

3. Write an application that takes a list of csv files as command line arguments, and replays those as markets using (1) and aggregates the output through (2)

Submission
----------

Your solution should take the form of a Haskell program with a stack build file, such that it can be built using `stack build`. To initialise the stack project, you can use the command:

  ```
  stack new nhire
  ```

Your application should be called **nhire-exe**. It should produce the correct output when run with:

  ```
  nhire-exe market_a.csv market_b.csv market_c.csv
  ```
