# Design Patterns -> Creational -> Builder

## Problem

Imagine a complex object that requires laborious, step-by-step initialization of many fields and  
nested objects. Such initialization code is usually buried inside a monstrous constructor with  
lots of parameters. Or even worse: scattered all over the client code.

---

## Solution

Builder is a creational design pattern that lets you construct complex objects step by step.  
The pattern allows you to produce different types and representations of an object using the  
same construction code.

With the Builder pattern we create the object step by step using methods rather than a constructor,  
and we can even delegate the building logic into entirely different class.

**Example #1**  

```TypeScript

class Sandwich {
    constructor(
        private bread: String,
        private cheese: Boolean,
        private tomato: Boolean,
        private onion: Boolean,
        private capsicum: Boolean,
        private cucumbers: Boolean,
        private bellPepper: Boolean,
        private jalapenos: Boolean,
        private olives: Boolean,
        private pickle: Boolean,
        private lettuce: Boolean,
        private takeaway: Boolean,
        private blackPepperPowder: Boolean,
        private salt: Boolean
    ) {}

    // We can have setter methods to set all the ingredients to make a Sandwich.
    // And getter methods to preview a Sandwich.

    setBread(bread: String): Sandwich {
        this.bread = bread;
        return this;
    }

    getBread(): String {
        return this.bread;
    }

    setCheese(value: Boolean): Sandwich {
        this.cheese = value;
        return this;
    }

    getCheese(): Boolean {
        return this.cheese;
    }

    // and so on...
}

// Usage in application

const sandwich = new Sandwich();
sandwich
    .setBread('Multigrain Honeyoats')
    .setCheese(true)
    .setTomato(true)
    .setOnion(true)
    // ...
    .setTakeaway(true);
```

---

**Example #2**  

```
interface IHouse is
    // An empty interface to define a common type for all kinds of House objects.

class SimpleHouse implements IHouse is
    private field walls
    private field door
    private field floor
    private field ceiling
    private field windows

class HouseWithGarden implements IHouse is
    private field walls
    private field door
    private field floor
    private field ceiling
    private field windows
    private field garden

class HouseWithGarage implements IHouse is
    private field walls
    private field door
    private field floor
    private field ceiling
    private field windows
    private field garage

class HouseWithSwimmingPool implements IHouse is
    private field walls
    private field door
    private field floor
    private field ceiling
    private field windows
    private field swimmingPool

interface IHouseBuilder is
    method buildWalls()
    method buildDoor()
    method buildFloor()
    method buildCeiling()
    method buildWindows()
    method buildGarden()
    method buildGarage()
    method buildSwimmingPool()

class HouseBuilder implements IHouseBuilder is
    private field house: IHouse

    constructor HouseBuilder() is

    // A fresh builder instance initialized with a blank house
    // object which it uses in further assembly.
    method init(house: IHouse)
        this.house = house

    // The reset method clears the object being built. Helps to restart the building process.
    method reset() is
        this.house = null

    method buildWalls() is
        // Build walls

    method buildDoor() is
        // Build door

    method buildFloor() is
        // Build floor

    method buildCeiling() is
        // Build ceiling

    method buildWindows() is
        // Build windows

    method buildGarden() is
        // Build garden

    method buildGarage() is
        // Build garage

    method buildSwimmingPool() is
        // Build swimming pool

    method getHouse(): IHouse is
        house = this.house
        this.reset()
        return house

class Director is
    method constructSimpleHouse(builder: IHouseBuilder): IHouse is
        // This will build a SimpleHouse
        builder.init(SimpleHouse)
        builder.buildWalls()
        builder.buildFloor()
        builder.buildCeiling()
        builder.buildDoor()
        builder.buildWindows()
        return builder.getHouse()

    method constructHouseWithGarden(builder: IHouseBuilder): IHouse is
        // This will build a HouseWithGarden
        builder.init(HouseWithGarden)
        builder.buildWalls()
        builder.buildFloor()
        builder.buildCeiling()
        builder.buildDoor()
        builder.buildWindows()
        builder.buildGarden()
        return builder.getHouse()

    method constructHouseWithGarage(builder: IHouseBuilder): IHouse is
        // This will build a HouseWithGarage
        builder.init(HouseWithGarage)
        builder.buildWalls()
        builder.buildFloor()
        builder.buildCeiling()
        builder.buildDoor()
        builder.buildWindows()
        builder.buildGarage()
        return builder.getHouse()

    method constructHouseWithSwimmingPool(builder: IHouseBuilder): IHouse is
        // This will build a HouseWithSwimmingPool
        builder.init(HouseWithSwimmingPool)
        builder.buildWalls()
        builder.buildFloor()
        builder.buildCeiling()
        builder.buildDoor()
        builder.buildWindows()
        builder.buildSwimmingPool()
        return builder.getHouse()

// Usage in application

director = new Director()

houseBuilder = new HouseBuilder()

// Build a simple house
simpleHouse = director.constructSimpleHouse(houseBuilder)

// Build a house with garden
houseWithGarden = director.constructHouseWithGarden(houseBuilder)
```
