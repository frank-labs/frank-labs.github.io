---
layout: post
mermaid: true
title:  "Sketchpad, practice UI design and state management"
---

In a Human-Computer Interface class, one of the assignments was to design a sketchpad. At the time, I completed most of the features, though not all. Looking back, it was an interesting task that allowed me to practice a variety of concepts, including object-oriented programming, state management, and exploring how we interact with applications. Recently, I revisited the project, completed all the features, and added some improvements based on my understanding. Here, I will document my thought process.

### Requirements

Let’s begin by listing the key requirements for the sketchpad application:

- Straight lines
- Rectangles and squares
- Ellipses and circles
- Scribbled freehand lines
- Polygons, including open and closed polygons
- Color selection
- Move
- Cut, copy, and paste
- Delete
- Group/Ungroup
- Undo/Redo
- Save/Load

### Programming Language Choice

This project does not impose any restrictions on the programming language, so we are free to choose one that suits us. I have experience with Java, Python, and HTML/CSS/JavaScript. After some consideration, I decided to use Python because of the following reasons:

Python's flexibility and simplicity make it well-suited for rapid development.
Tkinter, Python’s default GUI library, is lightweight and capable enough for the requirements of this project. It is also included by default, which simplifies setup and development.

### Class Design

To ensure the code is well-organized, readable, and maintainable, I opted for an object-oriented programming (OOP) approach. Below is the analysis and final class design:

#### Analysis of Classes

1. **Circles and Squares as Special Cases**:  
   Circles and squares are special cases of ellipses and rectangles, respectively. They share similar properties but are distinguished by additional constraints.

2. **Approach for Drawing Circles**:  
   There are two common approaches for drawing circles:
   - **Start from the center (O)**: The mouse movement determines the radius (R).
   - **Draw like a square**: The circle is drawn inside a square defined by the mouse movement.  
     I prefer the second approach as it provides better control over the size of the circle.

3. **Serialization/Deserialization**:  
   - **Regular Shapes (Straight Lines, Rectangles, Ellipses)**:  
     These shapes can be defined by two points: the starting point and the ending point. (For ellipses, we assume they are drawn inside a rectangle defined by these two points.)
   - **Irregular Shapes (Freehand and Polygons)**:  
     These shapes are defined by a series of points. The shape can be determined by connecting these points together.

4. **Open vs. Closed Polygons**:  
   Do we need an additional property to distinguish between them?  
   - A **closed polygon** is one where the ending point is the same as the first point.  
   - An **open polygon** is one where the ending point is different from the first point.  
     Therefore, we can determine this based on the points alone.  
     Note: Tkinter's `create_polygon` function only supports closed polygons, which can be confusing. For this reason, we avoid using it.

5. **Color Property**:  
   Since we only need the line color (not the fill color), all shapes have a single `color` property.

#### Final Class Design

Based on the above considerations, we define the following class hierarchy for the project:

- **`Shape`** (Base Class):  
  - Property: `color`  
  - This is the overarching class for all shapes.

- **`RegularShape`** (Derived Class for Regular Shapes):  
  - Handles shapes like straight lines, rectangles, and ellipses.  
  - Defined by two points (start and end).

- **`IrregularShape`** (Derived Class for Irregular Shapes):  
  - Handles shapes like freehand drawings and polygons.  
  - Defined by a series of points.

By following this design, the code remains modular, flexible, and easy to extend for future enhancements.



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

### Beheivior Design

Drawing shapes on a canvas is straightforward, as Tinker already has some built-in methods for it. However, there are some deeper considerations that need to be addressed. Below, I will list these key points.

#### 1. Open and Closed Polygons

Tinker Canvas has a built-in method for drawing polygons, but it only supports closed polygons. An open polygon, on the other hand, is essentially a series of connected lines. Fortunately, Tinker Canvas supports this approach. We can use the `drawline` method to draw open polygons.

**Key observation:**  
If the last point is the same as the first point, the polygon becomes a closed one. Therefore, we can use the `drawline` method to handle both closed and open polygons.

##### How to Draw:
To give users a clear view of the current drawing line, we use a preview dotted line. When the user clicks and moves the mouse, this line will be shown to provide guidance.

##### How to Close:
1. If the mouse clicks near the first point, we will assume that the user wants to close the polygon. The system will snap to close the polygon.
2. If the user wants to draw an open polygon, they need to right-click to indicate the last point. This will then close the polygon.
3. If the user switches modes (to other shapes or editing tools) during the drawing, the polygon will remain open, and the last point will be the clicked point before switching modes.

#### 2. Select/Moving Mode

The simplest approach is to have a move button, and when in this mode, the user can drag shapes to move them. However, this isn't how users typically interact with apps.

Based on how we interact with a file explorer, I added the following behavior features:

1. Single-click to select, drag to move.
2. Ctrl + Click to select multiple shapes.
3. If multiple shapes are selected:
   - Clicking on one of them will deselect the others (only the currently clicked shape remains selected). Clicking on a blank space will deselect all shapes.
   - Dragging on one selected shape will move all selected shapes (similar to how file explorers behave).
4. If **Ctrl** is pressed and multiple shapes are selected:
   - Clicking on a shape will toggle its selection status (clicking on a selected shape will deselect it, and clicking on an unselected shape will select it).
   - Dragging one selected shape will move them all (again, similar to how file explorers behave).

This part is the most complex, and as you can see from the code, I used many `if`/`else` statements to decide the flow of actions.

#### 3. How to Distinguish Between a Click and a Drag

For a single mouse button, there are three main events in Tinker Canvas:

- `Button down <Button-1>`
- `Mouse move while button down <B1-Motion>`
- `Button up <ButtonRelease-1>`

To distinguish between a click and a drag, we cannot judge this solely from the `button down` event, since they are all the same at that point. The method is to judge during the `mouse move while button down` event (`<B1-Motion>`). We set a small threshold: if the mouse moves more than the threshold, we consider it a drag. If the movement is less (accounting for shaky hands in a normal click), we consider it a click.

From StackOverflow, I learned that this is the way Windows handles the distinction between a click and a drag.

#### To be continued