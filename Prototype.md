# Design Patterns -> Creational -> Prototype

## Problem

You want to create an exact copy of an object.

How would you do it?  

You can create a new object of the same class.  
Then you go through all the fields of the original object and  
copy them one by one in the new object.  

Problems are:  

- You can't access private fields and so can't copy them.
- Your code becomes dependent on that class and its dependencies as well.
- You can't get an exact idea about the injected dependencies (what exact type of object was injected).

In short copying/cloning an object is very tough if not impossible.

---

## Solution

Prototype = Clone

The Prototype pattern delegates the cloning process to the actual objects that are being cloned. The pattern declares a common interface for all objects that support cloning. This interface lets you clone an object without coupling your code to the class of that object. Usually, such an interface contains just a single clone method.

The implementation of the clone method is very similar in all classes. The method creates an object of the current class and carries over all of the field values of the old object into the new one. You can even copy private fields because most programming languages let objects access private fields of other objects that belong to the same class.

An object that supports cloning is called a prototype. When your objects have dozens of fields and hundreds of possible configurations, cloning them might serve as an alternative to subclassing.

Here’s how it works: you create a set of objects, configured in various ways. When you need an object like the one you’ve configured, you just clone a prototype instead of constructing a new object from scratch.

The Prototype pattern is an alternative way to implement inheritance,  
instead of inheriting functionality from a class it comes from an object  
that has already been created.  

Prototype is a creational design pattern that lets you copy/clone  
existing objects without making your code dependent on their classes.

JavaScript supports Prototypal inheritance out of the box.

**Example #1**  

```TypeScript
// Base credit card instance with some defaults.
const baseCreditCard = {
    company: 'VISA',
    number: '1234 5678 9012',
    expiryMonth: 12,
    expiryYear: 2030,
    customerName: 'John Wick',

    makePayment() {
        //
    },

    payBill() {
        //
    },

    getCreditBalance() {
        //
    },

    generateStatement() {
        //
    },

    getStatement() {
        //
    },

    applyOverdueCharges() {
        //
    },

    applyInterest() {
        //
    }
};

// We cloned baseCreditCard.
const customerCreditCard = Object.create(baseCreditCard);

console.log('Initial customerCreditCard', customerCreditCard);

// Modifying base object with customer specific details.
customerCreditCard.company = 'Mastercard';
customerCreditCard.number = '1111 2222 3333';
customerCreditCard.customerName = 'Peter Parker';
customerCreditCard.addRewards = function() {
    console.log('Add rewards...');
}
customerCreditCard.redeemRewards = function() {
    console.log('Redeem rewards...');
}

console.log('After modification customerCreditCard', customerCreditCard);
```

---

**Example #2**  

```TypeScript
interface Prototype {
    clone(): Prototype
}

class CreditCard implements Prototype {
    company: String;
    number: String;
    expiryMonth: Number;
    expiryYear: Number;
    customerName: String;

    constructor(company, number, expiryMonth, expiryYear, customerName) {
        this.company = company;
        this.number = number;
        this.expiryMonth = expiryMonth;
        this.expiryYear = expiryYear;
        this.customerName = customerName;
    }

    makePayment() {
        //
    },

    payBill() {
        //
    },

    getCreditBalance() {
        //
    },

    generateStatement() {
        //
    },

    getStatement() {
        //
    },

    applyOverdueCharges() {
        //
    },

    applyInterest() {
        //
    }

    clone(): CreditCard {
        // // Using JSON.parse(JSON.stringify(targetObject))
        // const clonedInstance = JSON.parse(JSON.stringify(this));

        // OR

        // The structuredClone() method of the Window interface
        // The structuredClone() global function in Node.js
        const clonedInstance = structuredClone(this);

        // OR

        // // Library like Lodash's _.cloneDeep()
        // const clonedInstance = _.cloneDeep(this);

        return clonedInstance;
    }
}

// Usage
const visaCreditCard = new CreditCard('VISA', '123456789012', 12, 2030, 'John Wick');
const mastercardCreditCard = new CreditCard('Mastercard', '111122223333', 12, 2030, 'John Wick');
const rupayCreditCard = new CreditCard('Rupay', '999988887777', 12, 2030, 'John Wick');

// When a customer selects VISA credit card, we can easily clone visaCreditCard and then do 
// customer specific changes on that instance.
const customerCard = visaCreditCard.clone();
customerCard.number = '222233335555';
customerCard.customerName = 'Manoj Bajpayee';
```
