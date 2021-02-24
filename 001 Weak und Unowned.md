# Weak und Unowned

## Aufgabe

Stelle sicher, dass die erzeugten Objekte nach dem Funktionsaufruf von `trip` wieder frei gegeben werden.

```swift
class Car {...}

class Driver {...}

func trip() {
    let mercedes = Car()
    let peter = Driver()
    mercedes.driver = peter
    peter.car = mercedes
}

trip()
```

Ausgabe:

```
Driver deinit
Car deinit
```

## Ausführung

```swift
class Car {
    weak var driver: Driver?
    deinit {
        print("Car deinit")
    }
}

class Driver {
    var car: Car?
    deinit {
        print("Driver deinit")
    }
}
                                // peter   | mercedes
                                // --------|---------
func trip() {                   // (0)  0  | (0)  0   -> enter function
    let mercedes = Car()        // (0)  0  | (1)  1
    let peter = Driver()        // (1)  1  | (1)  1
    mercedes.driver = peter     // (2)  1  | (1)  1
    peter.car = mercedes        // (2)  1  | (2)  2
}                               // --------|--------- -> exit function
                                // (1)  0  | (1)  1   -> no peter-reference
                                // (1)  0  | (1)  0
trip()
```

* In der Tabelle zähle ich die Referenzen. Die eingeklammerten Zahlen stehen für die Anzahl der Referenzen, wenn `driver` nicht `weak` wäre.

## Aufgabe

Ändern wir das vorherige Beispiel und setzen diesmal voraus, dass `Car` immer einen `Driver` haben **muss**, diese Eigenschaft also nicht `nil` sein kann.

## Ausführung

```swift
class Car {
    unowned let driver: Driver
    
    init(driver: Driver) {
        self.driver = driver
    }
    
    deinit {
        print("Car deinit")
    }
}

class Driver {
    var car: Car?
    deinit {
        print("Driver deinit")
    }
}
                                        // peter   | mercedes
                                        // --------|---------
func trip() {                           // (0)  0  | (0)  0   -> enter function
    let peter = Driver()                // (1)  1  | (0)  0
    let mercedes = Car(driver: peter)   // (2)  1  | (1)  1
    peter.car = mercedes                // (2)  1  | (2)  2
}                                       // --------|--------- -> exit function
                                        // (1)  0  | (1)  1   -> no peter-reference
                                        // (1)  0  | (1)  0
trip()
```

## Diskussion

* `weak` Objekte müssen immer optional sein. `unowned` Objekte können optional sein, sind es aber in der Regel nicht.
