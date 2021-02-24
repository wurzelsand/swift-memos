## ValueObservation aus GRDB verstehen

### Aufgabe

```swift
struct Observation {
    ...
}

struct Database {
    var values = [1, 2, 3, 4]
}

let database = Database()
let publisher = Observation.tracking { database in
    database.values.filter { $0 > 2}
}.publisher(in: database)

publisher()
```

Ausgabe:

<pre><code>3
4</pre></code>

Wie muss `struct Observation` implementiert sein?

### Ausführung

```swift
struct Observation {
    var reducer: (Database) -> [Int]
    
    static func tracking(reducer: @escaping (Database) -> [Int]) -> Self {
        Self.init(reducer: reducer)
    }
    
    func publisher(in database: Database) -> () -> Void {
        {
            reducer(database).forEach {
                print($0)
            }
        }
    }
}
```
