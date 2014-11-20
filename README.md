Download and install APSG module
--------------------------------

APSG is distributed as a single file with no traditional python install implemented yet. For now, download `apsg.py` file and save it to working directory or to any folder on `PYTHONPATH`.

First steps with APSG module
============================

APSG defines several new python classes to easily manage, analyze and visualize orientational structural geology data. Base class `Vec3` is derived from `numpy.array` class and affers several new method which will be explained on following examples.

Import APSG module
------------------

APSG module could be imported either into own namespace or into active one for easier interactive work:

~~~~ {.python}
>>> from apsg import *
~~~~

Basic operations with vectors
-----------------------------

Instance of vector object `Vec3` could be created from any iterable object as list, tuple or array:

~~~~ {.python}
>>> u = Vec3([1, -2, 3])
>>> v = Vec3([-2, 1, 1])
~~~~

For common vector operation we can use standard mathematical operators or special methods using dot notation:

~~~~ {.python}
>>> u+v
V(-1.000, -1.000, 4.000)
>>> u-v
V(3.000, -3.000, 2.000)
>>> 3*u-2*v
V(7.000, -8.000, 7.000)
~~~~

Its magnitude or length is most commonly defined as its Euclidean norm and could be calculated using `abs`:

~~~~ {.python}
>>> abs(v)
2.4494897427831779
>>> abs(u)
3.7416573867739413
>>> abs(2*u)
7.4833147735478827
~~~~

For *dot product* we can use multiplification operator `*` or `dot` method:

~~~~ {.python}
>>> u*v
-1
>>> u.dot(v)
-1
~~~~

For *cross product* we can use operator `**` or method `cross`:

~~~~ {.python}
>>> u**v
V(-5.000, -7.000, -3.000)
>>> u.cross(v)
V(-5.000, -7.000, -3.000)
~~~~

To project vector `u` onto vector `v` we can use method `proj`:

~~~~ {.python}
>>> u.proj(v)
V(0.333, -0.167, -0.167)
~~~~

To find angle (in degrees) between to vectors we use method `angle`:

~~~~ {.python}
>>> u.angle(v)
96.263952719927218
~~~~

Method `rotate` provide possibility to rotate vector around another vector. For example, to rotate vector `u` around vector `v` for 45°:

~~~~ {.python}
>>> u.rotate(v,45)
V(2.248, 0.558, 2.939)
~~~~

Classes Lin, Fol and Pole
=========================

To work with orientational data in structural geology, APSG provide two classes derived from `Vec3` class. There is `Fol` class to represent planar features by planes and `Lin` class to represent linear feature by lines. Both classes provide all `Vec3` methods, but they differ in way how instance is created and how some operations are calculated, as structural geology data are commonly axial in nature. The special class `Pole` is derived from `Fol` and only differs in way how it is visualized on stereographic projection.

To create instance of `Lin`, `Fol` or `Pole` class, we have to provide dip direction and dip, both in degrees:

~~~~ {.python}
>>> Lin(120,60)
L:120/60
>>> Fol(216,62)
S:216/62
>>> Pole(216,62)
P:216/62
~~~~

or we can create instance from `Vec3` object:

~~~~ {.python}
>>> u.aslin
L:297/53
>>> u.asfol
S:117/37
>>> u.aspole
P:117/37
~~~~

Vec3 methods for Lin, Fol and Pole
----------------------------------

To find angle between two linear or planar features:

~~~~ {.python}
>>> l1 = Lin(110,40)
>>> l2 = Lin(160,30)
>>> l1.angle(l2)
41.597412680035468
>>> p1 = Fol(330,50)
>>> p2 = Fol(250,40)
>>> p1.angle(p2)
54.696399321975328
~~~~

To construct planar feature defined by two linear features:

~~~~ {.python}
>>> l1**l2
S:113/40
~~~~

To construct linear feature defined as intersection of two planar features:

~~~~ {.python}
>>> p1**p2
L:278/36
~~~~

**Cross product** of planar and linear features could be used to:

-   construct plane defined by linear feature and normal of planar feature:

~~~~ {.python}
>>> l2**p2
S:96/53
~~~~

-   or to find perpendicular linear feature on given plane.

~~~~ {.python}
>>> p2**l2
L:276/37
~~~~

To rotate structural features we can use method `rotate`:

~~~~ {.python}
>>> p2.rotate(l2,45)
S:269/78
~~~~

Group class
===========

`Group` class serve as a homogeneous container for `Lin`, `Fol` or `Pole` objects. It allows grouping of features either for visualization or batch analysis.

~~~~ {.python}
>>> d = Group([Lin(120,60), Lin(116,50), Lin(132,45),
...            Lin(90,60), Lin(84,52)], name='L1')
... 
>>> print(d)
L1: 5 Lin
~~~~

