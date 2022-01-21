### share

```swift
import UIKit
import Combine

guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else {
    fatalError("Invalid URL")
}

let request = URLSession.shared.dataTaskPublisher(for: url)
    .map(\.data)
    .print()
    .share() // share

let subscription1 = request.sink(receiveCompletion: { _ in }) {
    print("Subscription 1")
    print($0)
}

let subscription2 = request.sink(receiveCompletion: { _ in }) {
    print("Subscription 2")
    print($0)
}
```

### problem with share

- some async task can’t get shared resources

```swift
import UIKit
import Combine

var subscription3: AnyCancellable? = nil

guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else {
    fatalError("Invalid URL")
}

let request = URLSession.shared.dataTaskPublisher(for: url)
    .map(\.data)
    .print()
    .share() // share

let subscription1 = request.sink(receiveCompletion: { _ in }) {
    print("Subscription 1")
    print($0)
}

let subscription2 = request.sink(receiveCompletion: { _ in }) {
    print("Subscription 2")
    print($0)
}

// not possible with share
DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
    subscription3 = request.sink(receiveCompletion: {_ in }, receiveValue: {
        print("Subscription 3")
        print($0)
    }) // nothing will be printed out 
}
```

### Multicast

```swift
import UIKit
import Combine

guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else {
    fatalError("Invalid URL")
}

let subject = PassthroughSubject<Data, URLError>()

let request = URLSession.shared.dataTaskPublisher(for: url)
    .map(\.data)
    .print()
    .multicast(subject: subject)

let subscription1 = request.sink(receiveCompletion: { _ in }) {
    print("Subscription 1")
    print($0)
}

let subscription2 = request.sink(receiveCompletion: { _ in }) {
    print("Subscription 2")
    print($0)
}

let subscription3 = request.sink(receiveCompletion: {_ in }, receiveValue: {
    print("Subscription 3")
    print($0)
})

request.connect() // allows multicast 
subject.send(Data())
```
