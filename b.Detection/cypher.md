## Factory Method

$Factory Method=TMM\cup TM\cup TT + TMT \times 2 +TT$

```cypher
MATCH
	TMM_TM_TT=(ConcreteCreator:Type)-[:has_method]->(factoryMethodImpl:Method)-[:overrides|implements*..]->(factoryMethod:Method)<-[:has_method]-(Creator:Type)<-[:inherits*..]-(ConcreteCreator:Type),
    MT1=(factoryMethodImpl:Method)-[:actual_return]->(ConcreteProduct:Type),
    MT2=(factoryMethod:Method)-[:return]->(Product:Type),
    TT=(ConcreteProduct:Type)-[:inherits*..]->(Product:Type)
RETURN DISTINCT
	Creator.qualifiedName as Creator,
	ConcreteCreator.qualifiedName as ConcreteCreator,
    Product.qualifiedName as Product,
    ConcreteProduct.qualifiedName as ConcreteProduct
ORDER BY
	Creator.qualifiedName,ConcreteCreator.qualifiedName,Product.qualifiedName
```

## Abstract Factory

$Abstract Factory=(TMM\cup TM\cup TT)^2 + TMT \times 3 +TT^2$

```cypher
MATCH
	TMM_TM_TT1=(ConcreteFactory1:Type)-[:has_method]->(createProduct1Impl:Method)-[:implements*..]->(createProduct:Method)<-[:has_method]-(AbstractFactory:Type)<-[:inherits*..]-(ConcreteFactory1:Type),
    TMM_TT2=(AbstractFactory:Type)<-[:inherits*..]-(ConcreteFactory2:Type)-[:has_method]->(createProduct2Impl:Method)-[:implements*..]->(createProduct:Method),
    MT1=(createProduct1Impl:Method)-[:actual_return]->(ConcreteProduct1:Type),
    MT2=(createProduct2Impl:Method)-[:actual_return]->(ConcreteProduct2:Type),
    MT=(createProduct:Method)-[:return]->(AbstractProduct:Type),
    TT_2=(ConcreteProduct1:Type)-[:inherits*..]->(AbstractProduct:Type)<-[:inherits*..]-(ConcreteProduct2:Type)
RETURN DISTINCT
	AbstractFactory.qualifiedName as AbstractFactory,
	ConcreteFactory1.qualifiedName as ConcreteFactory1,
	ConcreteFactory2.qualifiedName as ConcreteFactory2,
	AbstractProduct.qualifiedName as AbstractProduct,
	ConcreteProduct1.qualifiedName as ConcreteProduct1,
	ConcreteProduct2.qualifiedName as ConcreteProduct2,
	createProduct.signature as createProduct
```



## Builder

$Builder=TMI_mF\cup TMI_mM+TMM\cup TM\cup TT+TMT+FT$

```cypher
MATCH
	TMI_mF=(Director:Type)-[:has_method]->(construct:Method)-[:has_invoke]->(methodCall:MethodCall)-[:caller]->(builder:Field),
	I_mM=(methodCall:MethodCall)-[:target]->(buildPart:Method),
	TMM_TM_TT=(ConcreteBuilder:Type)-[:has_method]->(buildPartImpl:Method)-[:overrides|implements*..]->(buildPart:Method)<-[:has_method]-(Builder:Type)<-[:inherits*..]-(ConcreteBuilder:Type),
	FT=(builder:Field)-[:associates]->(Builder:Type),
	TMT2=(ConcreteBuilder:Type)-[:has_method]->(getResult:Method)-[:return]->(Product:Type)
RETURN DISTINCT
	Director.qualifiedName as Director,
	Builder.qualifiedName as Builder
UNION MATCH	// 调用点扩展
	TMI_mF=(Director:Type)-[:has_method]->(construct:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:caller]->(builder:Field),
	I_mM=(methodCall1:MethodCall)-[:target]->(buildPart:Method),
	TMM_TM_TT=(ConcreteBuilder:Type)-[:has_method]->(buildPartImpl:Method)-[:overrides|implements*..]->(buildPart:Method)<-[:has_method]-(Builder:Type)<-[:inherits*..]-(ConcreteBuilder:Type),
	FT=(builder:Field)-[:associates]->(Builder:Type),
	TMT2=(ConcreteBuilder:Type)-[:has_method]->(getResult:Method)-[:return]->(Product:Type)
RETURN DISTINCT
	Director.qualifiedName as Director,
	Builder.qualifiedName as Builder
UNION MATCH	// 委托获取builder
	TMI_mM=(Director:Type)-[:has_method]->(construct:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(getBuilder:Method),
	I_mM=(methodCall:MethodCall)-[:target]->(buildPart:Method),
	TMM_TM_TT=(ConcreteBuilder:Type)-[:has_method]->(buildPartImpl:Method)-[:overrides|implements*..]->(buildPart:Method)<-[:has_method]-(Builder:Type)<-[:inherits*..]-(ConcreteBuilder:Type),
	MT=(getBuilder:Method)-[:actual_return]->(ConcreteBuilder:Type),
	TMT2=(ConcreteBuilder:Type)-[:has_method]->(getResult:Method)-[:return]->(Product:Type)
RETURN DISTINCT
	Director.qualifiedName as Director,
	Builder.qualifiedName as Builder
```



## Prototype

$Prototype=TMI_mF\cup TMI_mM+TMM\cup TM\cup TT+TMT\times 2+FT$

