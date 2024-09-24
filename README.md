# Classes for computational geometry

Some classes and utility functions for general computational geometry



## Introduction

This article presents two classes and a set of utility functions for computational geometry. ` C3Point` is a 3D counterpart to ` CPoint` and ` CPolygon` encapsulates a set of `C3Point`'s and provides general polygon handling functions. The classes have been mildly optimised for speed. The classes were originally written for use in discretising 2D surfaces into element networks and for calculating properties of the resultant elements. Care must be taken when using some of the functions such as the curvature and area functions to ensure that the results returned by the functions are consistent with your needs and definitions.

The classes make use of a ` typedef` ` REAL` that is either double or float depending on whether ` USING_DOUBLE` or ` USING_FLOAT` has been defined in geometry.h. Obviously using template classes would have been neater, but these classes were developed to get a job done, not to be the epitome of structured programming. A number of conversion functions have been provided:

```cpp
D2Real(x) (x)             // double => REAL
F2Real(x) (x)             // float => REAL
Real2D(x) (x)             // REAL => double
Real2F(x) ((float)(x))    // REAL => float
Int2Real(x) ((double)(x)) // int => REAL
Real2Int(x) ((int)(x))    // REAL => int
Real2Int(double d0)       // REAL => int (faster than a (cast)).
```

All the classes and utility functions are provided 'as-is'. I've been meaning to write this class up for a long time and figured it was best to at least post *something* than nothing at all.

## C3Point

`C3Point` is a 3D counterpart to `CPoint`. It contains 3 data members x,y and z and a set of functions for calculating properties, scaling, translating and for arithmetic operations.

```cpp
class C3Point {
// Attributes
public:
    REAL x,y,z;

//Operations
public:
    C3Point() {}                          // constructor
    C3Point(double x, double y, double z) // constructor
    REAL Length2()                        // length squared
    REAL Length()                         // length
    void Scale(REAL factor)               // scale by a factor
    void Normalise();                     // convert to a unit length
    void operator=(C3Point& P)            // assign
    C3Point operator-(C3Point P)          // subtract
    C3Point operator-()                   // unary -
    C3Point operator+(C3Point P)          // add
    C3Point operator+=(C3Point P)         // add +=
    C3Point operator-=(C3Point P)         // subtract -=
    REAL operator*(C3Point P)             // vector dot product
    C3Point operator*(REAL f)             // scalar product
    C3Point operator/(REAL f)             // scalar div
    C3Point operator*=(REAL f)            // scalar mult *=
    C3Point operator/=(REAL f)            // scalar div /=
    C3Point operator^(C3Point P)          // cross product
    BOOL operator==(C3Point& P);          // is equal to?
    BOOL operator!=(C3Point& P)           // is not equal to?
};

#define VECTOR C3Point
```

## CPolygon

`CPolygon` encapsulates a set of `C3Point`'s and provides general polygon handling functions.

```cpp
CPolygon();
CPolygon(int);                     // Construct with a preallocated number of points

BOOL Closed();                     // Is the polygon closed?
int GetSize()                      // Number of points

// is vertex 'index' between start,end inclusive?
BOOL InSpan(int start, int end, int index);       
// is vertex 'index' between start,end exclusive?
BOOL InSpanProper(int start, int end, int index); 

BOOL PointIn(C3Point P);           // Is point inside polygon?
BOOL SetSize(int);                 // Change size of polygon
void RemoveAll()                   // Empty polygon of all points
BOOL Trim(int, int);               // Trims polygon down so that points before 
                                   //   "Start" and after "End" are removed.    
                                   //   Start and End must be in the range 0..GetSize()-1
BOOL Close();                      // Make polygon closed
BOOL Add(C3Point);                 // Add point to polygon
BOOL SetAt(int nPos, C3Point& p);  // set vertex nPos as point p
void Delete(int);                  // Delete a vertex
BOOL InsertAt(int nPosition, C3Point P); // insert point P at pos nPosition (0..N-1)

void FreeExtra();                  // Free extra memory left over after deletes

int PointSeparation(int Point1, int Point2); // Distance (in pts) between 2 points
void Rationalise(int nAngle);      // Combines adjacent line segments if the angle between 
                                   //   them is less than nAngle (degrees).
REAL SegmentLength(int,int);       // Length of a segment of the polygon
C3Point GetClosestPoint(C3Point p, int *nIndex = NULL);
REAL Area();                       // returns polygon area
C3Point Centroid();                // Calculate centroid of polygon
BOOL GetAttributes(REAL *pArea, 
                   C3Point *pCentroid, 
                   C3Point *pNorm,
                   REAL *pSlope, 
                   REAL *pAspect);
BOOL Diagonal(int i, int j);             // Returns TRUE iff (v_i, v_j) is a proper 
                                         //   internal or external diagonal of this polygon
virtual void Translate(VECTOR v);        // Translate polygon
BOOL Intersected(C3Point& p1, C3Point& p2);     // Does p1-p2 intersect polygon?
BOOL IntersectedProp(C3Point& p1, C3Point& p2); // Does p1-p2 intersect polygon properly?
BOOL Triangulate(CPolygon*);            // Triangulate: Ear clip triangulation
BOOL CPTriangulate(CPolygon*, C3Point); // Central point triangulation
BOOL DelauneyTri(CPolygon*);            // Triangulate: Delauney triangulation
```