Method `len` returns number of features in group:

~~~~ {.python}
>>> len(d)
5
~~~~

Property `resultant` gives mean or resultant of all features in group:

~~~~ {.python}
>>> d.resultant
L:110/55
~~~~

To measure angles between all features in group and another feature, we can use method `angle`:

~~~~ {.python}
>>> d.angle(d.resultant)
array([  7.60329482,   6.24648167,  17.37186861,  11.6536752 ,  15.3996262 
])
~~~~

To rotate all features in group around another feature, we can use method `rotate`:

~~~~ {.python}
>>> d.rotate(Lin(150, 30), 45)
L1: 5 Lin
~~~~

To calculate orientation tensor of all features in group, we can use method `ortensor`:

~~~~ {.python}
>>> d.ortensor
Ortensor:
(E1:4.77,E2:0.2011,E3:0.02874)
[[ 0.36990905 -0.48027385 -0.71621555]
 [-0.48027385  1.42230591  2.10464496]
 [-0.71621555  2.10464496  3.20778504]]
~~~~

Ortensor class
==============

`Ortensor` class represents orientation tensor of set of planar or linear features. Eigenvalues and eigenvectors could be obtained by methods `eigenvals` and `eigenvects`. Eigenvectors could be also represented by linear or planar features using properties eigenlins and eigenfols.

~~~~ {.python}
>>> ot = Ortensor(d)
>>> ot.eigenvals
(0.95403846865963882, 0.040212749461964618, 0.0057487818783964056)
>>> ot.eigenvects
(V(0.192, -0.542, -0.818), V(-0.981, -0.082, -0.176), V(-0.028, -0.836, 0.5
47))
>>> ot.eigenlins
(E1: 1 Lin, E2: 1 Lin, E3: 1 Lin)
>>> ot.eigenfols
(E1: 1 Fol, E2: 1 Fol, E3: 1 Fol)
~~~~

Density class
=============

`Density` class represents Gaussian point density distribution of features from dataset. Parameters of calculation could be defined by parameter `k` and by amount of counting points `npoints`. Number of countours and color map could be modified by `nc` and `cm` properties.

~~~~ {.python}
>>> c = Density(d, npoints=90)
>>> c
Density grid from 5 data with 6 contours.
Gridded on 90 points.
Values: k=100 E=0.05 s=0.1565
Max. weight: 1.422
>>> c.plotcountgrid()
~~~~

![](figures/apsg_figure25_1.png)

Schmidt projection
==================

Any `Fol`, `Lin`, `Pole`, `Vec3` or `Group` object could be visualized in Schmidt projection:

~~~~ {.python}
>>> SchmidtNet(Fol(214,55), Lin(120,60), Pole(240,60), Vec3([-1,-2,1]))
<apsg.SchmidtNet object at 0x7f071f0bdf50>
~~~~

![](figures/apsg_figure26_1.png)

Features could be added to Schmidt projection programatically as well:

~~~~ {.python}
>>> s = SchmidtNet()
>>> s.add(Fol(150,40))
>>> s.add(Pole(150,40))
>>> s.add(Lin(112,30))
>>> s.show()
~~~~

![](figures/apsg_figure27_2.png)

`Dataset` properties as color and name are used during visualization:

~~~~ {.python}
>>> s.clear()
>>> d = Group([Lin(120,60), Lin(116,50), Lin(132,45), Lin(95,52)], name='Te
st')
>>> s.add(d)
>>> s.add(d.ortensor)
>>> s.show()
~~~~

![](figures/apsg_figure28_2.png)

All mentioned classes could be freely combined:

~~~~ {.python}
>>> s.clear()
>>> d = Group([Lin(120,70), Lin(116,42), Lin(132,45),
...              Lin(95,52), Lin(114,48), Lin(118,58) ],
...              name='G1', color='red')
... 
>>> s.add(d)
>>> s.add(d.resultant)
>>> s.add(*d.ortensor.eigenfols)
>>> c = Density(d, nc=8)
>>> s.add(c)
>>> s.show()
~~~~

![](figures/apsg_figure29_2.png)

Some tricks
-----------

Double cross product is allowed:

~~~~ {.python}
>>> s.clear()
>>> p = Fol(250,40)
>>> l = Lin(160,30)
>>> s.add(p,l)
>>> s.add(l**p,p**l)
>>> s.add(l**p**l,p**l**p)
>>> s.show()
~~~~

![](figures/apsg_figure30_2.png)

Correct measurements of planar linear pairs:

~~~~ {.python}
>>> p1, l1 = fixpair(p,l)
>>> s.clear()
>>> s.add(p,l)
>>> s.add(p1,l1)
>>> s.show()
~~~~

![](figures/apsg_figure31_2.png)

