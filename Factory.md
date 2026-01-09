# Design Patterns -> Creational -> Factory

## Problem

Imagine that you’re creating a logistics management application. The first version of your app can only handle transportation by trucks, so the bulk of your code lives inside the Truck class.

After a while, your app becomes pretty popular. Each day you receive dozens of requests from sea transportation companies to incorporate sea logistics into the app.

Great news, right? But how about the code? At present, most of your code is coupled to the Truck class. Adding Ships into the app would require making changes to the entire codebase. Moreover, if later you decide to add another type of transportation to the app, you will probably need to make all of these changes again.

As a result, you will end up with pretty nasty code, riddled with conditionals that switch the app’s behavior depending on the class of transportation objects.

```TypeScript
// Initially the app only supported road logistics and transportation handled by trucks only.

class RoadLogistics {
    // ...

    planDelivery() {
        // ...
    }

    // ...
}

class Truck {
    // ...

    deliver() {
        // ...
    }

    // ...
}

// Usage in application

// The app only supported road logistics.
// So RoadLogistics & Truck concrete classes are used everywhere.

// Without using Factory pattern, we will have to write conditional code like this.
let transportation = null;

if (logistics === 'road') {
    transportation = new Truck();
} elseif (logistics === 'sea') {
    transportation = new Ship();
} elseif (logistics === 'air') {
    transportation = new Plane();
} else {
    transportation = null;
}
```

---

## Solution

Factory Method is a creational design pattern that provides an interface for creating objects in a  
superclass, but allows subclasses to alter the type of objects that will be created.

```TypeScript
// 1. The Product interface declares the operations that all 
//    concrete products must implement.
interface Transport {
    deliver(): void;
}

// 2. Concrete Products provide various implementations of the 
//    Product interface.
class Truck implements Transport {
    deliver(): void {
        console.log("Delivering by land in a box via Truck.");
    }
}

class Ship implements Transport {
    deliver(): void {
        console.log("Delivering by sea in a container via Ship.");
    }
}

// 3. The Creator class (Logistics) declares the factory method 
//    that returns an object of a Product class.
abstract class Logistics {
    // This is the Factory Method
    public abstract createTransport(): Transport;

    // The Creator's primary responsibility isn't creating products. 
    // It usually contains some core business logic that relies on Product objects.
    public planDelivery(): void {
        // Call the factory method to create a Product object.
        const transport = this.createTransport();
        // Now, use the product.
        console.log("Logistics: The same creator's code has just started the delivery.");
        transport.deliver();
    }
}

// 4. Concrete Creators override the factory method to change 
//    the resulting product's type.
class RoadLogistics extends Logistics {
    public createTransport(): Transport {
        return new Truck();
    }
}

class SeaLogistics extends Logistics {
    public createTransport(): Transport {
        return new Ship();
    }
}

/**
 * Usage (The Client Code)
 * The client code works with an instance of a concrete creator through 
 * its base interface (Logistics).
 */
function runBusinessLogic(creator: Logistics) {
    // ...
    creator.planDelivery();
    // ...
}

console.log("App: Launched with RoadLogistics.");
runBusinessLogic(new RoadLogistics());

console.log("\nApp: Launched with SeaLogistics.");
runBusinessLogic(new SeaLogistics());
```

### Why this is better

* **Decoupling:** The `runBusinessLogic` function doesn't know if it's dealing with a `Truck` or a `Ship`. It only knows it's dealing with something that follows the `Logistics` contract.

* **Open/Closed Principle:** If you want to add **AirLogistics** (Planes), you just create a new subclass of `Logistics` and `Transport`. You **don't** have to change any of the existing logic in your `Logistics` base class or your client code.

* **Single Responsibility:** The code that creates products is moved to one place (the concrete creators), making the rest of the application cleaner.

> The "Magic" of the Factory Method is that the base `Logistics` class handles the delivery process, but it **defers** the decision of *which* vehicle to use to its subclasses.
