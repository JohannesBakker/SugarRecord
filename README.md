# <center>![xcres](https://github.com/carambalabs/SugarRecord/raw/master/Assets/Banner.png)</center>

# SugarRecord

[![Twitter: @carambalabs](https://img.shields.io/badge/contact-@carambalabs-blue.svg?style=flat)](https://twitter.com/carambalabs)
[![Language: Swift](https://img.shields.io/badge/lang-Swift-yellow.svg?style=flat)](https://developer.apple.com/swift/)
[![Language: Swift](https://img.shields.io/badge/license-MIT-lightgrey.svg?style=flat)](http://opensource.org/licenses/MIT)
[![Build Status](https://travis-ci.org/carambalabs/SugarRecord.svg)](https://travis-ci.org/carambalabs/SugarRecord)
[![Slack Status](http://sugar-record.herokuapp.com/badge.svg)](http://sugar-record.herokuapp.com/)


**If you want to receive updates about the status of SugarRecord, you can subscribe to our mailing list [here](http://eepurl.com/57tqX)**

## What is SugarRecord?
SugarRecord is a persistence wrapper designed to make working with persistence solutions like CoreData/Realm/... in a much easier way. Thanks to SugarRecord you'll be able to use CoreData with just a few lines of code: Just choose your stack and start playing with your data.

The library is maintained by [@carambalabs](https://github.com/carambalabs). You can reach me at [pepibumur@gmail.com](mailto://pepibumur@gmail.com) for help or whatever you need to commend about the library.

## Features
- Swift 2.3 compatible (Xcode 7.3).
- Fully rewritten from the version 1.0.
- Protocols based design.
- For **beginners** and **advanced** users
- Fully customizable. Build your own stack!
- Friendly syntax (fluent)
- Away from Singleton patterns! No shared states :tada:
- Compatible with OSX/iOS/watchOS/tvOS
- Fully tested (thanks Nimble and Quick)
- Actively supported

## Setup

### [CocoaPods](https://cocoapods.org)

1. Install [CocoaPods](https://cocoapods.org). You can do it with `gem install cocoapods`
2. Edit your `Podfile` file and add the following line `pod 'SugarRecord'`
3. Update your pods with the command `pod install`
4. Open the project from the generated workspace (`.xcworkspace` file).

*Note: You can also test the last commits by specifying it directly in the Podfile line*

**Available specs**
Choose the right one depending ton the configuration you need for you app.

```ruby
pod "SugarRecord/CoreData"
pod "SugarRecord/CoreData+iCloud"
pod "SugarRecord/Realm"
```

#### Notes
- SugarRecord 2.0 is not compatible with the 1.x interface. If you were using that version you'll have to update your project to support this version.

## Reference
You can check generated SugarRecord documentation [here](http://cocoadocs.org/docsets/SugarRecord/2.0.0/) generated automatically with [CocoaDocs](http://cocoadocs.org/)

# How to use

#### Creating your Storage
A storage represents your database, Realm, or CoreData. The first step to start using SugarRecord is initializing the storage. SugarRecord provides two default storages, one for CoreData, `CoreDataDefaultStorage` and another one for Realm, `RealmDefaultStorage`.

```swift
// Initializing CoreDataDefaultStorage
func coreDataStorage() -> CoreDataDefaultStorage {
    let store = CoreData.Store.Named("db")
    let bundle = NSBundle(forClass: self.classForCoder())
    let model = CoreData.ObjectModel.Merged([bundle])
    let defaultStorage = try! CoreDataDefaultStorage(store: store, model: model)
    return defaultStorage
}

// Initializing RealmDefaultStorage
func realmStorage() -> RealmDefaultStorage {
  return RealmDefaultStorage()
}
```

##### Creating an iCloud Storage

SugarRecord supports the integration of CoreData with iCloud. It's very easy to setup since it's implemented in its own storage that you can use from your app, `CoreDataiCloudStorage`:

```swift
// Initializes the CoreDataiCloudStorage
func icloudStorage() -> CoreDataiCloudStorage {
    let bundle = NSBundle(forClass: self.classForCoder())
    let model = CoreData.ObjectModel.Merged([bundle])
    let icloudConfig = iCloudConfig(ubiquitousContentName: "MyDb", ubiquitousContentURL: "Path/", ubiquitousContainerIdentifier: "com.company.MyApp.anothercontainer")
    return CoreDataiCloudStorage(model: model, iCloud: icloudConfig)
}
```

#### Contexts
Storages offer multiple kind of contexts that are the entry points to the database. For curious developers, in case of CoreData a context is a wrapper around `NSManagedObjectContext`, in case of Realm a wrapper around `Realm`. The available contexts are:

- **MainContext:** Use it for main thread operations, for example fetches whose data will be presented in the UI.
- **SaveContext:** Use this context for background operations. The context is initialized when the storage instance is created. That context is used for storage operations.
- **MemoryContext:** Use this context when you want to do some tests and you don't want your changes to be persisted.

#### Fetching data

```swift
let pedros: [Person] = try! db.fetch(Request<Person>().filteredWith("name", equalTo: "Pedro"))
let tasks: [Task] = try! db.fetch(Request<Task>())
let citiesByName: [City] = try! db.fetch(Request<City>().sortedWith("name", ascending: true))
let predicate: NSPredicate = NSPredicate(format: "id == %@", "AAAA")
let john: User? = try! db.fetch(Request<User>().filteredWith(predicate: predicate)).first
```

#### Remove/Insert/Update operations

Although `Context`s offer `insertion` and `deletion` methods that you can use it directly SugarRecords aims at using the `operation` method method provided by the storage for operations that imply modifications of the database models:

- **Context**: You can use it for fetching, inserting, deleting. Whatever you need to do with your data.
- **Save**: All the changes you apply to that context are in a memory state unless you call the `save()` method. That method will persist the changes to your store and propagate them across all the available contexts.

```swift
do {
  db.operation { (context, save) throws -> Void in
    // Do your operations here
    save()
  }
}
catch {
  // There was an error in the operation
}
```

##### New model
You can use the context `new()` method to initialize a model **without inserting it in the context**:

```swift
do {
  db.operation { (context, save) throws -> Void in
    let newTask: Track = try! context.new()
    newTask.name = "Make CoreData easier!"
    try! context.insert(newTask)
    save()
  }
}
catch {
  // There was an error in the operation
}
```
> In order to insert the model into the context you use the insert() method.

##### Creating a model
You can use the `create()` for initializing and inserting in the context in the same operation:

```swift
do {
  db.operation { (context, save) throws -> Void in
    let newTask: Track = try! context.create()
    newTask.name = "Make CoreData easier!"
    save()
  }
}
catch {
  // There was an error in the operation
}
```

##### Delete a model
In a similar way you can use the `remove()` method from the context passing the objects you want to remove from the database:

```swift
do {
  db.operation { (context, save) -> Void in
    let john: User? = try! context.request(User.self).filteredWith("id", equalTo: "1234").fetch().first
    if let john = john {
      try! context.remove([john])
      save()
    }
  }
}
catch {
  // There was an error in the operation
}
```

<br>
> This is the first approach of SugarRecord for the  interface. We'll improve it with the feedback you can report and according to the use of the framework. Do not hesitate to reach us with your proposals. Everything that has to be with making the use of CoreData/Realm easier, funnier, and enjoyable is welcome! :tada:

### RequestObservable

SugarRecord provides a component, `RequestObservable` that allows observing changes in the DataBase. It uses Realm notifications and CoreData `NSFetchedResultsController` under the hood.

**Observing**

```swift
class Presenter {
  var observable: RequestObservable<Track>!

  func setup() {
      let request: Request<Track> = Request<Track>().filteredWith("artist", equalTo: "pedro")
      self.observable = storage.instance.observable(request)
      self.observable.observe { changes in
        case .Initial(let objects):
          print("\(objects.count) objects in the database")
        case .Update(let deletions, let insertions, let modifications):
          print("\(deletions.count) deleted | \(insertions.count) inserted | \(modifications.count) modified")
        case .Error(let error):
          print("Something went wrong")
      }
  }
}
```
> **Retain**: RequestObservable must be retained during the observation lifecycle. When the `RequestObservable` instance gets released from memory it stops observing changes from your storage.

> **NOTE**: This was renamed from Observable -> RequestObservable so we are no longer stomping on the RxSwift Observable namespace.

**:warning: `RequestObservable` is not available for CoreData + OSX**

## Resources
- [Quick](https://github.com/quick/quick)
- [Nimble](https://github.com/quick/nimble)
- [CoreData and threads with GCD](http://www.cimgf.com/2011/05/04/core-data-and-threads-without-the-headache/)
- [Jazzy](https://github.com/realm/jazzy)
- [iCloud + CoreData (objc.io)](http://www.objc.io/issue-10/icloud-core-data.html)

## About

<img src="https://github.com/carambalabs/Foundation/blob/master/ASSETS/avatar_rounded.png?raw=true" width="70" />

This project is funded and maintained by [Caramba](http://caramba.in). We 💛 open source software!

Check out our other [open source projects](https://github.com/carambalabs/), read our [blog](http://blog.caramba.in) or say :wave: on twitter [@carambalabs](http://twitter.com/carambalabs).

## Contribute

Contributions are welcome :metal: We encourage developers like you to help us improve the projects we've shared with the community. Please see the [Contributing Guide](https://github.com/carambalabs/Foundation/blob/master/CONTRIBUTING.md) and the [Code of Conduct](https://github.com/carambalabs/Foundation/blob/master/CONDUCT.md).

## License
The MIT License (MIT)

Copyright (c) <2014> <Pedro Piñera>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
