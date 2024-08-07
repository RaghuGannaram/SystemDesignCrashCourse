# S.O.L.I.D Principles

SOLID is an acronym for five design principles intended to make software designs more understandable, flexible, and maintainable. These principles were introduced by Robert C. Martin in his 2000 paper Design Principles and Design Patterns.

## Single Responsibility Principle (SRP)

A class should have one, and only one, reason to change. This means that a class should have only one job.

### Example

-   Bad example: A class with multiple responsibilities

    ```java
    public class UserService {
        public void addUser(User user) {
            // Add user to the database
        }

        public void sendWelcomeEmail(User user) {
            // Send a welcome email to the user
        }
    }
    ```

-   Good example: Separate classes for each responsibility

    ```java
    public class UserService {
        public void addUser(User user) {
            // Add user to the database
        }
    }

    public class EmailService {
        public void sendWelcomeEmail(User user) {
            // Send a welcome email to the user
        }
    }
    ```

## Open/Closed Principle (OCP)

Software entities should be open for extension but closed for modification. This means that a class should be easily extendable without modifying the class itself.

### Example

-   Bad example: A class that is not open for extension

    ```java
    public class Rectangle {
        public double width;
        public double height;

        public Rectangle(double width, double height) {
            this.width = width;
            this.height = height;
        }

        public double calculateArea() {
            return this.width * this.height;
        }
    }

    public class AreaCalculator {
        public double calculateRectangleArea(Rectangle rectangle) {
            return rectangle.calculateArea();
        }
    }
    ```

-   Good example: A class that is open for extension

    ```java
    public interface Shape {
        double calculateArea()
    }

    public class Rectangle implements Shape {
        public double height;
        public double width;

        public Rectangle(double height, double width){
            this.height = height;
            this.width = width;
        }

        @Override
        public double calculateArea(){
            return this.height * this.width;
        }
    }

    public class Circle implements Shape {
        public double radius;

        public Circle (double radius){
            this.radius = radius;
        }

        @Override
        public double calculateArea(){
            return Math.PI * this.radius * this.radius;
        }
    }

    public class AreaCalculator {
        public double calculateArea(Shape shape){
            return shape.calculateArea();
        }
    }
    ```

## Liskov Substitution Principle (LSP)

Objects in a program should be replaceable with instances of their subtypes without altering the correctness of the program. This means that a subclass should override the parent class methods in a way that does not break the functionality from a client’s point of view.

### Example

-   Bad example: A class that violates the Liskov Substitution Principle

    ```java
    public class Bird {
        public void fly() {
            // Fly
        }
    }

    public class Ostrich extends Bird {
        @Override
        public void fly() {
            throw new UnsupportedOperationException("Ostrich cannot fly");
        }
    }
    ```

-   Good example: A class that follows the Liskov Substitution Principle

    ```java
    public class Bird {
        // Common bird properties and methods
    }

    public class Ostrich extends Bird {
        // Ostrich specific properties and methods
    }
    ```

## Interface Segregation Principle (ISP)

A client should never be forced to implement an interface that it doesn’t use. This means that a class should not be forced to implement interfaces that it does not use.

### Example

-   Bad example: A class that violates the Interface Segregation Principle

    ```java
    public interface Worker {
        void work();
        void eat();
    }

    public class HumanWorker implements Worker{

        @Override
        public void work(){
            // Work
        }

        @Override
        public void eat(){
            // Eat
        }
    }

    public class RobotWorker implements Worker{

        @Override
        public void work(){
            //Work
        }

        @Override
        public void eat(){
            throw new UnsupportedOperationException("Robots do not eat");
        }
    }
    ```

-   Good example: A class that follows the Interface Segregation Principle

    ```java
    public interface Workable {
        void work();
    }

    public interface Eatble {
        void eat();
    }

    public class HumanWorker implements Workable, Eatble {
        @Override
        public void work(){
            // Work
        }

        @Override
        public void eat(){
            // Eat
        }
    }

    public class RobotWorker implements Workable {
        @Override
        public void work(){
            // Work
        }
    }
    ```

## Dependency Inversion Principle (DIP)

High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.

### Example

-   Bad example: A class that violates the Dependency Inversion Principle

    ```java
    public class LightBulb{
        public void turnOn{
            //Trun on the light bulb
        }

        public void turnOff{
            //Turn off the light bulb
        }
    }

    public class Swith{
        private LightBulb lightBulb;

        public Switch(LightBulb lightBulb){
            this.lightBulb = lightBulb;
        }

        public void operate(){
            this.lightBulb.turnOn();
            this.lightBulb.turnOff();
        }
    }
    ```

-   Good example: A class that follows the Dependency Inversion Principle

    ```java
    public interface Switchable {
        void turnOn();
        void turnOff();
    }

    public class LightBulb implements Switchable{

        @Override
        public void turnOn(){
            //Turn on the light bulb
        }

        @Override
        public vloid turnOff(){
            //Turn off the light bulb
        }
    }

    public class Switch{
        private Switchable device;

        public Switch(Switchable device){
            this.device = device;
        }

        public void operate(){
            device.turnOn();
            device.turnOff();
        }
    }

    ```
