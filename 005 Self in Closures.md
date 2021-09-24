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
  
## Diskussion
  
`self` in einem Closure erzeugt nur dann ein *Retain Cycle*, wenn das Objekt eine Referenz auf das Closure speichert. Im folgenden gibt es daher kein *Retain Cycle*:
  
```swift
import Foundation

class Person {
    var sentence: String = "Hello!"
    
    func speak() {
        DispatchQueue.global(qos: .default).async {
            print(self.sentence)
            dispatchGroup.leave()
        }
    }
    
    deinit {
        print("deinit")
    }
}

let dispatchGroup = DispatchGroup()
dispatchGroup.enter()
do {
    let peter = Person()
    peter.sentence = "Bye!"
    peter.speak()
}
dispatchGroup.wait()
```

Manchmal erkennt man nicht auf den ersten Blick, dass ein Objekt eine Referenz auf ein Closure speichert. Bei [SwiftBySundell](https://www.swiftbysundell.com/clips/6/) bin ich z. B. auf so etwas gestoßen:

```swift
class CanvasViewController {
    private var canvas = Canvas()
    private lazy var previewImageView = ImageView()
    
    func renderPreviewImage() {
        canvas.renderAsImage { [weak self] image in
            self?.previewImageView.image = image
        }
    }
}
```

Wieso sollte hier `weak` notwendig sein? Schließlich speichert `canvas` nicht das Closure sondern übergibt es nur einem seiner Methoden als Parameter. Und tatsächlich ergibt sich nicht zwangsweise ein Retain-Cycle, `deinit` würde also auch ohne `weak` aufgerufen werden:

```swift
class Image {}

class ImageView {
    var image = Image()
}

class Canvas {
    private var image = Image()
    
    func renderAsImage(render: @escaping (Image) -> Void) { // escaping not necessary here
        render(image)
    }
}

class CanvasViewController {
    private var canvas = Canvas()
    private lazy var previewImageView = ImageView()
    
    func renderPreviewImage() {
        canvas.renderAsImage { image in
            self.previewImageView.image = image
        }
    }
    
    deinit {
        print("deinit")
    }
}

do {
    let canvasViewController = CanvasViewController()
    canvasViewController.renderPreviewImage()
}
```

Anders sieht es aber aus, wenn der Closure-Parameter in der Methode ein *Escaping-Closure* ist und in `Canvas` gespeichert wird:

```swift
class Image {}

class ImageView {
    var image = Image()
}

class Canvas {
    private var image = Image()
    var renderFunction: (Image) -> Void = { _ in }
    
    func renderAsImage(render: @escaping (Image) -> Void) {
        renderFunction = render
        render(image)
    }
}

class CanvasViewController {
    private var canvas = Canvas()
    private lazy var previewImageView = ImageView()
    
    func renderPreviewImage() {
        canvas.renderAsImage { [weak self] image in
            self?.previewImageView.image = image
        }
    }
    
    deinit {
        print("deinit")
    }
}

do {
    let canvasViewController = CanvasViewController()
    canvasViewController.renderPreviewImage()
}
```

Wenn wir also eine Methode einer Instanzvariablen unserer Klasse aufrufen und ihr ein Closure, das eine Referenz auf `self` enthält, als Parameter übergeben, muss dieses `self` entweder `weak` oder `unowned` sein.