```cypher
MATCH
	TMI_mF=(Client:Type)-[:has_method]->(operation:Method)-[:has_invoke]->(methodCall:MethodCall)-[:caller]->(prototype:Field),
	I_mM=(methodCall:MethodCall)-[:target]->(clone:Method),
	TMM_TM_TT=(ConcretePrototype:Type)-[:has_method]->(cloneImpl:Method)-[:implements*..]->(clone:Method)<-[:has_method]-(Prototype:Type)<-[:inherits*..]-(ConcretePrototype:Type),
	FT=(prototype:Field)-[:associates]->(Prototype:Type),
	MT1=(clone:Method)-[:return]->(Prototype:Type),
	MT2=(cloneImpl:Method)-[:actual_return]->(ConcretePrototype:Type)
RETURN DISTINCT
	Prototype.qualifiedName as Prototype,
	Client.qualifiedName as Client
UNION MATCH
	TMI_mF=(Client:Type)-[:has_method]->(operation:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:caller]->(prototype:Field),
	I_mM=(methodCall:MethodCall)-[:target]->(clone:Method),
	TMM_TM_TT=(ConcretePrototype:Type)-[:has_method]->(cloneImpl:Method)-[:implements*..]->(clone:Method)<-[:has_method]-(Prototype:Type)<-[:inherits*..]-(ConcretePrototype:Type),
	FT=(prototype:Field)-[:associates]->(Prototype:Type),
	MT1=(clone:Method)-[:return]->(Prototype:Type),
	MT2=(cloneImpl:Method)-[:actual_return]->(ConcretePrototype:Type)
RETURN DISTINCT
	Prototype.qualifiedName as Prototype,
	Client.qualifiedName as Client
UNION MATCH
	TMI_mF=(Client:Type)-[:has_method]->(operation:Method)-[:has_invoke]->(methodCall:MethodCall)-[:caller]->(prototype:Field),
	I_mM=(methodCall:MethodCall)-[:target]->(clone:Method),
	FT=(prototype:Field)-[:associates]->(Prototype:Type)
WHERE clone.signature= "clone()"
RETURN DISTINCT
	Prototype.qualifiedName as Prototype,
	Client.qualifiedName as Client
UNION MATCH
	TMI_mF=(Client:Type)-[:has_method]->(operation:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:caller]->(prototype:Field),
	I_mM=(methodCall1:MethodCall)-[:target]->(clone:Method),
	FT=(prototype:Field)-[:associates]->(Prototype:Type)
WHERE clone.signature= "clone()"
RETURN DISTINCT
	Prototype.qualifiedName as Prototype,
	Client.qualifiedName as Client
```



## Singleton

$Singleton=TFT+TMT+TC$

```cypher
MATCH
	TFT=(Singleton:Type)-[:has_field]->(instance:Field)-[:associates]->(Singleton:Type),
	TMT=(Singleton:Type)-[:has_method]->(getInstance:Method)-[:return]->(Singleton:Type),
	TC=(Singleton:Type)-[:has_constructor]->(constructor:Constructor)
WHERE
	getInstance.modifiers CONTAINS 'static'
AND 
	instance.modifiers CONTAINS 'static'
AND
	constructor.modifiers CONTAINS 'private'
RETURN DISTINCT
	Singleton.qualifiedName as Singleton,
	instance.name as instance,
	getInstance.signature as getInstance
```



## Class Adapter

$Adapter(class)=TMI_mM+TMM\cup TM\cup TT+TT\cup TM$

```cypher
MATCH
	TMM_TM_TT=(Adapter:Type)-[:has_method]->(requestImpl:Method)-[:overrides|implements*..]->(request:Method)<-[:has_method]-(Target:Type)<-[:inherits*..]-(Adapter:Type),
	TTM=(Adapter:Type)-[:inherits*..]->(Adaptee:Type)-[:has_method]->(specificRequest:Method),
	MI_mM=(requestImpl:Method)-[:has_invoke]->(methodCall)-[:target]->(specificRequest:Method)
WHERE NOT EXISTS{MATCH (Adapter:Type)-[:inherits*..]->(Adaptee:Type)}
AND NOT EXISTS{MATCH (Adapter:Type)<-[:inherits*..]-(Adaptee:Type)}
RETURN DISTINCT
	Adapter.qualifiedName as Adapter,
	Target.qualifiedName as Target,
	Adaptee.qualifiedName as Adaptee,
	request.signature as request,
	specificRequest.signature as specificRequest
UNION MATCH
	TMM_TM_TT=(Adapter:Type)-[:has_method]->(requestImpl:Method)-[:overrides|implements*..]->(request:Method)<-[:has_method]-(Target:Type)<-[:inherits*..]-(Adapter:Type),
	TTM=(Adapter:Type)-[:inherits*..]->(Adaptee:Type)-[:has_method]->(specificRequest:Method),
	MI_mM=(requestImpl:Method)-[:has_invoke]->(methodcall:MethodCall)-[:has_anonymous_argument|next*..]->(methodcall1:MethodCall)-[:target]->(specificRequest:Method)
WHERE NOT EXISTS{MATCH (Adapter:Type)-[:inherits*..]->(Adaptee:Type)}
AND NOT EXISTS{MATCH (Adapter:Type)<-[:inherits*..]-(Adaptee:Type)}
RETURN DISTINCT
	Adapter.qualifiedName as Adapter,
	Target.qualifiedName as Target,
	Adaptee.qualifiedName as Adaptee,
	request.signature as request,
	specificRequest.signature as specificRequest
```



## Object Adapter

$Adapter(object)=TMM\cup TM\cup TT +TFT+TM$

