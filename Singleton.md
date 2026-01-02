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
