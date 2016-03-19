Guía de estilo y convenciones de programacón para proyectos en Swift.

Esta guía es un intento para promover patrones que cumplan con las siguientes metas (no necesariamente en este orden):

 1. Conseguir un código más robusto y reducir la probabilidad de error al programar
 2. Aumentar la legibilidad del código, consiguiendo que sea más claro y autoexplicativo
 3. Reducir código innecesario y redundante
 4. Reducir los debates sobre estética

Si tienes sugerencias, por favor primero lee la  [guía para colaborar](CONTRIBUTING.md),
después haz un pull request. :zap:

----

#### Espacios en blanco

 * Tabuladores, no espacios.
 * El final del archivo acaba con una nueva línea.
 * Utiliza sin miedo los espacios en blanco para separar el código en trozos entendibles.
 * No deje espacios en blanco a la derecha de la línea.
   *  No indentar las líneas en blanco. Estas no deben llevar ni espacios ni tabulaciones.


#### Es preferlible utilizar `let`-bindings sobre `var`-bindings siempre que sea posible

Usar `let foo = …` sobre `var foo = …` siempre que sea posible (y también ante la duda). Solo usar `var` si estás forzado a ello (es decir, tu *sabes* que el valor va a cambiar, por ejemplo cuando usas la propiedad `weak`).

_Razón:_ La intención y el significado  de ambas palabras reservadas está clara, pero utilizar *let* por defecto es más seguro y más limpio.

El `let`-binding garantiza y *recalca al programador* que su valor nunca cambiará. Esto nos permite mantener una fuerte suposición de que su valor no cambiará en el código que le sigue.

Se vuelve más fácil para razonar sobre el código. Al utilizar `var` creamos una duda de si el valor de la variable ha cambiado, lo que deberemos comprobar de forma manual.

En cnosecuencia, siempre que veas un identificador `var`, asume que su valor cambiará y pregúntate por qué.

### Return y break

Cuando tengas que comprobar ciertas condiciones para continuar la ejecución, intenta salir cuanto antes. Entonces, en vez de:

```swift
if n.isNumber {
    // Use n here
} else {
    return
}
```

utiliza esto:
```swift
guard n.isNumber else {
    return
}
// Use n here
```

También lo puedes hacer con `if`, pero es mejor utilizar `guard`, porque `guard` nos obliga a utilizar `return`, `break` o `continue`, o el compilador lanzará un error. Por esta razón la salida está garantizada con `guard`.

#### Evita usar el Unwrapping forzado de los Optionals

Si tu tienes un identificador `foo` de tipo `FooType?` o `FooType!`, si es posible no fuerces su unwrap para conseguir el valor (`foo!`).
En vez de forzar es preferible:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```

Como alternativa, en el caso de que quieras utilizar la concatenación de Optionals, puedes hacerlo así:

```swift
// Call the function if `foo` is not nil. If `foo` is nil, ignore we ever tried to make the call
foo?.callSomethingIfFooIsNotNil()
```

_Razón:_ Es más seguro el uso de `if let`-binding para el resultado de optionals. El unwrapping forzado es más propenso a provocar errores en tiempo de ejecución.

#### Evita utilizar implícitamente Unwrapped Optionals

Donde sea posible, utiliza `let foo: FooType?` en vez de `let foo: FooType!` is `foo` puede ser nil (Por lo general, `?` se puede utilizar en vez de `!`).

_Razón:_ El resultado explícito de los optionals es más seguro. Implícitamente, el unwrapped de los optionals pueden producir fallos en tiempo de ejcución.

#### Utilizar los getters de forma implícita en las propiedades de solo lectura y subscripts

Cuando sea posible, omite el `get` en las propiedades de solo lectura y en los subscripts de solo lectura.

Por lo tanto, escribe esto:

```swift
var myGreatProperty: Int {
	return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

… y no esto:

```swift
var myGreatProperty: Int {
	get {
		return 4
	}
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

_Razón:_ La intención y el significado de la primera versión es clara y ocupa menos líneas de código.

#### Always specify access control explicitly for top-level definitions

Top-level functions, types, and variables should always have explicit access control specifiers:

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

However, definitions within those can leave access control implicit, where appropriate:

```swift
internal struct TheFez {
	var owner: Person = Joshaber()
}
```

_Rationale:_ It's rarely appropriate for top-level definitions to be specifically `internal`, and being explicit ensures that careful thought goes into that decision. Within a definition, reusing the same access control specifier is just duplicative, and the default is usually reasonable.

#### When specifying a type, always associate the colon with the identifier

When specifying the type of an identifier, always put the colon immediately
after the identifier, followed by a space and then the type name.

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

_Rationale:_ The type specifier is saying something about the _identifier_ so
it should be positioned with it.

Also, when specifying the type of a dictionary, always put the colon immediately
after the key type, followed by a space and then the value type.

```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

#### Only explicitly refer to `self` when required

When accessing properties or methods on `self`, leave the reference to `self` implicit by default:

```swift
private class History {
	var events: [Event]

	func rewrite() {
		events = []
	}
}
```

Only include the explicit keyword when required by the language—for example, in a closure, or when parameter names conflict:

```swift
extension History {
	init(events: [Event]) {
		self.events = events
	}

	var whenVictorious: () -> () {
		return {
			self.rewrite()
		}
	}
}
```

_Rationale:_ This makes the capturing semantics of `self` stand out more in closures, and avoids verbosity elsewhere.

#### Prefer structs over classes

Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead.

Note that inheritance is (by itself) usually _not_ a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

For example, this class hierarchy:

```swift
class Vehicle {
    let numberOfWheels: Int

    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func maximumTotalTirePressure(pressurePerWheel: Float) -> Float {
        return pressurePerWheel * Float(numberOfWheels)
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```

could be refactored into these definitions:

```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
}

func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
    return pressurePerWheel * Float(vehicle.numberOfWheels)
}

struct Bicycle: Vehicle {
    let numberOfWheels = 2
}

struct Car: Vehicle {
    let numberOfWheels = 4
}
```

_Rationale:_ Value types are simpler, easier to reason about, and behave as expected with the `let` keyword.

#### Make classes `final` by default

Classes should start as `final`, and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible _within_ the class should be `final` as well, following the same rules.

_Rationale:_ Composition is usually preferable to inheritance, and opting _in_ to inheritance hopefully means that more thought will be put into the decision.


#### Omit type parameters where possible

Methods of parameterized types can omit type parameters on the receiving type when they’re identical to the receiver’s. For example:

```swift
struct Composite<T> {
	…
	func compose(other: Composite<T>) -> Composite<T> {
		return Composite<T>(self, other)
	}
}
```

could be rendered as:

```swift
struct Composite<T> {
	…
	func compose(other: Composite) -> Composite {
		return Composite(self, other)
	}
}
```

_Rationale:_ Omitting redundant type parameters clarifies the intent, and makes it obvious by contrast when the returned type takes different type parameters.

#### Use whitespace around operator definitions

Use whitespace around operators when defining them. Instead of:

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

write:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

_Rationale:_ Operators consist of punctuation characters, which can make them difficult to read when immediately followed by the punctuation for a type or value parameter list. Adding whitespace separates the two more clearly.

#### Translations

* [English Version](https://github.com/github/swift-style-guide)
* [中文版](https://github.com/Artwalk/swift-style-guide/blob/master/README_CN.md)
* [日本語版](https://github.com/jarinosuke/swift-style-guide/blob/master/README_JP.md)
* [한국어판](https://github.com/minsOne/swift-style-guide/blob/master/README_KR.md)