```cypher
MATCH
	TMM_TM_TT=(Adapter:Type)-[:has_method]->(requestImpl:Method)-[:overrides|implements*..]->(request:Method)<-[:has_method]-(Target:Type)<-[:inherits*..]-(Adapter:Type),
	TFT=(Adapter:Type)-[:has_field]->(adaptee:Field)-[:associates]->(Adaptee:Type),
	TM=(Adaptee:Type)-[:has_method]->(specificRequest:Method),
	MI_mM=(requestImpl:Method)-[:has_invoke]->(methodcall:MethodCall)-[:target]->(specificRequest:Method),
	I_mF=(methodcall:MethodCall)-[:caller]->(adaptee:Field)
WHERE NOT EXISTS{MATCH (Adapter:Type)-[:inherits*..]->(Adaptee:Type)}
AND NOT EXISTS{MATCH (Adapter:Type)<-[:inherits*..]-(Adaptee:Type)}
AND adaptee.isContainer='true'
RETURN DISTINCT
	Adapter.qualifiedName as Adapter,
	Adaptee.qualifiedName as Adaptee
UNION MATCH
	TMM_TM_TT=(Adapter:Type)-[:has_method]->(requestImpl:Method)-[:overrides|implements*..]->(request:Method)<-[:has_method]-(Target:Type)<-[:inherits*..]-(Adapter:Type),
	TFT=(Adapter:Type)-[:has_field]->(adaptee:Field)-[:associates]->(Adaptee:Type),
	TM=(Adaptee:Type)-[:has_method]->(specificRequest:Method),
	MI_mM=(requestImpl:Method)-[:has_invoke]->(methodcall:MethodCall)-[:next|has_anonymous_argument*..]->(methodcall1:MethodCall)-[:target]->(specificRequest:Method),
	I_mF=(methodcall1:MethodCall)-[:caller]->(adaptee:Field)
WHERE NOT EXISTS{MATCH (Adapter:Type)-[:inherits*..]->(Adaptee:Type)}
AND NOT EXISTS{MATCH (Adapter:Type)<-[:inherits*..]-(Adaptee:Type)}
AND adaptee.isContainer='true'
RETURN DISTINCT
	Adapter.qualifiedName as Adapter,
	Adaptee.qualifiedName as Adaptee
```



## Bridge

$Bridge=TMI_mF\cup TMI_mM+TMM\cup TM\cup TT+TFT+TT$

```cypher
MATCH
	TMI_mM=(Abstraction:Type)-[:has_method]->(operation:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(operationImpl:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(imp:Field),
	TMM_TM_TT=(ConcreteImplementor:Type)-[:has_method]->(operationImplImpl:Method)-[:overrides|implements*..]->(operationImpl:Method)<-[:has_method]-(Implementor:Type)<-[:inherits*..]-(ConcreteImplementor:Type),
	FT=(imp:Field)-[:associates]->(Implementor:Type)
RETURN DISTINCT
	Abstraction.qualifiedName as Abstraction,
	Implementor.qualifiedName as Implementor,
	ConcreteImplementor.qualifiedName as ConcreteImplementor,
	imp.name as imp,
    operation.signature as operation,
    operationImpl.signature as operationImpl
UNION MATCH
	TMI_mM=(Abstraction:Type)-[:has_method]->(operation:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(operationImpl:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(imp:Field),
	TMM_TM_TT=(ConcreteImplementor:Type)-[:has_method]->(operationImplImpl:Method)-[:overrides|implements*..]->(operationImpl:Method)<-[:has_method]-(Implementor:Type)<-[:inherits*..]-(ConcreteImplementor:Type),
	FT=(imp:Field)-[:associates]->(Implementor:Type)
RETURN DISTINCT
	Abstraction.qualifiedName as Abstraction,
	Implementor.qualifiedName as Implementor,
	ConcreteImplementor.qualifiedName as ConcreteImplementor,
    imp.name as imp,
    operation.signature as operation,
    operationImpl.signature as operationImpl
```



## Composite

$Composite=TMI_mF\cup TMI_mM+(TMM\cup TM\cup TT)^2+TFT$

```cypher
MATCH
	TMI_mM=(Composite:Type)-[:has_method]->(composite_operation:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(operation:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(children:Field),
	MM_TM_TT1=(composite_operation:Method)-[:overrides|implements*..]->(operation:Method)<-[:has_method]-(Component:Type)<-[:inherits*..]-(Composite:Type),
	TMM_TT2=(Component:Type)<-[:inherits*..]-(Leaf:Type)-[:has_method]->(leaf_operation:Method)-[:overrides|implements*..]->(operation:Method),
	TFT=(Composite:Type)-[:has_field]->(children:Field)-[:associates]->(Component:Type)                   
WHERE
	children.isContainer='true'
RETURN DISTINCT
	Component.qualifiedName as Component,
	Composite.qualifiedName as Composite,
    Leaf.qualifiedName as Leaf,
    children.name as children,
    operation.signature as operation
UNION MATCH
	TMI_mM=(Composite:Type)-[:has_method]->(composite_operation:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(operation:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(children:Field),
	MM_TM_TT1=(composite_operation:Method)-[:overrides|implements*..]->(operation:Method)<-[:has_method]-(Component:Type)<-[:inherits*..]-(Composite:Type),
	TMM_TT2=(Component:Type)<-[:inherits*..]-(Leaf:Type)-[:has_method]->(leaf_operation:Method)-[:overrides|implements*..]->(operation:Method),
	TFT=(Composite:Type)-[:has_field]->(children:Field)-[:associates]->(Component:Type)                   
WHERE
	children.isContainer='true'
RETURN DISTINCT
	Component.qualifiedName as Component,
	Composite.qualifiedName as Composite,
    Leaf.qualifiedName as Leaf,
    children.name as children,
    operation.signature as operation
```



