# Design Patterns -> Structural -> Adapter

Adapter is a structural design pattern that allows objects  
with incompatible interfaces to collaborate.

It is also known as "Wrapper".

When you are integrating 3rd party libraries or APIs that  
don't quite match what your code expects.

## Problem

---

## Solution

**Example #1**

```TypeScript
// Interface for the stocks data in XML format that is used by application.
interface IStocksData {
    // XML Format
}

// Application uses stocks data in the XML format.
class StocksData implements IStocksData {
    // XML Format
}

// Interface for the stocks data in JSON format that is required by the Magic Analytics library (3rd party).
interface IMagicAnalytics {
    // JSON Format
}

class MagicAnalyticsAdaptor implements IMagicAnalytics {
    // Takes an instance of StocksData and converts it into
    // JSON format that is supported by Magic Analytics.
}

// Application then uses MagicAnalyticsAdaptor to work with Magic Analytics.
```
