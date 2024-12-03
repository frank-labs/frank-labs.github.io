---
layout: post
title:  "Sketchpad, practice UI design and state management"
---

In a Human-Computer Interface class, one of the assignments was to design a sketchpad. At the time, I completed most of the features, though not all. Looking back, it was an interesting task that allowed me to practice a variety of concepts, including object-oriented programming, state management, and exploring how we interact with applications. Recently, I revisited the project, completed all the features, and added some improvements based on my understanding. Here, I will document my thought process.

### Requirements

Let’s begin by listing the key requirements for the sketchpad application:

Drawing Tools:

Straight lines
Rectangles and squares
Ellipses and circles
Scribbled freehand lines
Open and closed polygons
Editing Tools:

Move
Cut, copy, and paste
Delete
Object Management:

Group/Ungroup
State Management:

Undo/Redo
File Operations:

Save/Load

### Programming Language Choice

This project does not impose any restrictions on the programming language, so we are free to choose one that suits us. I have experience with Java, Python, and HTML/CSS/JavaScript. After some consideration, I decided to use Python because of the following reasons:

Python's flexibility and simplicity make it well-suited for rapid development.
Tkinter, Python’s default GUI library, is lightweight and capable enough for the requirements of this project. It is also included by default, which simplifies setup and development.
Design Approach
To ensure the code is well-organized, readable, and maintainable, I opted for an object-oriented programming (OOP) approach. Below is the final class design for the project:

### Design Approach

To ensure the code is well-organized, readable, and maintainable, I opted for an object-oriented programming (OOP) approach. Below is the final class design for the project:


```mermaid
classDiagram
    class Shape {
        - color: str
        + draw(canvas)
        + contains_point(x, y): bool
        + move(dx, dy)
        + to_dict(): dict
        + from_dict(data): Shape
    }

    class IrRegularShape {
        - points: list
        + move(dx, dy)
        + to_dict(): dict
        + from_dict(data): IrRegularShape
        + add_point(x, y)
        + preview(canvas, x, y)
        + flatten_points(points): list
    }

    class RegularShape {
        - start_point: tuple
        - end_point: tuple
        + move(dx, dy)
        + to_dict(): dict
        + from_dict(data): RegularShape
    }

    class Polygon {
        + add_point(x, y)
        + draw(canvas)
        + preview(canvas, x, y)
        + contains_point(x, y): bool
        + flatten_points(points): list
    }

    class Freehand {
        + draw(canvas)
        + contains_point(x, y): bool
        + flatten_points(): list
    }

    class Line {
        + draw(canvas)
        + contains_point(x, y): bool
    }

    class Rectangle {
        + draw(canvas)
        + contains_point(x, y): bool
    }

    class Ellipse {
        + draw(canvas)
        + contains_point(x, y): bool
    }

    class Square {
        + draw(canvas)
    }

    class Circle {
        + draw(canvas)
    }

    class Group {
        - shapes: list
        + draw(canvas)
        + contains_point(x, y): bool
        + move(dx, dy)
        + to_dict(): dict
        + from_dict(data): Group
    }

    Shape <|-- IrRegularShape
    Shape <|-- RegularShape
    IrRegularShape <|-- Polygon
    IrRegularShape <|-- Freehand
    RegularShape <|-- Line
    RegularShape <|-- Rectangle
    RegularShape <|-- Ellipse
    Rectangle <|-- Square
    Ellipse <|-- Circle
    Shape <|-- Group
```