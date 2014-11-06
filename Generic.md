##Generic
Type Parameter vs. Type Argument

Unbound Generic Type vs. Constructed Type 

Open Type vs. Closed Type

##Type Constraints
1. Reference Type Constraints
	- `T : class`
2. Value Type Constraints
	- `T : struct`
3. Constructor Type Constraints
	- `T : new()`
	- Must be *last* constraint for any particular type parameter  
	- For any value type, any nonstatic, nonabstract class without any explicitly declared constructors, any nonabstract class with any explicit public parameterless constructor
	- Can be useful when you need to use factory-like patterns
4. Conversion Type Constraints
	- lets you specify another type that the type argument must be implicitly convertible to via an identity, reference, or boxing conversion
	- You can specify that one type argument be convertible to another type argument - this is called a *type parameter* constraint
	- You can specify multiple interfaces but only one class
	- The type you specified cannot be a value type, a sealed class (such as string), or any of the following "special" types:
		- `System.Object`
		- `System.Enum`
		- `System.ValueType`
		- `System.Delegate`
		
	  Workaround to the limitations: http://code.google.com/p/unconstrained-melody/

##The Generic Comparison Interfaces
Compare values for ordering:

	- IComparer<T>
	- IComparable<T>

Compare two items for equality and for finding the hash of an item:

	- IEqualityComparer<T>	
	- IEquatable<T>

Compare two different values:

	- IComparer<T>
	- IEqualityComparer<T>

Compare itself with another value:

	- IComparable<T>
	- IEquatable<T>

##Static fields and Static Constructors
Static fields belong to the type they're declared in. If you declare a static field `x` in class `SomeClass`, there's exactly one `SomeClass.x` field per *application domain*, no matter how many instances of `SomeClass` you create, and no matter how many types derives from `SomeClass`. Variables decorated with `[ThreadStatic]` violate this rule, too.

Each *closed* generic type has its own set of static fields: one static field per closed type. The same applies for static initializers and static constructors. Just as with nongeneric types, the static constructor for any closed type is only executed once.

For nested generic types, the same rule applies. That being said, there is only one `Outer<int>.Inner`, or `Outer<string>.Inner`, with separate set of static fields.

##How the JIT Compiler Handles Generics
The JIT creates different code for each closed type with a type argument that's a value type. But it shares the native code generated for all the closed type that use a reference type as the type argument. It can do this because all references are same size. The JIT can use the same optimizations to store references in registers regardless of the type.

Each of the types still has its own static fields, but the executable code itself is reused.

- http://geekswithblogs.net/akraus1/archive/2012/07/25/150301.aspx
- http://msdn.microsoft.com/en-us/magazine/cc163791.aspx

##Using typeof with generic types
There are two ways of using typeof with generic types - one retrieves the generic type definition (the unbound generic type) and one retrieves a particular constructed type.

    string listTypeName = "System.Collections.Generic.List`1";
    
    Type defByName = Type.GetType(listTypeName);
    
    Type closedByName = Type.GetType(listTypeName + "[System.String]");
    Type closedByMethod = defByName.MakeGenericType(typeof(string));
    Type closedByTypeof = typeof(List<string>);
    
    Console.WriteLine(closedByMethod == closedByName);
    Console.WriteLine(closedByName == closedByTypeof);
    
    Type defByTypeof = typeof(List<>);
    Type defByMethod = closedByName.GetGenericTypeDefinition();
    
    Console.WriteLine(defByMethod == defByName);
    Console.WriteLine(defByName == defByTypeof);

##Reflecting generic methods
    void Main()
    {
    	Type type = GetType();
    	MethodInfo definition = type.GetMethod("PrintTypeParameter");
    	MethodInfo constructed = definition.MakeGenericMethod(typeof(string));
    	constructed.Invoke(null, null);
    }
    
    public static void PrintTypeParameter<T>()
    {
    	Console.WriteLine (typeof(T));
    }

##Limitations of generics in C#
1. **Lack of generic variance (covariance and contravariance). Generics are invariant.** With covariance, you were trying to convert from `SomeType<Circle>` to `SomeType<IShape>`. Contravariance is about converting the other way - from `SomeType<IShape>` to `SomeType<Circle>`. Covariance is safe when `SomeType` only describes operations that returns the type parameter, and contravariance is safe when `SomeType` only describes operations that accept the type parameter.
2. **Lack of operator constraints or a "numeric" constraint.** A workaround to this limitation  http://yoda.arachsys.com/csharp/genericoperators.html
3. **Lack of generic properties, indexers, and other member types.**

##Main benefits of generics:
1. Compile-time type safety
2. Performance
3. Code expressiveness
