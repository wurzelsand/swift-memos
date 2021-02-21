# UIDelegates

Ein beliebtes Muster in UIKit sind Delegates. Wenn wir beispielsweise eine Klasse von *UICollectionViewController* ableiten, können wir zwar eine Reihe von Methoden überschreiben. z. B. teilen wir durch das Überschreiben von `numberOfSections(in collectionView: UICollectionView)` dem Controller mit, aus wie vielen Sektoren unser *UICollectionView* bestehen soll. Andere Funktionen des *UICollectionViewControllers* können wir aber erst nach dem Adaptieren des *UICollectionViewDelegateFlowLayout*-Interfaces beeinflussen, z. B. das Setzen der Zellengröße. Dabei müssen wir nicht einmal der Delegate-Instanzvariablen ein Objekt zuordnen. Ausreichend ist die bloße Deklaration und schon sind alle Instanzen unserer Klasse, die das Delegate adaptiert, automatisch auch Empfänger der Delegate-Methoden:

```swift
class MyCollectionViewController: UICollectionViewController, UICollectionViewDelegateFlowLayout {...}
```

## Aufgabe

Gegeben sind eine UI-Klasse und sein Delegate-Protokoll:

```swift
protocol UIObjectDelegate: AnyObject {
    func objectChanged(_ object: UIObject, to newValue: Int)
    func objectWillAppear(_ object: UIObject)
    func objectMoved(_ object: UIObject)
}

class UIObject {
    weak var delegate: UIObjectDelegate?
    
    var x = 42 ...
    
    func move() ... 
}
```

Wir leiten eine Klasse ab und deklarieren sie als Delegate. Das Delegate-Protokoll besteht aus 3 Methoden. Wir implementieren aber hier nur zwei davon:

```swift
class Object: UIObject, UIObjectDelegate {

    func objectChanged(_ object: UIObject, to newValue: Int) {
        print("Objects property changed to \(newValue)")
    }
    
    func objectWillAppear(_ object: UIObject) {
        print("Object with x = \(object.x) will appear")
    }
    
}
```

Nun testen wir das ganze:

```swift
do {
    let object = Object()
    object.x = 4711
    object.move()
}
```

Ausgabe:

```
Object with x = 42 will appear
Objects property changed to 4711
```

Damit es funktioniert, müssen wir *UIObject* noch vervollständigen und das Delegate-Protokoll mit einer Extension ergänzen.

## Ausführung

```swift
protocol UIObjectDelegate: AnyObject { // #1 {
    func objectChanged(_ object: UIObject, to newValue: Int)
    func objectWillAppear(_ object: UIObject)
    func objectMoved(_ object: UIObject)
}

extension UIObjectDelegate {
    func objectChanged(_ object: UIObject, to newValue: Int) {}
    func objectWillAppear(_ object: UIObject) {}
    func objectMoved(_ object: UIObject) {}
}

class UIObject {
    weak var delegate: UIObjectDelegate? // #2
    
    var x = 42 {
        didSet {
            delegate?.objectChanged(self, to: x)
        }
    }
    
    init() {
        self.delegate = self as? UIObjectDelegate
        delegate?.objectWillAppear(self)
    }
    
    func move() {
        delegate?.objectMoved(self)
    }
    
}

class Object: UIObject, UIObjectDelegate {

    func objectChanged(_ object: UIObject, to newValue: Int) {
        print("Objects property changed to \(newValue)")
    }
    
    func objectWillAppear(_ object: UIObject) {
        print("Object with x = \(object.x) will appear")
    }
    
//    func objectMoved(_ object: UIObject) {
//        print("Object with x = \(object.x) has moved")
//    }

}

do {
    let object = Object()
    object.x = 4711
    object.move()
}
```

Diskussion:

1. `AnyObject` bewirkt, dass nur Klassen, also keine *Structs*, das Interface implementieren können. Warum? Siehe 2.
2. *Delegates* sollten *weak* sein. Daher kommen nur Klassen in Frage.
