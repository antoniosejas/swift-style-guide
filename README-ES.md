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

_Razonamiento:_ La intención y el significado  de ambas palabras reservadas está clara, pero utilizar *let* por defecto es más seguro y más limpio.

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

_Razonamiento:_ Es más seguro el uso de `if let`-binding para el resultado de optionals. El unwrapping forzado es más propenso a provocar errores en tiempo de ejecución.

#### Evita utilizar implícitamente Unwrapped Optionals

Donde sea posible, utiliza `let foo: FooType?` en vez de `let foo: FooType!` is `foo` puede ser nil (Por lo general, `?` se puede utilizar en vez de `!`).

_Razonamiento:_ El resultado explícito de los optionals es más seguro. Implícitamente, el unwrapped de los optionals pueden producir fallos en tiempo de ejcución.

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

_Razonamiento:_ La intención y el significado de la primera versión es clara y ocupa menos líneas de código.

#### Espcifica siempre el control de acceso de forma expícita para definiciones de alto nivel

Funciones de alto nivel, tipos, y variables deberías siempre especificar explícitamente el control de acceso:

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

Sin embargo, las definiciones dentro de dichos objetos pueden expresar el control de acceso de manera implícita, en tal caso:

```swift
internal struct TheFez {
    var owner: Person = Joshaber()
}
```

_Razonamiento:_ Es raramente apropiado que las definiciones de alto nivel sean específicamente `internal`. Al ser explicitos asegura que tendremos en mente esa decisión. Dentro de una definición, no es necesario volver a especificar el acceso de control otra vez, sería duplicar código y el por defecto normalmente es lomás razobnable.

#### Posición de los dos pontos al definir el tipo de un identificador.

Cuando especifiques el tipo de un identificador, coloca los dos puntos inmediatamente después del identificador, sin espacio entre ellos y a continuación escribe un espacio y el nombre del tipo.

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

_Razonamiento:_ El tipo especifica algo sobre el _identificador_, entonces los dos puntos deben ir con el _identificador_.

También, cuando especifiques el tipo de un diccionario, siempre pon los dos puntos inmediatamente después de la clave, seguido de un espacio y el valor.

```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

#### Solo utiliza `self` cuando sea absolutamente necesario

Por defecto, no escribas `self` cuando accedas a propiedades o métodos de tu propia clase. Ya se entiende de forma implicita.

```swift
private class History {
    var events: [Event]

    func rewrite() {
        events = []
    }
}
```

Solamente incluye `self` de forma explícita cuando realmente sea impuesto por Swift, o cuando haya conflicto entre los nombres de los parámetros.

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

_Razonamiento:_ Esto nos permite destacar `self` en los sitios realmente necesarios y evita la verbosidad en el resto del código.

#### Preferlible usar structs y no classes

Utiliza siempre struct, a menos que necesites una funcionalidad que solo se puede conseguir utilizando una clase ( como identidad o desinicializadores ).

La herencia por si misma, normalmente no es una razón suficiente para utilizar clases, ya que el polimorfismo puede ser ofrecido por los protocolos, y la reusar código puede ser satisfacido por la composición.

Por ejemplo esta herencia de clase :

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

podría ser refactorizada en estas definiciones:

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

_Razonamiento:_ Los tipo valor son más simples, más fáciles de razonar, y su comportamiento es el esperado al declararlos con `let`

#### Por defecto, escribe clases `final`

Las clases deberían empezar con `final`, y sólo ser cambiadas para permitir subclases si hemos identificado una necesidad válida y suficiente para herencia. Incluso en el caso de que haya muchas definiciones, la clase también debería ser `final`, siguiendo las mismas reglas.

_Razonamiento:_ Composición normalmente es  preferible a herencia, y si optas por la herencia, que sea porque has lo has pensado cuidadosamente.


#### Omite el tipo de los parámetros siempre que sea posible

Los métodos de tipos parametrizados pueden omitir el tipo de los parámetros al recibir el tipo cuando son identicos a los del receptor. Por ejemplo:

```swift
struct Composite<T> {
    …
    func compose(other: Composite<T>) -> Composite<T> {
        return Composite<T>(self, other)
    }
}
```

podría ser sustituido por:

```swift
struct Composite<T> {
    …
    func compose(other: Composite) -> Composite {
        return Composite(self, other)
    }
}
```

_Razonamiento:_ Omitiendo el tipo de los parámetros redundantes deja más clara la intención y por el contrario , cuando especificamos el tipo, hace que sea evidiente que este es otro diferente.

#### Utiliza espacios en blanco alrededor de la definición de operadores

Cuando definas operadores, escribe espacios en balnco anes y después. En vez de:

```swift
func <|(lhs: Int, rhs: Int) -> Int
func <|<<A>(lhs: A, rhs: A) -> A
```

escribe:

```swift
func <| (lhs: Int, rhs: Int) -> Int
func <|< <A>(lhs: A, rhs: A) -> A
```

_Razonamiento:_ Los operadores son caracteres de puntuación, los cuales pueden ser dificiles de leer si están pegados a un tipo o parámetro. Añadiendo estos espacios, separamos los operadores de una forma clara.

#### Traducciones

* [English Version](https://github.com/github/swift-style-guide)
* [中文版](https://github.com/Artwalk/swift-style-guide/blob/master/README_CN.md)
* [日本語版](https://github.com/jarinosuke/swift-style-guide/blob/master/README_JP.md)
* [한국어판](https://github.com/minsOne/swift-style-guide/blob/master/README_KR.md)