## Decorator

$Decorator=TMI_mF\cup  TMI_mM+TMI_mM+(TMM\cup TM\cup TT)^2\cup TT+FT$

```cypher
MATCH
	TMI_mM=(Decorator:Type)-[:has_method]->(decorator_operation:Method)-[:has_invoke]->(methodCall1:MethodCall)-[:target]->(operation:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(component:Field),
	MM_TM_TT=(decorator_operation:Method)-[:overrides|implements*..]->(operation:Method)<-[:has_method]-(Component:Type)<-[:inherits*..]-(Decorator:Type),
	FT=(component:Field)-[:associates]->(Component:Type)
RETURN DISTINCT
	Component.qualifiedName as Component,
	Decorator.qualifiedName as Decorator
UNION MATCH
	TMI_mM1=(Decorator:Type)-[:has_method]->(decorator_operation:Method)-[:has_invoke]->(methodCall1:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall3:MethodCall)-[:target]->(operation:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(component:Field),
	MM_TM_TT=(decorator_operation:Method)-[:overrides|implements*..]->(operation:Method)<-[:has_method]-(Component:Type)<-[:inherits*..]-(Decorator:Type),
	FT=(component:Field)-[:associates]->(Component:Type)
RETURN DISTINCT
	Component.qualifiedName as Component,
	Decorator.qualifiedName as Decorator
```



## Flyweight

$Flyweight=TMT^2+TMM\cup TM\cup TT+TFT$

```cypher
MATCH
	TFT=(FlyweightFactory:Type)-[:has_field]->(flyweights:Field)-[:associates]->(Flyweight:Type),
	TMT1=(FlyweightFactory:Type)-[:has_method]->(getFlyweight:Method)-[:return]->(Flyweight:Type),
	MT2=(getFlyweight:Method)-[:actual_return]->(ConcreteFlyweight:Type),
	TMM_TM_TT=(ConcreteFlyweight:Type)-[:has_method]->(opertionImpl:Method)-[:overrides|implements*..]->(opertion:Method)<-[:has_method]-(Flyweight:Type)<-[:inherits*..]-(ConcreteFlyweight:Type)
WHERE
	getFlyweight.isGetter CONTAINS 'true'
AND
	getFlyweight.isSetter CONTAINS 'true'
AND
	flyweights.isContainer CONTAINS 'true'
RETURN DISTINCT
	FlyweightFactory.qualifiedName as FlyweightFactory,
	Flyweight.qualifiedName as Flyweight,
	ConcreteFlyweight.qualifiedName as ConcreteFlyweight,
	flyweights.name as flyweights
UNION MATCH
	TFT=(FlyweightFactory:Type)-[:inherits*..]->(FlyweightFactorySuper:Type)-[:has_field]->(flyweights:Field)-[:associates]->(Flyweight:Type),
	TMT1=(FlyweightFactory:Type)-[:has_method]->(getFlyweight:Method)-[:return]->(Flyweight:Type),
	MT2=(getFlyweight:Method)-[:actual_return]->(ConcreteFlyweight:Type),
	TMM_TM_TT=(ConcreteFlyweight:Type)-[:has_method]->(opertionImpl:Method)-[:overrides|implements*..]->(opertion:Method)<-[:has_method]-(Flyweight:Type)<-[:inherits*..]-(ConcreteFlyweight:Type)
WHERE
	getFlyweight.isGetter CONTAINS 'true'
AND
	getFlyweight.isSetter CONTAINS 'true'
AND
	flyweights.isContainer CONTAINS 'true'
RETURN DISTINCT
	FlyweightFactory.qualifiedName as FlyweightFactory,
	Flyweight.qualifiedName as Flyweight,
	ConcreteFlyweight.qualifiedName as ConcreteFlyweight,
	flyweights.name as flyweights
```



## Proxy

$Proxy=TMI_mF\cup TMI_mM+(TMM\cup TM\cup TT)^2+FT$

```cypher
MATCH
	TMI_mM=(Proxy:Type)-[:has_method]->(proxy_request:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(real_request:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(realSubject:Field),
	TT_2_TMM_MM=(Proxy:Type)-[:inherits*..]->(Subject:Type)<-[:inherits*..]-(RealSubject:Type)-[:has_method]->(real_request:Method)-[:implements|overrides*..]->(request:Method)<-[:implements*..]-(proxy_request:Method),
	TM=(Subject:Type)-[:has_method]->(request:Method),
	FT=(realSubject:Field)-[:associates]->(RealSubject:Type)
WHERE realSubject.isContainer='false'
RETURN DISTINCT
	Subject.qualifiedName as Subject,
	Proxy.qualifiedName as Proxy,
	RealSubject.qualifiedName as RealSubject,
	realSubject.name as realSubject,
    request.signature as request
UNION MATCH
	TMI_mM=(Proxy:Type)-[:has_method]->(proxy_request:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(real_request:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(realSubject:Field),
	TT_2_TMM_MM=(Proxy:Type)-[:inherits*..]->(Subject:Type)<-[:inherits*..]-(RealSubject:Type)-[:has_method]->(real_request:Method)-[:implements|overrides*..]->(request:Method)<-[:implements*..]-(proxy_request:Method),
	TM=(Subject:Type)-[:has_method]->(request:Method),
	FT=(realSubject:Field)-[:associates]->(RealSubject:Type)
WHERE realSubject.isContainer='false'
RETURN DISTINCT
	Subject.qualifiedName as Subject,
	Proxy.qualifiedName as Proxy,
	RealSubject.qualifiedName as RealSubject,
	realSubject.name as realSubject,
    request.signature as request
```



