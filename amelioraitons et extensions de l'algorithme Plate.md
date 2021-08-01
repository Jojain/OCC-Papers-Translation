## About : Extensions and improvments of Plate algorithm

The development of FDF, Toyota requirements for G2 continuity of H-prism, as well as the limitations of STYLER Powerfill and Powermorph actions have, for seemingly quite different demands, lead to improve Plate with the possibility to impose linear constraint that are different from the value of it's function derivative.

This "new version" of Plate make accessible spectaculary improvments in PowerFill (this has been tested, only user interface is left to be developped) and presumably same kind of improvments in PowerMorph
Furthermore it will have to be integrated in the Package GeomPlate, to allow enriched public syntaxes as much for "Shell Design" projet than for CAS.CADE clients.

You will find attached the internal specification document corresponding to these improvments and extensions.

Moreover, integration of this development in CAS.CADE is the subject of the study n° S 3816 opened by the Software Development Direction


# Improvments and Extensions of Plate 
The development of FDF, Toyota requirements for G2 continuity of H-prism, as well as the limitations of STYLER Powerfill and Powermorph actions have, for seemingly quite different demands, lead to improve Plate with the possibility to impose linear constraint that are different from the value of it's function derivative.

This document describe extensions brought in this context.
The design of these improvments is as much coming from recent demands than maturation of ideas resulting in more than 2 years of utilisation of Plate in various contexts.

Moreover, strategies of iterative computation, which constitutes a research domain and certain improvments in the case of PlateFE (Plate is based on finite element method, which will be the topic of another document) can, even if it is less obvious (and less mathematically justifiable) be tested with Plate. The improvments in this domain should allow to share concepts and code between the two tools Plate and PlateFE.

Finally, the public level of CAS.CADE API about surface creation by filling and interpolation is the package GeomPlate, developped by the modelisation team.
This document propose an alternative positioning of differents classes that works together in the Plate and GeomPlate package.

Integration of this development in CAS.CADE is the subject of the study n° S 3816 opened by the Software Development Direction

# Documentation and tests plan

## Documentation

Insofar as these developments does not concern any public syntax (Public syntaxes corresponding to the algorithmic kernel extensions specified here will need to be developped later on in GeomPlate package), we do not propose "client" documentation, and this document will be the internal documentation for developpers.

## Tests plan
### Non-regression tests for CAS.CADE API

Success of non-regression tests on corner fillets and on GeomPlate should make improbable regression on older functionality.
These tests grids are precisely :
- CFI005
- CFI012
- CFI013
as well as the test group :
- TOPOLOGY/gplate

### New functionality test

These ones have been tested only with a unit testing level on simple case. Intensive tests on new functionality on "industrial" cases will only come with the integration of these functionality in applications.
So, more likely in these integrations:
- H-Prism functions for TOYOTA
- PowerFill extensions in STYLER
- PowerMorph extensions in STYLER
Scientific Direction will assist in the development in order to finish to tests the new functionalities of Plate and NLPlate.

# Positioning of the differents classes
## Plate Package

Le package Plate (cf. ALR96346.DOC) allow to compute a function define in R² of value in R<sup>3</sup>.
This function minimize a **quadratic** criterion called energy (because it generalize linearized bending energy in a thin plate) by checking a given amount of **linear** Constraints.
The Plate Package only depends on Kernel and CAS.CADE math package.
Not knowing Geom Package, Plate Package doesn't manipulate *Curve* or *Surface* data from *Geom*. Curvilinear constraint or initialisation surfaces are then excluded from *Plate*.

Class *Plate* from same the package which share it's name holds most of the algorithmic.
It's always used given the following sequence :
1. We "load" all constrain (one after another) with the *Load* method
2. We solve the minimisation problem under constrain with the method *SolveTI*
3. We access the solution by positioning methods on the function, *Evaluate(...)* and *EvalutateDerivate(...)*

