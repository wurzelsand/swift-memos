# Self in Closures

## Aufgabe

Wir erzeugen innerhalb des `do`-Blocks ein Objekt der Klasse `Person`. Eigentlich sollte das Objekt nach Beendigung des `do`-Blocks deinitialisiert werden. Da wir hier aber ein *Retain-Cycle* haben, wird `deinit` nicht aufgerufen:

```swift
class Person {
    var sentence: String = "Hello!"
    var speak: () -> Void = {}
    
    init() {
        speak = {
            print(self.sentence)
        }
    }
    
    deinit {
        print("deinit")
    }
}

do {
    let peter = Person()
    peter.sentence = "Bye!"
    peter.speak()
}
```

Welche Möglichkeiten gibt es, den *Retain-Cycle* aufzulösen?

## Ausführung

* `unowned`

  ```swift
  speak = { [unowned self] in
      print(self.sentence) // Bye!
  }
  ```

* `unowned`; ohne explizites `self` im Closure

  ```swift
  speak = { [unowned self] in
      print(sentence) // Bye!
  }
  ```

* capture

  ```swift
  speak = { [sentence] in
      print(sentence) // Hello!
  }
  ```
  
* `weak`

  ```swift
  speak = { [weak self] in
      if let self = self {
          print(self.sentence) // Bye!
      }
  }
  ```