## Chain Of Responsibility

$ChainOfResponsibility=TFT+TMI_mM+TMM\cup TM\cup TT$

```cypher
MATCH
	TFT=(Handler:Type)-[:has_field]->(successor:Field)-[:associates]->(Handler:Type),
	TMI_mM=(Handler:Type)-[:has_method]->(handleRequest:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(handleRequest:Method),
	TMM_TT=(Handler:Type)<-[:inherits*..]-(ConcreteHandler:Type)-[:has_method]->(handleRequestImpl:Method)-[:implements*..]->(handleRequest:Method)
RETURN DISTINCT
	Handler.qualifiedName as Handler,
	ConcreteHandler.qualifiedName as ConcreteHandler,
    successor.name as successor,
    handleRequest.signature as handleRequest
UNION MATCH
	TFT=(Handler:Type)-[:has_field]->(successor:Field)-[:associates]->(Handler:Type),
	TMI_mM=(Handler:Type)-[:has_method]->(handleRequest:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(handleRequest:Method),
	TMM_TT=(Handler:Type)<-[:inherits*..]-(ConcreteHandler:Type)-[:has_method]->(handleRequestImpl:Method)-[:implements*..]->(handleRequest:Method)
RETURN DISTINCT
	Handler.qualifiedName as Handler,
	ConcreteHandler.qualifiedName as ConcreteHandler,
    successor.name as successor,
    handleRequest.signature as handleRequest
```



## Command

$Command=TMI_mF\cup TMI_mM+TMM\cup TM\cup TT+TFT+TM+FT$

```cypher
MATCH
	TMI_mM=(ConcreteCommand:Type)-[:has_method]->(executeImpl:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(action:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(receiver:Field),
	TFT=(Invoker:Type)-[:has_field]->(command:Field)-[:associates]->(Command:Type),
	FT=(receiver:Field)-[:associates]->(Receiver:Type),
	MM_TM_TT=(ConcreteCommand:Type)-[:inherits*..]->(Command:Type)-[:has_method]->(execute:Method)<-[:overrides|implements*..]-(executeImpl:Method),
	TM=(Receiver:Type)-[:has_method]->(action:Method)
RETURN DISTINCT
	Invoker.qualifiedName as Invoker,
	Command.qualifiedName as Command,
	ConcreteCommand.qualifiedName as ConcreteCommand,
	Receiver.qualifiedName as Receiver,
    command.name as command,
    receiver.name as receiver,
    execute.signature as execute,
    action.signature as action
UNION MATCH
	TMI_mM=(ConcreteCommand:Type)-[:has_method]->(executeImpl:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(action:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(receiver:Field),
	TFT=(Invoker:Type)-[:has_field]->(command:Field)-[:associates]->(Command:Type),
	FT=(receiver:Field)-[:associates]->(Receiver:Type),
	MM_TM_TT=(ConcreteCommand:Type)-[:inherits*..]->(Command:Type)-[:has_method]->(execute:Method)<-[:overrides|implements*..]-(executeImpl:Method),
	TM=(Receiver:Type)-[:has_method]->(action:Method)
RETURN DISTINCT
	Invoker.qualifiedName as Invoker,
	Command.qualifiedName as Command,
	ConcreteCommand.qualifiedName as ConcreteCommand,
	Receiver.qualifiedName as Receiver,
    command.name as command,
    receiver.name as receiver,
    execute.signature as execute,
    action.signature as action
```



## Iterator

$Iterator=TMM\cup TM\cup TT+TMT\times 2+TFT+TT$

```cypher
MATCH
	TMM_TM_TT=(ConcreteAggregate:Type)-[:inherits*..]->(Aggregate:Type)-[:has_method]->(createIterator:Method)<-[:implements*..]-(createIteratorImpl:Method)<-[:has_method]-(ConcreteAggregate:Type),
	MT1=(createIterator:Method)-[:return]->(Iterator:Type),
	MT2=(createIteratorImpl:Method)-[:actual_return]->(ConcreteIterator:Type),
	TFT=(ConcreteIterator:Type)-[:has_field]->(aggregate:Field)-[:associates]->(ConcreteAggregate:Type),
	TT=(ConcreteIterator:Type)-[:inherits*..]->(Iterator:Type)
WHERE
	aggregate.isContainer CONTAINS 'true'
RETURN DISTINCT
	Aggregate.qualifiedName as Aggregate,
	ConcreteAggregate.qualifiedName as ConcreteAggregate,
	Iterator.qualifiedName as Iterator,
	ConcreteIterator.qualifiedName as ConcreteIterator,
    aggregate.name as aggregate,
    createIterator.signature as createIterator
UNION MATCH
	TMM_TM_TT=(ConcreteAggregate:Type)-[:inherits*..]->(Aggregate:Type)-[:has_method]->(createIterator:Method)<-[:implements*..]-(createIteratorImpl:Method)<-[:has_method]-(ConcreteAggregate:Type),
	MT1=(createIterator:Method)-[:return]->(Iterator:Type),
	MT2=(createIteratorImpl:Method)-[:actual_return]->(ConcreteIterator:Type),
	TTFT=(ConcreteIterator:Type)-[:inherits*..]->(ConcreteIteratorSuper:Type)-[:has_field]->(aggregate:Field)-[:associates]->(ConcreteAggregate:Type),
	TT=(ConcreteIterator:Type)-[:inherits*..]->(Iterator:Type)
WHERE
	aggregate.isContainer CONTAINS 'true'
RETURN DISTINCT
	Aggregate.qualifiedName as Aggregate,
	ConcreteAggregate.qualifiedName as ConcreteAggregate,
	Iterator.qualifiedName as Iterator,
	ConcreteIterator.qualifiedName as ConcreteIterator,
    aggregate.name as aggregate,
    createIterator.signature as createIterator
```



