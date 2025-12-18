# Factories
The problem of creation: applications should be able to create objects without depending on those objects

Consider app using shape interface and circle and square objects implement these

If app needs to know about circle and shape -> violation of dependency inversion principle

## Abstract factory
```
                    ┌─────────────────────────────┐
                    │      ConcreteFactory1       │
                    │─────────────────────────────│
                    │ ...                         │
                    │─────────────────────────────│
                    │ + createProductA(): ProductA│
                    │ + createProductB(): ProductB│
                    └─────────────────────────────┘
                                  │
                                  │ implements
                                  ▼
┌───────────────────────────────────────────────────┐      ┌─────────────────────────────────┐
│              «interface»                          │      │            Client               │
│            AbstractFactory                        │◄─────│─────────────────────────────────│
│───────────────────────────────────────────────────│      │ - factory: AbstractFactory      │
│ + createProductA(): ProductA                      │      │─────────────────────────────────│
│ + createProductB(): ProductB                      │      │ + Client(f: AbstractFactory)    │
└───────────────────────────────────────────────────┘      │ + someOperation()               │
                                  △                        └─────────────────────────────────┘
                                  │ implements                           │
                                  │                                      ▼
                    ┌─────────────────────────────┐      ┌───────────────────────────────────┐
                    │      ConcreteFactory2       │      │ ProductA pa = factory.            │
                    │─────────────────────────────│      │               createProductA()    │
                    │ ...                         │      └───────────────────────────────────┘
                    │─────────────────────────────│
                    │ + createProductA(): ProductA│──────┐
                    │ + createProductB(): ProductB│      │
                    └─────────────────────────────┘      │
                                                         ▼
                                          ┌─────────────────────────────┐
                                          │ return new                  │
                                          │ ConcreteProductA2()         │
                                          └─────────────────────────────┘


        PRODUCT HIERARCHIES
        
┌─────────────────┐   ┌─────────────────┐
│   Concrete      │   │   Concrete      │
│   ProductA1     │   │   ProductB1     │
└────────┬────────┘   └────────┬────────┘
         │                     │
         ▼                     ▼
┌─────────────────┐   ┌─────────────────┐
│    Abstract     │   │    Abstract     │
│    ProductA     │   │    ProductB     │
└────────┬────────┘   └────────┬────────┘
         △                     △
         │                     │
┌────────┴────────┐   ┌────────┴────────┐
│   Concrete      │   │   Concrete      │
│   ProductA2     │   │   ProductB2     │
└─────────────────┘   └─────────────────┘
```
| # | Component | Description |
|---|-----------|-------------|
| 1 | **AbstractProductA / AbstractProductB** | Declares interfaces for each distinct product type |
| 2 | **ConcreteProducts (A1, A2, B1, B2)** | Implements abstract products for each factory variant |
| 3 | **AbstractFactory** | Interface declaring creation methods for each product type |
| 4 | **ConcreteFactory1 / ConcreteFactory2** | Implements creation methods to produce family-specific products |
| 5 | **Client** | Uses factory via interface; factory is injected as dependency |

Introduce new interface: shape factory

Application will use shape factory
Two methods:
- make square
- make circle

Shape factory implementation: makes square and circle

Now shape, app and shape factory know nothing about shape factory impl, square and circle: their changes will not affect them

Only change needed: if you add a new shape, shape factory needs a change. Needs a new method, for example like makeTriangle

How to break this coupling?
- Have only 1 make method that takes argument that specifies type
- Enum is not a good way: change below line makes change above line
- use string: shape factory could have getShapeNames method

Independent deployability: being able to swap out components without breaking system

Unit tests to deal with lack of type safety

## Factory method
```
┌─────────────────────────────────┐
│ Product p = createProduct()     │
│ p.doStuff()                     │
└─────────────────────────────────┘
        │
        ▼
┌───────────────────────────────┐         ┌─────────────────────┐
│          Creator              │         │    «interface»      │
│───────────────────────────────│ ──────▶ │      Product        │
│ ...                           │         │─────────────────────│
│───────────────────────────────│         │ + doStuff()         │
│ + someOperation()             │         └─────────────────────┘
│ + createProduct(): Product    │                       △
└───────────────────────────────┘                       │
              △                               ┌─────────┴─────────┐
              │                               │                   │
    ┌─────────┴─────────┐             ┌────────────────┐ ┌────────────────┐
    │                   │             │   Concrete     │ │   Concrete     │
┌────────────────┐ ┌────────────────┐ │   ProductA     │ │   ProductB     │
│ConcreteCreatorA│ │ConcreteCreatorB│ └────────────────┘ └────────────────┘
│────────────────│ │────────────────│
│ ...            │ │ ...            │
│────────────────│ │────────────────│
│+createProduct()│ │+createProduct()│
│  : Product     │ │  : Product     │
└────────────────┘ └────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ return new ConcreteProductA()│
└──────────────────────────────┘
```

| # | Component | Description |
|---|-----------|-------------|
| 1 | **Product** (interface) | Declares the interface for objects the factory method creates |
| 2 | **ConcreteProduct A/B** | Implements the Product interface |
| 3 | **Creator** | Declares the factory method `createProduct()` that returns a Product |
| 4 | **ConcreteCreator A/B** | Overrides the factory method to return a ConcreteProduct instance |

Logical equivalent of abstract factory, difference only physical

consider an abstract class shape application
- public void runShapeApplication();
- protected abstract Shape makeSquare(Point origin, double side);
- protected abstract Shape makeCircle(Point origin, double radius);

And implementation ShapeApplicationImpl
- protected Shape makeSquare(Point origin, double side);
- protected Shape makeCircle(Point origin, double radius);

## Trade off
Downside factory method:
- you can't change factory at runtime

Downside abstract factory:
- bit more complex
- little bit more CPU time/memory

Most of the time abstract factory is best choice

## Prototype pattern
Logically equivalent to abstract factory but exchanges objects for methods