```cpp
// Load polygon from X-Y or X-Y-Z data file
BOOL LoadXY(LPCTSTR Filename, REAL Zdefault = D2Real(0.0));
BOOL LoadXY(FILE* fp, REAL Zdefault = D2Real(0.0));
BOOL LoadXYZ(LPCTSTR Filename, BOOL bWarn = TRUE);
BOOL LoadXYZ(FILE* fp);

// Save file either as:
//    Num Points, elevation, x-y pairs...,
// or
//    x-y-z triplets...
BOOL Save(LPCTSTR Filename, BOOL bAsPoints = FALSE, BOOL bWarn = TRUE);

void NaturalSpline(double*& b, double*& c, double*& d); // Natural cubic spline
REAL Curvature(int i);                                  // Curvature at vertex i
REAL Curvature(int nIndex, int nSampleSize);            // Avg curvature at i over 
                                                        // a number of points

C3Point& operator[](int index);
C3Point& Point(int index);
void operator=(CPolygon& P);
```

## General Functions

These functions provide general routines for vectors (`C3Point`s) and polygons.

```cpp
inline REAL Dot(C3Point V1, C3Point V2)       // dot product
inline C3Point Cross(C3Point p1, C3Point p2)  // cross product
```

```cpp
C3Point GetClosestPoint2D(C3Point& start, C3Point& end, C3Point& P);
REAL   Angle(C3Point, C3Point, C3Point);    // Angle between 2 vectors formed from 3 points (deg)
REAL   Angle(VECTOR v, VECTOR u);           // Angle between 2 vectors (degrees)
REAL   TriArea2(C3Point, C3Point, C3Point); // Area^2 between 2 vectors formed from 3 points 
REAL   TriArea2(VECTOR u, VECTOR v);        // Area^2 between 2 vectors
BOOL   IntersectProp(C3Point a, C3Point b,  // Returns true iff ab properly intersects cd: 
                     C3Point c, C3Point d)  //    they share a point interior to both segments.
                                            //    The properness of the intersection is ensured 
                                            //    by using strict leftness.

BOOL   Intersect(C3Point a, C3Point b,             // Returns true iff segments ab and cd
                 C3Point c, C3Point d);            // intersect, properly or improperly.
BOOL   Left(C3Point a, C3Point b, C3Point c);      // Returns true iff c is strictly to the left 
                                                   //   of the directed line through a to b.
BOOL   LeftOn(C3Point a, C3Point b, C3Point c);    // Same as Left, but c may be on the line ab.
BOOL   Colinear(C3Point a, C3Point b, C3Point c);  // Returns TRUE if a,b,c are colinear
BOOL   Between(C3Point a, C3Point b, C3Point c);   // Returns TRUE iff (a,b,c) are collinear and 
                                                   //   point c lies on the closed segement ab.
VECTOR Normal(C3Point p1, C3Point p2, C3Point p3); // Computes the normal (NOT unit normal) of 
                                                   //   a triangle, with points in Counter 
                                                   //   Clockwise direction.
VECTOR Scale(REAL factor, VECTOR v);               // Scales a vector by a factor.
```

## Credits

The algorithms used are based in part from the book [Computational Geometry in C](http://www.amazon.com/exec/obidos/ASIN/0521649765/thecodeprojec-20/) by Joseph O'Rourke.