## Mediator

$Mediator=TFT^2+TFT+TT^2+TT$

```cypher
MATCH
	TFT_2=(ConcreteColleague1:Type)<-[:associates]-(concreteColleague1:Field)<-[:has_field]-(ConcreteMediator:Type)-[:has_field]->(concreteColleague2:Field)-[:associates]->(ConcreteColleague2:Type),
	TFT=(Colleague:Type)-[:has_field]->(mediator:Field)-[:associates]->(Mediator:Type),
	TT=(ConcreteMediator:Type)-[:inherits*..]->(Mediator:Type),
	TT_2=(ConcreteColleague1:Type)-[:inherits*..]->(Colleague:Type)<-[:inherits*..]-(ConcreteColleague2:Type)
	
RETURN DISTINCT
	Colleague.qualifiedName as Colleague,
	ConcreteColleague1.qualifiedName as ConcreteColleague1,
	ConcreteColleague2.qualifiedName as ConcreteColleague2,
	Mediator.qualifiedName as Mediator,
	ConcreteMediator.qualifiedName as ConcreteMediator,
    concreteColleague1.name as concreteColleague1,
    concreteColleague2.name as concreteColleague2,
    mediator.name as mediator
```



## Memento

$Memento=TMI_mP\cup TMI_mM+TMPT+TMT+TFT+TM^2$

```cypher
MATCH
	TMI_mM=(Originator:Type)-[:has_method]->(setMemento:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(getState:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(m:Parameter),
	MPT=(setMemento:Method)-[:has_parameter]->(m:Parameter)-[:associates]->(Memento:Type),
	TMT=(Originator:Type)-[:has_method]->(createMemento:Method)-[:return]->(Memento:Type),
	TFT=(Caretaker:Type)-[:has_field]->(memento:Field)-[:associates]->(Memento:Type),
	TM1=(Memento:Type)-[:has_method]->(getState:Method),
	TM2=(Memento:Type)-[:has_method]->(setState:Method)
WHERE
	memento.modifiers CONTAINS 'private'
AND
	getState.isGetter CONTAINS 'true'
AND
	setState.isSetter CONTAINS 'true'
RETURN DISTINCT
	Originator.qualifiedName as Originator,
	Memento.qualifiedName as Memento,
	Caretaker.qualifiedName as Caretaker,
	memento.name as memento,
	setMemento.signature as setMemento,
	createMemento.signature as createMemento,
	getState.signature as getState,
	setState.signature as setState
UNION MATCH
	TMI_mM=(Originator:Type)-[:has_method]->(setMemento:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(getState:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(m:Parameter),
	MPT=(setMemento:Method)-[:has_parameter]->(m:Parameter)-[:associates]->(Memento:Type),
	TMT=(Originator:Type)-[:has_method]->(createMemento:Method)-[:return]->(Memento:Type),
	TFT=(Caretaker:Type)-[:has_field]->(memento:Field)-[:associates]->(Memento:Type),
	TM1=(Memento:Type)-[:has_method]->(getState:Method),
	TM2=(Memento:Type)-[:has_method]->(setState:Method)
WHERE
	memento.modifiers CONTAINS 'private'
AND
	getState.isGetter CONTAINS 'true'
AND
	setState.isSetter CONTAINS 'true'
RETURN DISTINCT
	Originator.qualifiedName as Originator,
	Memento.qualifiedName as Memento,
	Caretaker.qualifiedName as Caretaker,
	memento.name as memento,
	setMemento.signature as setMemento,
	createMemento.signature as createMemento,
	getState.signature as getState,
	setState.signature as setState
```



## Observer

$Observer=TMI_mM+TMI_mF+TMM\cup TM \cup TT+TFT\times 2+TT$

```cypher
MATCH
	TMI_mM=(Subject:Type)-[:has_method]->(notify:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(update:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(observers:Field),
	TM=(Observer:Type)-[:has_method]->(update:Method),
	TF=(Subject:Type)-[:has_field]->(observers:Field)
WHERE
	observers.isContainer='true'
AND
	methodCall.blockKind='loop'
RETURN DISTINCT
	Subject.qualifiedName as Subject,
	Observer.qualifiedName as Observer
UNION MATCH
	TMI_mM=(Subject:Type)-[:has_method]->(notify:Method)-[:has_invoke]->(methodCall:MethodCall)-[:has_anonymous_argument|next*..]->(methodCall1:MethodCall)-[:target]->(update:Method),
	I_mF=(methodCall:MethodCall1)-[:caller]->(observers:Field),
	TM=(Observer:Type)-[:has_method]->(update:Method),
	TF=(Subject:Type)-[:has_field]->(observers:Field)
WHERE
	observers.isContainer='true'
AND
	methodCall1.blockKind='loop'
RETURN DISTINCT
	Subject.qualifiedName as Subject,
	Observer.qualifiedName as Observer
```



## State

$State=TMI_mF\cup TMI_mM+TMM\cup TM\cup TT+TFT\times 2$

