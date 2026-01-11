# Design Patterns -> Behavioral -> Strategy

Strategy is a behavioral design pattern that lets you define  
a family of algorithms, put each of them into a separate class,  
and make their objects interchangeable.

It follows OCP OR in other words it is the same as OCP in S[O]LID.

## Problem

---

## Solution

**Example #1**

```TypeScript
interface ICheckpoint {
    // Defines a Checkpoint type
}

// A strategy interface
interface IRouteStrategy {
    // Builds route and return an array of checkpoints.
    buildRoute(origin, destination): Checkpoint[];
}

// Concrete strategies - Air, Road & Train

class AirTravelStrategy implements IRouteStrategy {
    buildRoute(origin, destination): Checkpoint[] {
        // 
    }
}

class RoadTravelStrategy implements IRouteStrategy {
    buildRoute(origin, destination): Checkpoint[] {
        // 
    }
}

class TrainTravelStrategy implements IRouteStrategy {
    buildRoute(origin, destination): Checkpoint[] {
        // 
    }
}

// Context class Navigator which uses one of the concrete strategies to build a route.
// Concrete strategy class instance will be provided by application/client which uses
// this context class.
// It follows OCP.
class Navigator {
    constructor(routeStrategy: IRouteStrategy) {
        //
    }

    buildRoute() {
        return routeStrategy.buildRoute();
    }
}

// Usage in application.

// Create the concrete strategy instance as per the user input.
const routeStrategy = new TrainTravelStrategy();

const navigator = new Navigator(routeStrategy);
const routeCheckpoints = navigator.buildRoute();
```
