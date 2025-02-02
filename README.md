# reto_06

Este código está organizado en clases que representan diversas formas geométricas, como puntos, líneas, triángulos, rectángulos y cuadrados, y permite calcular propiedades como perímetros, áreas y ángulos. La clase base `Shape` gestiona un conjunto de puntos y calcula el perímetro de la forma, además de verificar si la figura es regular, es decir, si todos sus lados y ángulos son iguales. Cada clase específica, como `Triangle` y `Rectangle`, se encargan de asegurar que sus características sean válidas, como en el caso de un triángulo que debe tener tres puntos o un rectángulo que debe tener ángulos de 90 grados.

En cuanto a las excepciones, el código maneja errores mediante la excepción personalizada `InvalidShapeError`, que se lanza cuando se intentan crear formas con puntos inválidos o al realizar cálculos con configuraciones incorrectas. Se utilizan bloques `try-except` para capturar errores durante los cálculos de ángulos o áreas, y `raise` para lanzar excepciones personalizadas.


``` python
import math

# Custom exception for invalid shapes


class InvalidShapeError(Exception):
    """
    Exception raised for errors in the definition of a geometric shape.
    This helps ensure that only valid shapes are created and operated upon.
    """
    pass

# Class to represent a point


class Point:
    def __init__(self, x: int = 0, y: int = 0) -> None:
        self.x = x
        self.y = y

    def compute_distance(self, point: "Point") -> float:
        """
        Computes the Euclidean distance between two points
        """
        dx = self.x - point.x
        dy = self.y - point.y
        return math.sqrt(dx**2 + dy**2)

# Class to represent a line


class Line:
    def __init__(self, start_point: Point, end_point: Point) -> None:
        if start_point == end_point:
            # Ensures a valid line segment
            raise ValueError("A line must have two distinct points")
        self.start_point = start_point
        self.end_point = end_point
        self.length = start_point.compute_distance(end_point)

# Base class for shapes


class Shape:
    def __init__(self, points: list[Point]) -> None:
        if len(points) < 3:
            # Ensures a valid polygon
            raise InvalidShapeError("A polygon must have at least 3 points")
        self.points = points
        try:
            self.edges = [Line(points[i], points[(i + 1) %
                            len(points)]).length for i in range(len(points))]
        except ValueError as e:
            # Handles invalid shapes where two consecutive points are identical
            raise InvalidShapeError("Invalid shape: " + str(e))

    def compute_perimeter(self) -> float:
        """
        Computes the perimeter of the shape by summing the edge lengths
        """
        return sum(self.edges)

    def compute_angles(self) -> list[float]:
        """
        Computes the internal angles of the shape using the cosine rule
        """
        angles = []
        for i in range(len(self.points)):
            p1 = self.points[i - 1]
            p2 = self.points[i]
            p3 = self.points[(i + 1) % len(self.points)]
            a = p1.compute_distance(p2)
            b = p2.compute_distance(p3)
            c = p1.compute_distance(p3)
            try:
                angle = math.acos((a**2 + b**2 - c**2) / (2 * a * b))
            except ValueError:
                # Handles cases where angle calculation fails due to invalid point placement
                raise InvalidShapeError(
                    "Invalid angles due to incorrect point configuration")
            angles.append(math.degrees(angle))
        return angles

    def is_regular(self) -> bool:
        """
        Checks if the shape is regular (all sides and angles are equal)
        """
        return len(set(self.edges)) == 1 and len(set(self.compute_angles())) == 1

# Class to represent a triangle


class Triangle(Shape):
    def __init__(self, points: list[Point]) -> None:
        if len(points) != 3:
            # Ensures only valid triangles are created
            raise InvalidShapeError("A triangle must have exactly 3 points")
        super().__init__(points)

    def compute_area(self) -> float:
        """
        Computes the area of the triangle using Heron's formula
        """
        a, b, c = self.edges
        s = self.compute_perimeter() / 2
        try:
            # Ensures a valid computation
            return math.sqrt(s * (s - a) * (s - b) * (s - c))
        except ValueError:
            # Handles invalid area computation due to incorrect point configurations
            raise InvalidShapeError(
                "Triangle area computation error: Check points configuration")

# Subclasses of triangles


class Equilateral(Triangle):
    def __init__(self, points: list[Point]) -> None:
        super().__init__(points)


class Isosceles(Triangle):
    def __init__(self, points: list[Point]) -> None:
        super().__init__(points)


class Scalene(Triangle):
    def __init__(self, points: list[Point]) -> None:
        super().__init__(points)


class RightTriangle(Triangle):
    def __init__(self, points: list[Point]) -> None:
        super().__init__(points)

# Function to identify the type of triangle


def define_triangle(points: list[Point]) -> Triangle:
    """
    Determines the type of triangle based on its edge lengths and angles
    """
    triangle = Triangle(points)
    edges = sorted(triangle.edges)
    tolerance = 1e-9

    if all(abs(edge - edges[0]) < tolerance for edge in edges):
        return Equilateral(points)
    elif len(set(edges)) == 2:
        return Isosceles(points)
    elif abs(edges[0]**2 + edges[1]**2 - edges[2]**2) < tolerance:
        return RightTriangle(points)
    else:
        return Scalene(points)

# Class to represent a rectangle


class Rectangle(Shape):
    def __init__(self, points: list[Point]) -> None:
        if len(points) != 4:
            # Ensures only valid rectangles are created
            raise InvalidShapeError("A rectangle must have exactly 4 points")
        super().__init__(points)
        if not self._is_rectangle():
            raise InvalidShapeError("The points do not form a rectangle")

    def _is_rectangle(self) -> bool:
        """
        Checks if the given points form a rectangle by verifying the angles
        """
        angles = self.compute_angles()
        tolerance = 1e-9
        return all(abs(angle - 90) < tolerance for angle in angles)

    def compute_area(self) -> float:
        """
        Computes the area of the rectangle using width * height
        """
        return self.edges[0] * self.edges[1]

# Subclass of the rectangle to represent a square


class Square(Rectangle):
    def __init__(self, points: list[Point]) -> None:
        super().__init__(points)
        if not self.is_regular():
            raise InvalidShapeError(
                "A square must have 4 equal sides and right angles")

```