```cypher
MATCH
	TMI_mM=(Context:Type)-[:has_method]->(request:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(handle:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(state:Field),
	TMM_TM_TT=(ConcreteState:Type)-[:inherits*..]->(State:Type)-[:has_method]->(handle:Method)<-[:implements*..]-(handleImpl:Method)<-[:has_method]-(ConcreteState:Type),
	TFT=(Context:Type)-[:has_field]->(state:Field)-[:associates]->(State:Type),
	TFT2=(ConcreteState:Type)-[:has_field]->(context:Field)-[:associates]->(Context:Type)
RETURN DISTINCT
	Context.qualifiedName as Context,
	State.qualifiedName as State,
	ConcreteState.qualifiedName as ConcreteState,
    state.name as state,
    request.signature as request,
    handle.signature as handle
UNION MATCH
	TMI_mM=(Context:Type)-[:has_method]->(request:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(handle:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(state:Field),
	TMM_TM_TT=(ConcreteState:Type)-[:inherits*..]->(State:Type)-[:has_method]->(handle:Method)<-[:implements*..]-(handleImpl:Method)<-[:has_method]-(ConcreteState:Type),
	TFT=(Context:Type)-[:has_field]->(state:Field)-[:associates]->(State:Type),
	TFT2=(ConcreteState:Type)-[:has_field]->(context:Field)-[:associates]->(Context:Type)
RETURN DISTINCT
	Context.qualifiedName as Context,
	State.qualifiedName as State,
	ConcreteState.qualifiedName as ConcreteState,
    state.name as state,
    request.signature as request,
    handle.signature as handle
UNION MATCH
	TMI_mM=(Context:Type)-[:has_method]->(request:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(handle:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(state:Field),
	TMM_TM_TT=(ConcreteState:Type)-[:inherits*..]->(State:Type)-[:has_method]->(handle:Method)<-[:implements*..]-(handleImpl:Method)<-[:has_method]-(ConcreteState:Type),
	TFT=(Context:Type)-[:inherits*..]->(ContextSuper:Type)-[:has_field]->(state:Field)-[:associates]->(State:Type),
	TFT2=(ConcreteState:Type)-[:has_field]->(context:Field)-[:associates]->(Context:Type)
RETURN DISTINCT
	Context.qualifiedName as Context,
	State.qualifiedName as State,
	ConcreteState.qualifiedName as ConcreteState,
    state.name as state,
    request.signature as request,
    handle.signature as handle
UNION MATCH
	TMI_mM=(Context:Type)-[:has_method]->(request:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(handle:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(state:Field),
	TMM_TM_TT=(ConcreteState:Type)-[:inherits*..]->(State:Type)-[:has_method]->(handle:Method)<-[:implements*..]-(handleImpl:Method)<-[:has_method]-(ConcreteState:Type),
	TFT=(Context:Type)-[:inherits*..]->(ContextSuper:Type)-[:has_field]->(state:Field)-[:associates]->(State:Type),
	TFT2=(ConcreteState:Type)-[:has_field]->(context:Field)-[:associates]->(Context:Type)
RETURN DISTINCT
	Context.qualifiedName as Context,
	State.qualifiedName as State,
	ConcreteState.qualifiedName as ConcreteState,
    state.name as state,
    request.signature as request,
    handle.signature as handle
```



## Strategy

$Strategy=TMI_mF\cup TMI_mM+TMM\cup TM\cup TT+TFT$

```cypher
MATCH
	TMI_mM=(Context:Type)-[:has_method]->(contextInterface:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(algorithmInterface:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(strategy:Field),
	TMM_TM_TT=(ConcreteStrategy:Type)-[:inherits*..]->(Strategy:Type)-[:has_method]->(algorithmInterface:Method)<-[:implements*..]-(algorithmInterfaceImpl:Method)<-[:has_method]-(ConcreteStrategy:Type),
	TFT=(Context:Type)-[:has_field]->(strategy:Field)-[:associates]->(Strategy:Type)
RETURN DISTINCT
	Context.qualifiedName as Context,
	Strategy.qualifiedName as Strategy,
	ConcreteStrategy.qualifiedName as ConcreteStrategy,
    strategy.name as strategy,
    contextInterface.signature as contextInterface,
    algorithmInterface.signature as algorithmInterface
UNION MATCH
	TMI_mM=(Context:Type)-[:has_method]->(contextInterface:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(algorithmInterface:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(strategy:Field),
	TMM_TM_TT=(ConcreteStrategy:Type)-[:inherits*..]->(Strategy:Type)-[:has_method]->(algorithmInterface:Method)<-[:implements*..]-(algorithmInterfaceImpl:Method)<-[:has_method]-(ConcreteStrategy:Type),
	TFT=(Context:Type)-[:has_field]->(strategy:Field)-[:associates]->(Strategy:Type)
RETURN DISTINCT
	Context.qualifiedName as Context,
	Strategy.qualifiedName as Strategy,
	ConcreteStrategy.qualifiedName as ConcreteStrategy,
    strategy.name as strategy,
    contextInterface.signature as contextInterface,
    algorithmInterface.signature as algorithmInterface
UNION MATCH
	TMI_mM=(Context:Type)-[:has_method]->(contextInterface:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(algorithmInterface:Method),
	I_mF=(methodCall1:MethodCall)-[:caller]->(strategy:Field),
	TMM_TM_TT=(ConcreteStrategy:Type)-[:inherits*..]->(Strategy:Type)-[:has_method]->(algorithmInterface:Method)<-[:implements*..]-(algorithmInterfaceImpl:Method)<-[:has_method]-(ConcreteStrategy:Type),
	TFT=(Context:Type)-[:inherits*..]->(ContextSuper:Type)-[:has_field]->(strategy:Field)-[:associates]->(Strategy:Type)
RETURN DISTINCT
	Context.qualifiedName as Context,
	Strategy.qualifiedName as Strategy,
	ConcreteStrategy.qualifiedName as ConcreteStrategy,
    strategy.name as strategy,
    contextInterface.signature as contextInterface,
    algorithmInterface.signature as algorithmInterface
UNION MATCH
	TMI_mM=(Context:Type)-[:has_method]->(contextInterface:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(algorithmInterface:Method),
	I_mF=(methodCall:MethodCall)-[:caller]->(strategy:Field),
	TMM_TM_TT=(ConcreteStrategy:Type)-[:inherits*..]->(Strategy:Type)-[:has_method]->(algorithmInterface:Method)<-[:implements*..]-(algorithmInterfaceImpl:Method)<-[:has_method]-(ConcreteStrategy:Type),
	TFT=(Context:Type)-[:inherits*..]->(ContextSuper:Type)-[:has_field]->(strategy:Field)-[:associates]->(Strategy:Type)
RETURN DISTINCT
	Context.qualifiedName as Context,
	Strategy.qualifiedName as Strategy,
	ConcreteStrategy.qualifiedName as ConcreteStrategy,
    strategy.name as strategy,
    contextInterface.signature as contextInterface,
    algorithmInterface.signature as algorithmInterface
```



