# my-throttler

Very basic, zero-dependency, Promise-based rate limiter.

```js
// Always resolve, never reject.
// If getLatestPriceImpl() is resolved, resolve to a {ticker,value} object.
// If getLatestPriceImpl() is rejected, resolve to a {ticker,err} object.
async function getLatestPrice(ticker) {
  try {
    const value = await getLatestPriceImpl(ticker);
    return {ticker, value};
  } catch (err) {
    return {ticker, err};
  }
}

// 3 requests allowed per minute
const requestsPerPeriod = 3;
const period = 60 * 1000;
const shouldLog = true;
const thr = throttler(requestsPerPeriod, period, shouldLog);

const tickers = ["MSFT", "GOOG", "WMT", "TXN", "KO", "INTC"];
const promises = tickers.map((ticker) => {
  const optionalName = ticker;
  const promiseSupplier = () => {
    // something that returns a Promise
    // e.g.:
    return getLatestPrice(ticker);
  };
  // Add the promiseSupplier to the throttler,
  // with an optional name (that will be used in the logging,
  // if logging is turned on).
  // The return value is a Promise that will resolve
  // when the promiseSupplier's Promise resolves
  // (but within the constraints of the rate limiting).
  return thr.add(promiseSupplier, optionalName);
});

const results = await Promise.all(promises);
```
