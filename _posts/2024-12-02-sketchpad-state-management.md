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

To ensure the code is well-organized, readable, and maintainable, I opted for an object-oriented programming (OOP) approach. Lets analyse the classes here, 1. circles and squares are a special case for Ellipses and Rectangles. 2. When draing circles there are two approaches here, first is start from the centre or origin O, and the distance mouse moves is the radius R, second is like drawing a square, and put the circle into the square, I like the second approach better, because it allow me to control the size better 3.When considering Serialization/Deserialization, for the  'regular' shapes like Straight lines, Rectangles, Ellipses they can be decided by two point, the starting point and ending point,(assuming when drawing Ellipse, we drawing this Ellipse inside the Rectangle our mouth defined).   for  the IrRegular Shapes(Freehand and Polygon), actually they are a series of points. we can determin the shape jsut by connecting all these points together.  4, as for  open Polygon and closed Polygon, do we need another property to judge? actually a closed polygon is just the ending point is the first point, while for open polygon, the ending point is diffrent from the first point. so we can decide by this. Tkinter have a  create_polygon function, but it can only draw closed polygon, so it is a bit confusing, and we wouldn't use it. 5. as for color, as for we only need line color, not the filling color, all shapes have one color property. So, the final class design is we have an overarching class [Shape] with property [color], a RegularShape class with the  Below is the final class design for the project:


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