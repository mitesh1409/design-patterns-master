# Design Patterns - Singleton

Singleton -> Creational Design Pattern

Singleton design pattern is used whenever you want  
only one instance of the class to be created.  
Singleton instance is available globally so that everyone can use it.

Basically, any situation where multiple instances of a class  
can lead to some issue or conflict,  
you can think of using Singleton design pattern.  

Examples are:  

- Database Connection Instance
- Application Logger Instance
- Application Configuration Instance

```TypeScript
// Bad - Multiple loggers creating chaos
const logger1 = new Logger();
const logger2 = new Logger(); // another logger, writing to same file? :(

// Good - Singleton, single logger everyone can use.
const logger = Logger.getInstance();
```

**Pros**  

- Guaranteed single instance
    Prevents multiple copies of resources/state.

- Global access point
    Easy to use anywhere in application.

**Cons**  

- Testing difficulties
    Can't easily mock for unit tests.
    Complex test setup.

- Threading issues
    Race conditions during instance creation.
    Need thread-safe implementation.

Use Singleton design pattern when you genuinely need single instance guarantee,  
not when you just want a global state.

Example  

```TypeScript
// Logger.ts

export default class Logger {
    // 1. Hold the single instance in a private static property
    private static instance: Logger;

    // 2. Make the constructor private so no one can call 'new Logger()'
    private constructor() {
        // Optional: Initialization logic (e.g., setting up log levels)
    }

    // 3. The static method to control access to the instance
    public static getInstance(): Logger {
        if (!Logger.instance) {
            Logger.instance = new Logger();
        }
        return Logger.instance;
    }

    private getTimestamp(): string {
        return new Date().toISOString();
    }

    // Logging methods
    public log(...args: any[]): void {
        console.log(`[${this.getTimestamp()}] LOG:`, ...args);
    }

    public info(...args: any[]): void {
        console.info(`[${this.getTimestamp()}] INFO:`, ...args);
    }

    public error(...args: any[]): void {
        console.error(`[${this.getTimestamp()}] ERROR:`, ...args);
    }
}

// Usage:
const logger = Logger.getInstance();
logger.log("Application started");
logger.error("Something went wrong", { code: 500 });

// This will be the same instance as 'logger' above
const anotherLogger = Logger.getInstance(); 
console.log(logger === anotherLogger); // true
```

**How the Singleton Pattern Works**  

The Singleton pattern restricts the instantiation of a class to one "single" instance. This is useful when exactly one object is needed to coordinate actions across the system.

**The Three Core Ingredients:**  

1. Private Static Property (instance):  
This acts as the storage for our single instance.  
Because it’s static, it belongs to the class itself, not to any specific object.

2. Private Constructor:  
By marking the constructor private, you prevent the use of the new keyword  
outside of the class. If you try `const log = new Logger()`,  
TypeScript will throw an error.

3. Static Getter Method (`getInstance`):  
This is the "gatekeeper".  
If the instance doesn't exist yet, it creates one.  
If it already exists, it simply returns the existing one.

## References

* [7 Design Patterns EVERY Developer Should Know](https://www.youtube.com/watch?v=BJatgOiiht4)
