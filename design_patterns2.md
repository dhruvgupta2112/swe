# Design Patterns (Continued)

## 1. Adapter Pattern

### Problem
The Adapter pattern is primarily used when you need to integrate a legacy or third-party system (whose interface you can't modify) with your new system. The goal is to adapt the interface of one class to another expected interface.

### Working of the pattern:
- You create an adapter class that "wraps" the object you want to adapt, implementing the desired interface.
- This allows you to call the methods of the adapted object through the standard interface.

### Code example:

```java
// Target interface (expected by the client code)
interface Target {
    void request();
}

// Adaptee (legacy system with incompatible interface)
class OldSystem {
    public void specificRequest() {
        System.out.println("Specific request from OldSystem.");
    }
}

// Adapter class that converts the interface
class Adapter implements Target {
    private OldSystem oldSystem;

    public Adapter(OldSystem oldSystem) {
        this.oldSystem = oldSystem;
    }

    // Adapter method
    @Override
    public void request() {
        oldSystem.specificRequest();  // Delegating to the legacy system
    }
}

// Client code
public class AdapterExample {
    public static void main(String[] args) {
        OldSystem oldSystem = new OldSystem();
        Target target = new Adapter(oldSystem);  // Using the adapter
        target.request();  // Adapter converts the call to the legacy system's specificRequest()
    }
}
```

### Explanation:
- **Target** is the interface the client code expects.
- **OldSystem** is the class you cannot change.
- **Adapter** is the class that implements `Target` and delegates method calls to `OldSystem`.

### Use Case:
- When you need to integrate legacy or third-party libraries into your code without modifying their implementation.

---

## 2. Bridge Pattern

### Problem:
The Bridge pattern is used when you have two classes that should be independent but need to be linked. Instead of inheriting, you separate the abstraction (interface) and implementation (details), allowing them to evolve independently.

### Working of the pattern:
- The **Abstraction** defines the higher-level interface.
- The **Implementor** defines the interface for the implementation.
- The **RefinedAbstraction** extends the abstraction.
- The **ConcreteImplementor** provides the actual implementation.

### Code example:

```java
// Implementor (defines low-level implementation)
interface DrawingAPI {
    void drawCircle(double radius, double x, double y);
}

// Concrete Implementor 1
class ConcreteDrawingAPI1 implements DrawingAPI {
    @Override
    public void drawCircle(double radius, double x, double y) {
        System.out.println("API1: Drawing a circle at (" + x + ", " + y + ") with radius " + radius);
    }
}

// Concrete Implementor 2
class ConcreteDrawingAPI2 implements DrawingAPI {
    @Override
    public void drawCircle(double radius, double x, double y) {
        System.out.println("API2: Drawing a circle at (" + x + ", " + y + ") with radius " + radius);
    }
}

// Abstraction (high-level interface)
abstract class Shape {
    protected DrawingAPI drawingAPI;  // Reference to implementor

    protected Shape(DrawingAPI drawingAPI) {
        this.drawingAPI = drawingAPI;
    }

    public abstract void draw();
    public abstract void resizeByPercentage(double percent);
}

// RefinedAbstraction (extends abstraction)
class Circle extends Shape {
    private double x, y, radius;

    public Circle(double x, double y, double radius, DrawingAPI drawingAPI) {
        super(drawingAPI);  // Call to the parent constructor to set the drawingAPI
        this.x = x;
        this.y = y;
        this.radius = radius;
    }

    @Override
    public void draw() {
        drawingAPI.drawCircle(radius, x, y);  // Delegates drawing to the concrete API
    }

    @Override
    public void resizeByPercentage(double percent) {
        radius *= (1 + percent / 100);  // Resize the circle
    }
}

// Client code
public class BridgeExample {
    public static void main(String[] args) {
        Shape circle1 = new Circle(5, 10, 2, new ConcreteDrawingAPI1());
        Shape circle2 = new Circle(10, 15, 3, new ConcreteDrawingAPI2());

        circle1.draw();
        circle2.draw();
    }
}
```

### Explanation:
- The **Abstraction** is the `Shape` class, which holds a reference to the `DrawingAPI`.
- **ConcreteImplementors** (like `ConcreteDrawingAPI1` and `ConcreteDrawingAPI2`) provide the actual drawing functionality.
- The **RefinedAbstraction** (`Circle`) extends `Shape` and utilizes the `DrawingAPI` to delegate drawing.

### Use Case:
- When you have multiple classes with multiple variants of an operation (like drawing different shapes using different drawing APIs) and need flexibility to change the abstraction or implementation independently.

---

## 3. Visitor Pattern

### Problem:
The Visitor pattern lets you add new operations to a class hierarchy without modifying the classes. You separate algorithms from the objects on which they operate, making it easier to introduce new operations.

### Working of the pattern:
- Define a `Visitor` interface that declares a `visit()` method for each type of object it can visit.
- Concrete visitors implement these `visit()` methods.
- Each object in the class hierarchy (like `Circle`, `Rectangle`) has an `accept()` method that calls the visitor's corresponding `visit()` method.

### Code example:

```java
// Element Interface
interface Shape {
    void accept(Visitor visitor);
}

// Concrete Element 1
class Circle implements Shape {
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);  // Calls visitor's visit method for Circle
    }
}

// Concrete Element 2
class Rectangle implements Shape {
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);  // Calls visitor's visit method for Rectangle
    }
}

// Visitor Interface
interface Visitor {
    void visit(Circle circle);
    void visit(Rectangle rectangle);
}

// Concrete Visitor (e.g., area calculator)
class AreaCalculator implements Visitor {
    @Override
    public void visit(Circle circle) {
        System.out.println("Calculating area for Circle");
    }

    @Override
    public void visit(Rectangle rectangle) {
        System.out.println("Calculating area for Rectangle");
    }
}

// Client code
public class VisitorExample {
    public static void main(String[] args) {
        Shape[] shapes = new Shape[] { new Circle(), new Rectangle() };
        Visitor areaCalculator = new AreaCalculator();
        
        for (Shape shape : shapes) {
            shape.accept(areaCalculator);  // Visitor handles each shape
        }
    }
}
```

### Explanation:
- **Shape** is the `Element` interface with an `accept()` method.
- **Visitor** is the interface with `visit()` methods for each shape type.
- **AreaCalculator** is a concrete visitor that implements the algorithm (calculating the area) for different `Shape` types.

### Use Case:
- When you need to perform operations on objects of different classes, but you don’t want to add methods to those classes directly (e.g., for calculations, rendering, or reporting).

---

## 4. Command Pattern

### Problem:
The Command pattern turns requests into objects, allowing you to parameterize clients with different requests, queue requests, and support undoable operations.

### Working of the pattern:
- The **Command** interface declares an `execute()` method.
- Concrete command classes implement this interface and define the action.
- The **Invoker** is responsible for calling the command, and the **Receiver** does the actual work.

### Code example:

```java
// Command Interface
interface Command {
    void execute();
}

// Concrete Command
class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.turnOn();  // Executes the command
    }
}

// Receiver class (does the actual work)
class Light {
    public void turnOn() {
        System.out.println("Light is ON");
    }

    public void turnOff() {
        System.out.println("Light is OFF");
    }
}

// Invoker class (requests the command)
class RemoteControl {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void pressButton() {
        command.execute();  // Executes the command
    }
}

// Client code
public class CommandExample {
    public static void main(String[] args) {
        Light light = new Light();
        Command lightOn = new LightOnCommand(light);  // Command object

        RemoteControl remote = new RemoteControl();  // Invoker
        remote.setCommand(lightOn);  // Setting command
        remote.pressButton();  // Invoker triggers command execution
    }
}
```

### Explanation:
- **Command** interface defines an `execute()` method.
- **LightOnCommand** is a concrete command that calls the receiver's `turnOn()` method.
- **RemoteControl** is the invoker that triggers the command.

### Use Case:
- When you want to decouple the object that invokes the operation from the one that performs the operation, such as implementing undo/redo functionality or batch processing.

---

### 5. Observer Pattern

### Problem:
The Observer pattern is used when you have a subject (e.g., a

 data model) that needs to notify a number of dependents (observers) whenever its state changes, without knowing who or what those dependents are.

### Working of the pattern:
- The **Subject** maintains a list of observers and provides methods to add/remove observers.
- The **Observer** interface defines an `update()` method that gets called when the subject's state changes.

### Code example:

```java
import java.util.ArrayList;
import java.util.List;

// Subject
interface Subject {
    void addObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}

// Concrete Subject
class WeatherStation implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private float temperature;

    public void setTemperature(float temperature) {
        this.temperature = temperature;
        notifyObservers();
    }

    @Override
    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature);  // Notify each observer
        }
    }
}

// Observer
interface Observer {
    void update(float temperature);
}

// Concrete Observer
class TemperatureDisplay implements Observer {
    @Override
    public void update(float temperature) {
        System.out.println("Temperature updated to: " + temperature);
    }
}

// Client code
public class ObserverExample {
    public static void main(String[] args) {
        WeatherStation station = new WeatherStation();
        TemperatureDisplay display = new TemperatureDisplay();
        
        station.addObserver(display);  // Observer subscribes to updates
        
        station.setTemperature(25.0f);  // Subject notifies observer
        station.setTemperature(30.5f);  // Subject notifies observer again
    }
}
```

### Explanation:
- **Subject** (WeatherStation) maintains a list of observers and notifies them when its state changes.
- **Observer** (TemperatureDisplay) reacts to changes in the subject's state by calling the `update()` method.

### Use Case:
- When you have a one-to-many relationship between objects, where a change in one object should automatically update other dependent objects (e.g., in UI event handling, push notification systems).
