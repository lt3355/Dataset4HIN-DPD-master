# type
simpleName
qualifiedName
packageName
modifiers
typeKind: (Class/Interface/Enum/primitive/generic)
isNested
isLocalType：whether is defined in an executable
isAnonymous

# executable
modifiers
signature
qualifiedSignature: type's qualifiedName + executable's signature
## constructor
## method
name
isGetter
isSetter
isTopDefinition

# variable
name
modifiers
isContainer
isArray
isInitialized
## field, local variable
## parameter
order

# invocation
code
order
blockKind: (loop/condition...)
## constructor call
## method call