## Template Method

$TemplateMethod=TMI_mM+TMM\cup TM\cup TT$

```cypher
MATCH
	TMI_mM=(AbstractClass:Type)-[:has_method]->(templateMethod:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(primitiveOperation:Method),
	TMM_TM_TT=(ConcreteClass:Type)-[:inherits*..]->(AbstractClass:Type)-[:has_method]->(primitiveOperation:Method)<-[:overrides|implements*..]-(primitiveOperationImpl:Method)<-[:has_method]-(ConcreteClass:Type)
RETURN DISTINCT
	AbstractClass.qualifiedName as AbstractClass,
	ConcreteClass.qualifiedName as ConcreteClass,
    templateMethod.signature as templateMethod,
    primitiveOperation.signature as primitiveOperation
UNION MATCH
	TMI_mM=(AbstractClass:Type)-[:has_method]->(templateMethod:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(primitiveOperation:Method),
	TMM_TM_TT=(ConcreteClass:Type)-[:inherits*..]->(AbstractClass:Type)-[:has_method]->(primitiveOperation:Method)<-[:overrides|implements*..]-(primitiveOperationImpl:Method)<-[:has_method]-(ConcreteClass:Type)
RETURN DISTINCT
	AbstractClass.qualifiedName as AbstractClass,
	ConcreteClass.qualifiedName as ConcreteClass,
    templateMethod.signature as templateMethod,
    primitiveOperation.signature as primitiveOperation
```



## Visitor

$Visitor=TMI_mP\cup  TMI_mM+TMPT\times 2+TMM\cup TM\cup TT\times 2$

```cypher
MATCH
	TMI_mM=(ConcreteElement:Type)-[:has_method]->(acceptImpl:Method)-[:has_invoke]->(methodCall:MethodCall)-[:target]->(visit:Method),
	I_mP=(methodCall:MethodCall)-[:caller]->(v:Parameter),
	TMPT1=(Visitor:Type)-[:has_method]->(visit:Method)-[:has_parameter]->(element:Parameter)-[:associates]->(ConcreteElement:Type),
	MPT2=(acceptImpl:Method)-[:has_parameter]->(v:Parameter)-[:associates]->(Visitor:Type),
	TMM_TT1=(Visitor:Type)<-[:inherits*..]-(ConcreteVisitor:Type)-[:has_method]->(visitImpl:Method)-[:overrides|implements*..]->(visit:Method),
	TMM_TT2=(ConcreteElement:Type)-[:inherits*..]->(Element:Type)-[:has_method]->(accept:Method)<-[:overrides|implements*..]-(acceptImpl:Method)
RETURN DISTINCT
	Element.qualifiedName as Element,
	ConcreteElement.qualifiedName as ConcreteElement,
	Visitor.qualifiedName as Visitor,
	ConcreteVisitor.qualifiedName as ConcreteVisitor,
    visit.signature as visit,
    accept.signature as accept
UNION MATCH
	TMI_mM=(ConcreteElement:Type)-[:has_method]->(acceptImpl:Method)-[:has_invoke]->(methodCall:MethodCall)-[:next|has_anonymous_argument*..]->(methodCall1:MethodCall)-[:target]->(visit:Method),
	I_mP=(methodCall1:MethodCall)-[:caller]->(v:Parameter),
	TMPT1=(Visitor:Type)-[:has_method]->(visit:Method)-[:has_parameter]->(element:Parameter)-[:associates]->(ConcreteElement:Type),
	MPT2=(acceptImpl:Method)-[:has_parameter]->(v:Parameter)-[:associates]->(Visitor:Type),
	TMM_TT1=(Visitor:Type)<-[:inherits*..]-(ConcreteVisitor:Type)-[:has_method]->(visitImpl:Method)-[:overrides|implements*..]->(visit:Method),
	TMM_TT2=(ConcreteElement:Type)-[:inherits*..]->(Element:Type)-[:has_method]->(accept:Method)<-[:overrides|implements*..]-(acceptImpl:Method)
RETURN DISTINCT
	Element.qualifiedName as Element,
	ConcreteElement.qualifiedName as ConcreteElement,
	Visitor.qualifiedName as Visitor,
	ConcreteVisitor.qualifiedName as ConcreteVisitor,
    visit.signature as visit,
    accept.signature as accept
```

