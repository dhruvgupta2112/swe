Explanation and implementation of the some creational design patterns in Java:

# 1) Factory Method Pattern

## Explanation:
- The Factory Method pattern defines an interface for creating an object but lets subclasses alter the type of objects that will be created.
- Instead of instantiating objects directly using new, the client calls a factory method, which creates and returns an instance.

## Implementation:

```java
// Step 1: Create an interface
interface Product {
    void use();
}

// Step 2: Concrete implementations of the Product
class ConcreteProductA implements Product {
    public void use() {
        System.out.println("Using Product A");
    }
}

class ConcreteProductB implements Product {
    public void use() {
        System.out.println("Using Product B");
    }
}

// Step 3: Factory class
abstract class ProductFactory {
    public abstract Product createProduct();
}

class ProductAFactory extends ProductFactory {
    public Product createProduct() {
        return new ConcreteProductA();
    }
}

class ProductBFactory extends ProductFactory {
    public Product createProduct() {
        return new ConcreteProductB();
    }
}

// Step 4: Demonstration
public class FactoryMethodDemo {
    public static void main(String[] args) {
        ProductFactory factoryA = new ProductAFactory();
        Product productA = factoryA.createProduct();
        productA.use();

        ProductFactory factoryB = new ProductBFactory();
        Product productB = factoryB.createProduct();
        productB.use();
    }
}
```

### Output:
```
Using Product A
Using Product B
```

# 2) Abstract Factory Pattern

## Explanation:
- The Abstract Factory pattern provides an interface for creating families of related objects without specifying their concrete classes.
- Unlike the Factory Method pattern, it involves multiple factories, each responsible for creating a set of related objects.

## Implementation:

```java
// Step 1: Define abstract product interfaces
interface Chair {
    void sitOn();
}

interface Table {
    void placeItems();
}

// Step 2: Concrete product implementations
class ModernChair implements Chair {
    public void sitOn() {
        System.out.println("Sitting on a modern chair.");
    }
}

class VintageChair implements Chair {
    public void sitOn() {
        System.out.println("Sitting on a vintage chair.");
    }
}

class ModernTable implements Table {
    public void placeItems() {
        System.out.println("Placing items on a modern table.");
    }
}

class VintageTable implements Table {
    public void placeItems() {
        System.out.println("Placing items on a vintage table.");
    }
}

// Step 3: Abstract Factory
interface FurnitureFactory {
    Chair createChair();
    Table createTable();
}

// Step 4: Concrete Factories
class ModernFurnitureFactory implements FurnitureFactory {
    public Chair createChair() {
        return new ModernChair();
    }
    
    public Table createTable() {
        return new ModernTable();
    }
}

class VintageFurnitureFactory implements FurnitureFactory {
    public Chair createChair() {
        return new VintageChair();
    }

    public Table createTable() {
        return new VintageTable();
    }
}

// Step 5: Demonstration
public class AbstractFactoryDemo {
    public static void main(String[] args) {
        FurnitureFactory modernFactory = new ModernFurnitureFactory();
        Chair modernChair = modernFactory.createChair();
        Table modernTable = modernFactory.createTable();
        
        modernChair.sitOn();
        modernTable.placeItems();

        FurnitureFactory vintageFactory = new VintageFurnitureFactory();
        Chair vintageChair = vintageFactory.createChair();
        Table vintageTable = vintageFactory.createTable();

        vintageChair.sitOn();
        vintageTable.placeItems();
    }
}
```

### Output:
```
Sitting on a modern chair.
Placing items on a modern table.
Sitting on a vintage chair.
Placing items on a vintage table.
```

# 3) Singleton Pattern

## Explanation:
- The Singleton pattern ensures that a class has only one instance and provides a global access point to it.
- It is useful for managing shared resources, such as database connections, configurations, etc.

## Implementation:

```java 
// Step 1: Singleton Class
class Singleton {
    // Private static variable to hold the single instance
    private static Singleton instance;

    // Private constructor prevents instantiation from outside
    private Singleton() {}

    // Public static method to provide global access to the instance
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) { // Thread safety
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    public void showMessage() {
        System.out.println("Singleton instance invoked!");
    }
}

// Step 2: Demonstration
public class SingletonDemo {
    public static void main(String[] args) {
        Singleton singleton1 = Singleton.getInstance();
        Singleton singleton2 = Singleton.getInstance();

        singleton1.showMessage();

        // Check if both instances are the same
        System.out.println("Are both instances same? " + (singleton1 == singleton2));
    }
}
```

### Output:
```
Singleton instance invoked!
Are both instances same? true
```
