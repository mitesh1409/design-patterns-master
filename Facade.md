# Design Patterns -> Structural -> Facade

Facade is basically just a simplified API to hide other low-level  
details in your codebase.

Facade = setting up a pretty front to hide everything behind it.

Client code interacts with Facade to get things done without worrying  
about the low level details.

Use Facade pattern when you have a mess of complex subsystems that you need to simplify.

## Problem

---

## Solution

**Example #1**

```TypeScript
class PlumbingSystem {
    setPressure(p: number) {}
    turnOn() {}
    turnOff() {}
}

class ElectricalSystem {
    setVoltage(v: number) {}
    turnOn() {}
    turnOff() {}
}

// Facade
// This is a wrapper class that hides all the low level details.
// Client will use this class to manage house operations (plumbing, electrical etc.).
class House {
    private plumbing = new PlumbingSystem();
    private electrical = new ElectricalSystem();

    public turnOnSystems() {
        this.plumbing.setPressure(5);
        this.plumbing.turnOn();

        this.electrical.setVoltage(10);
        this.electrical.turnOn();
    }

    public turnOffSystems() {
        this.plumbing.turnOff();
        this.electrical.turnOff();
    }
}

// Usage in application.

// Setup a house.
const house = new House();
house.turnOnSystems(); // Behind the scenes House takes care of everything required to turn on all the systems. Ugly details are hidden.

// Turn off house when not in use.
house.turnOffSystems(); // Behind the scenes House takes care of everything required to turn off all the systems. Ugly details are hidden.
```

---

**Example #2**

Almost every package that we install in JavaScript application can be  
considered a Facade in some way.  
We directly use the APIs without worrying about the inner details or  
how it works internally.

---

**Example #3**

For example in a quick commerce application, when customer places an order  
and complete the payment, we need to do the following:  

- process payment (PaymentProcessor => collectPayment())
- place an order (Order => create())
- update inventory (Inventory => update())
- assign a delivery partner (DeliveryPartner => assignOrder())
- notify customer (CustomerNotification => notify())

```TypeScript

// Without Facade

const paymentProcessor = new PaymentProcessor();
const order = new Order();
const inventory = new Inventory();
const deliveryPartner = new DeliveryPartner();
const customerNotification = CustomerNotification();

// Process payment
paymentProcessor.collectPayment();
// Create an order
order.create();
// Update inventory
inventory.update();
// Assign a delivery partner
deliveryPartner.assignOrder();
// Notify customer
customerNotification.notify();

// With Facade

class OrderFacade {
    private paymentProcessor = new PaymentProcessor();
    private order = new Order();
    private inventory = new Inventory();
    private deliveryPartner = new DeliveryPartner();
    private customerNotification = CustomerNotification();

    public completeOrder() {
        // Process payment
        paymentProcessor.collectPayment();
        // Create an order
        order.create();
        // Update inventory
        inventory.update();
        // Assign a delivery partner
        deliveryPartner.assignOrder();
        // Notify customer
        customerNotification.notify();
    }
}

const orderFacade = new OrderFacade();
orderFacade.completeOrder();
```