To understand correctly the positioning and the collaboration between different classes, one has to remember that as for today there is two way of using *Plate* :
1. Computation of a surface sum of a given initial surface and the *Plate* function.
2. Deforming of a topology by applying a space deforming function (from R<sup>3</sup> to R<sup>3</sup>) defined with the help of *Plate* function.

Today, only the component FdF (Powermorph) enters the second use case.
However, the G2 continuity of H-prism for Toyota, Powerfill (filling n-sided of STYLER) as well as fillets computations of CAS.CADE around vertices uses the first method.

*Plate* package proposes :
- *Plate* class (that contains most of the algorithmic)
- classes that represents Constraints, all called *XXXConstraint*

Constraints are decomposed in 3 categories:

## Base Constraints

There is 3 base constraints that are the only ones known by the solver.

- *PinPointConstraint*
- *LinearXYZConstraint*
- *LinearScalarConstraint*

The last two are new and are described in detail in the following chapter.

## "Composed" constraint

"Composed" constraint, which are common to all use cases of *Plate* (1 and 2) corespond to a particular user need.
Their constructor consist in the creation of a set of base constraints which is their equivalent.
The addition of new composed constraints will be necessary and will required only :
- The creation in their constructor of the equivalent base constraint
- Writing of the corresponding *Load* method in the *Plate* class, which only collects base constraints contained in the field of the composed constraint.

We can cite :
- *PlaneConstraint*
- *LineConstraint*
- *SampledCurveConstraint*
- *GlobalTransformationConstraint*
The first two creates a particular *LinearScalarConstraint* and the two last creates a particular *LinearXYZConstraint*
All of these are new.

## Specific "Composed" constraint

They differ from the previous only by the fact that they are specific of the first method of use. They suppose that *Plate* is used to create a *Surface* as the sum of an initial surface and the *Plate* function. As for today it exists the following :
- *GtoCConstraint*
- *FreeGtoCConstraint*

which should be completed by they equivalent of type *SampledCurve*.
These Constraints creates *PinPointConstraint* for the first one and both *PinPointConstraint* and *LinearScalarConstraint* for the second one, which is new.
These base constraints are build from information on the initial surface and on the surface which they wants to establish a contact of order k (k=1,2 or 3)


## *GeomPlate* package

Package *GeomPlate* is meant to present the CAS.CADE public API for functions of surface creation by filling, smoothing and interpolation of punctual or curvilinear constraints based on *Plate* and *PlateFE*.

This package knows the CAS.CADE geometry (in particular *Geom* Package) but not the topology.

The class *BuildPlateSurface* of the package *GeomPlate*, developped by the modelisation team, has for main goal the computation of a surface verifying a set of punctual or curvilinear constraints. For this :
1. an initial surface is computed
2. curvilinear constraint are sampled in punctual constraints
3. *Plate* is called with the utilisation mode 1
4. depending on obtained errors, sampled is refined.

A new class *NLPlate* (cf. chpater IV C), as been added to *GeomPlate* package in order to handle the differents iterative strategies (or incremental loading) based on *Plate*.

For now, this class has only been tested in the context of the Powerfill action of STYLER.
In the future, *BuildPlateSurface* will also have to call *NLPlate* (or, on option *PlateFE*) instead of *Plate*.

An abstract class manipulated by *Handle* of punctual Geometric Constraint (*GPPConstraint*) as been created in order to allow *NLPlate* (and later on *PlateFE*) to handle uniformly the differents punctual constraints, whatever their provenance.
Independently of the fact that the choice of an abstract class allow to switch more easily to an "interface" API, it allows, in the case of a large number of punctual constraints (for the RDS for example) to create a concrete type containing only the minimal necessary volume of data.

Concrete classes deriving from *GPPConstraint* (*G0Constraint*, *G0G1Constraint*, *G0G2Constraint*, ...) have been created in order to easily create the most usual constraints.

Overview of class that are collaborating in packages *Plate* and *GeomPlate*

 ![img](/extracted_imgs/p8.PNG) 






