# AERecord
**Super awesome Core Data wrapper for iOS written in Swift**


Why do we need yet another one Core Data wrapper? You tell me!

>Inspired by many different (spoiler alert) magical solutions,
I needed something which combines complexity and functionality just the way I want.
All of that boilerplate for setting up of Core Data stack can be packed in 
one reusable and customizible line of code, and it should be.
Passing the right `NSManagedObjectContext` all accross the project, 
worrying about threads and stuff, shouldn't really be my concern in every single project.
And what about that similar `NSFetchRequest` code chunks for querying or creating of data? Boring!
Finally when it comes to connecting your data with the tableView, 
we know that the best approach is to use `NSFetchedResultsController`,
but `CoreDataTableViewController` from [Stanford's CS193p](http://www.stanford.edu/class/cs193p/cgi-bin/drupal/downloads-2013-winter) is the best thing ever,
and should be used everywhere.
I liked it so much that I made `CoreDataCollectionViewController` in the same fashion.
So, `AERecord` should solve all of these for me, I hope you will like it too.


**AERecord** is a [minion](http://tadija.net/public/minion.png) which consists of these classes / extensions:  

Class | Description
------------ | -------------
`AERecord` | main public class
`AEStack` | private class which takes care of stack
`NSManagedObject extension` | super easy data querying
`CoreDataTableViewController` | Core Data driven UITableViewController
`CoreDataCollectionViewController` | Core Data driven UICollectionViewController


## Features
- Create default or custom Core Data stack easily accessible from everywhere
- Have main and background contexts, always in sync, but don't worry about it
- Create, delete or fetch data in many ways with one liners
- Connect UI (tableView or collectionView) with Core Data, and just manage the data
- That's all folks (for now)


## Index
- [Examples](#examples)
  - [About AERecordExample project](#about-aerecordexample-project)
  - [Create Core Data stack](#create-core-data-stack)
  - [Manage contexts](#manage-contexts)
  - [Easy querying](#easy-querying)
  	- [Creating](#creating)
  	- [Deleting](#deleting)
  	- [Finding first](#finding-first)
  	- [Finding all](#finding-all)
  	- [Custom fetch requests](#custom-fetch-requests)
  - [Use Core Data with tableView](#use-core-data-with-tableview)
  - [Use Core Data with collectionView](#use-core-data-with-collectionview)
- [API](#api)
  - [AERecord](#aerecord-class)
  - [NSManagedObject extension](#nsmanagedobject-extension)
  - [CoreDataTableViewController](#coredatatableviewcontroller)
  - [CoreDataCollectionViewController](#coredatacollectionviewcontroller)
- [Requirements](#requirements)
- [Installation](#installation)
- [License](#license)


## Examples

### About AERecordExample project
This project is made of default Master-Detail Application template with Core Data enabled,
but modified to show off some of the `AERecord` features such as creating of Core Data stack,
using data driven tableView and collectionView, along with few simple querying.  
I mean, just compare it with the default template and think about that.

### Create Core Data stack
Almost everything in `AERecord` is made with optional parameters (which have defaults if you don't specify anything).
So you can create CoreData stack like this:

```swift
AERecord.setupCoreDataStack()
```

or like this:

```swift
let myModel: NSManagedObjectModel = ...
let myStoreType = NSInMemoryStoreType
let myConfiguration = ...
let myStoreURL = ...
let myOptions = [NSMigratePersistentStoresAutomaticallyOption : true]
AERecord.setupCoreDataStack(managedObjectModel: myModel, storeType: myStoreType, configuration: myConfiguration, storeURL: myStoreURL, options: myOptions)
```

or any combination of these.

### Manage contexts

```swift
AERecord.mainContext // get NSManagedObjectContext for main thread
AERecord.backgroundContext // get NSManagedObjectContext for background thread
AERecord.defaultContext // get NSManagedObjectContext for current thread

AERecord.saveContext() // save default context
AERecord.saveContext(context: myContext) // save specific context

AERecord.saveContextAndWait() // save default context and wait for save to finish
AERecord.saveContextAndWait(context: myContext) // save specific context and wait for save to finish
```

### Easy querying
All queries are called on NSManagedObject (or it's subclass), 
and defaultContext is used if you don't specify any (all examples below are using defaultContext).
All finders have optional parameter for `NSSortDescriptor` which is not used in these examples.

#### Creating
```swift
NSManagedObject.create() // create new object

let attributes = ...
NSManagedObject.createWithAttributes(attributes) // create new object and sets it's attributes

NSManagedObject.firstOrCreateWithAttribute("city", value: "Belgrade") // get existing object or create new (if there's not existing object) with given attribute name and value
```

#### Deleting
```swift
let managedObject = ...
managedObject.delete() // delete object (call on instance)

NSManagedObject.deleteAll() // delete all objects

NSManagedObject.deleteAllWithAttribute("fat", value: true) // delete all objects with given attribute name and value

let predicate = ...
NSManagedObject.deleteAllWithPredicate(predicate) // delete all objects with given predicate
```

#### Finding first
```swift
NSManagedObject.first() // get first object

let predicate = ...
NSManagedObject.firstWithPredicate(predicate) // get first object with predicate

NSManagedObject.firstWithAttribute("bike", value: "KTM") // get first object with given attribute name and value

NSManagedObject.firstOrderedByAttribute("speed", ascending: false) // get first object ordered by given attribute name
```

#### Finding all
```swift
NSManagedObject.all() // get all objects

let predicate = ...
NSManagedObject.allWithPredicate(predicate) // get all objects with predicate

NSManagedObject.allWithAttribute("year", value: 1984) // get all objects with given attribute name and value
```

#### Custom fetch requests
If you need to execute custom `NSFetchRequest` you can use this to create some request, 
tweak it as you wish and finally execute.

```swift
let predicate = ...
let sortDescriptors = ...
NSManagedObject.createFetchRequest(predicate: predicate, sortDescriptors: sortDescriptors) // get request for entity

let request = ...
NSManagedObject.executeFetchRequest(request) // execute any request and get array of objects
```

### Use Core Data with tableView
`CoreDataTableViewController` mostly just copies the code from `NSFetchedResultsController`
documentation page into a subclass of UITableViewController.

Just subclass it and set it's `fetchedResultsController` property.

After that you'll only have to implement `tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell` and `fetchedResultsController` will take care of other required data source methods.
It will also update `UITableView` whenever the underlying data changes (insert, delete, update, move).

#### CoreDataTableViewController Example
```swift
import UIKit
import CoreData

class MyTableViewController: CoreDataTableViewController {

	override func viewDidLoad() {
	    super.viewDidLoad()
	    
	    // setup fetchedResultsController property
	    refreshFetchedResultsController()
	}

	func refreshFetchedResultsController() {
	    let sortDescriptors = [NSSortDescriptor(key: "timeStamp", ascending: true)]
	    let request = Event.createFetchRequest(sortDescriptors: sortDescriptors)
	    fetchedResultsController = NSFetchedResultsController(fetchRequest: request, managedObjectContext: AERecord.defaultContext, sectionNameKeyPath: nil, cacheName: nil)
	}

	override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
	    let cell = tableView.dequeueReusableCellWithIdentifier("Cell", forIndexPath: indexPath) as UITableViewCell
	    if let frc = fetchedResultsController {
	        if let object = frc.objectAtIndexPath(indexPath) as? Event {
	            cell.textLabel.text = object.timeStamp.description
	        }
	    }
	    return cell
	}

}
```

### Use Core Data with collectionView
Same as with the tableView.


## API

### AERecord class
`class AERecord`

Property | Description
------------ | -------------
`class var defaultContext: NSManagedObjectContext` | context for current thread
`class var mainContext: NSManagedObjectContext` | context for main thread
`class var backgroundContext: NSManagedObjectContext` | context for background thread
`class var persistentStoreCoordinator: NSPersistentStoreCoordinator?` | persistent store coordinator

Setup Stack | Description
------------ | -------------
`class func setupCoreDataStack(managedObjectModel: NSManagedObjectModel = AEStack.defaultModel, storeType: String = NSSQLiteStoreType, configuration: String? = nil, storeURL: NSURL = AEStack.defaultURL, options: [NSObject : AnyObject]? = nil) -> NSError?` | You need to do this only once. `AEStack.defaultModel` is `NSManagedObjectModel.mergedModelFromBundles(nil)!` and `AEStack.defaultURL` is `bundleIdentifier + ".sqlite"` in `applicationDocumentsDirectory`.

Save Context | Description
------------ | -------------
`class func saveContext(context: NSManagedObjectContext? = nil)` | save context (if not specified `defaultContext` is used)
`class func saveContextAndWait(context: NSManagedObjectContext? = nil)` | save context and wait for save to finish (if not specified `defaultContext` is used)


### NSManagedObject extension
`extension NSManagedObject`

Property | Description
------------ | -------------
`class var entityName: String` | entity name

Creating | Description
------------ | -------------
`class func create(context: NSManagedObjectContext = AERecord.defaultContext) -> Self` | create new object
`class func createWithAttributes(attributes: [NSObject : AnyObject], context: NSManagedObjectContext = AERecord.defaultContext) -> Self` | create new object and sets it's attributes
`class func firstOrCreateWithAttribute(attribute: String, value: AnyObject, context: NSManagedObjectContext = AERecord.defaultContext) -> NSManagedObject` | get existing object or create new (if there's not existing object) with given attribute name and value

Deleting | Description
------------ | -------------
`func delete(context: NSManagedObjectContext = AERecord.defaultContext)` | delete object
`class func deleteAll(context: NSManagedObjectContext = AERecord.defaultContext)` | delete all objects
`class func deleteAllWithPredicate(predicate: NSPredicate, context: NSManagedObjectContext = AERecord.defaultContext)` | delete all objects with given attribute name and value
`class func deleteAllWithAttribute(attribute: String, value: AnyObject, context: NSManagedObjectContext = AERecord.defaultContext)` | delete all objects with given predicate

Finding first | Description
------------ | -------------
`class func first(sortDescriptors: [NSSortDescriptor]? = nil, context: NSManagedObjectContext = AERecord.defaultContext) -> NSManagedObject?` | get first object
`class func firstWithPredicate(predicate: NSPredicate, sortDescriptors: [NSSortDescriptor]? = nil, context: NSManagedObjectContext = AERecord.defaultContext) -> NSManagedObject?` | get first object with predicate
`class func firstWithAttribute(attribute: String, value: AnyObject, sortDescriptors: [NSSortDescriptor]? = nil, context: NSManagedObjectContext = AERecord.defaultContext) -> NSManagedObject?` | get first object with given attribute name and value
`class func firstOrderedByAttribute(name: String, ascending: Bool = true, context: NSManagedObjectContext = AERecord.defaultContext) -> NSManagedObject?` | get first object ordered by given attribute name

Finding all | Description
------------ | -------------
`class func all(sortDescriptors: [NSSortDescriptor]? = nil, context: NSManagedObjectContext = AERecord.defaultContext) -> [NSManagedObject]?` | get all objects
`class func allWithPredicate(predicate: NSPredicate, sortDescriptors: [NSSortDescriptor]? = nil, context: NSManagedObjectContext = AERecord.defaultContext) -> [NSManagedObject]?` | get all objects with predicate
`class func allWithAttribute(attribute: String, value: AnyObject, sortDescriptors: [NSSortDescriptor]? = nil, context: NSManagedObjectContext = AERecord.defaultContext) -> [NSManagedObject]?` | get all objects with given attribute name and value

Custom fetch requests | Description
------------ | -------------
`class func createFetchRequest(predicate: NSPredicate? = nil, sortDescriptors: [NSSortDescriptor]? = nil) -> NSFetchRequest` | get request for entity
`class func executeFetchRequest(request: NSFetchRequest, context: NSManagedObjectContext = AERecord.defaultContext) -> [NSManagedObject]` | execute any request and get array of objects


### CoreDataTableViewController
`class CoreDataTableViewController`

Property | Description
------------ | -------------
`var fetchedResultsController: NSFetchedResultsController?` | you must set this property
`var suspendAutomaticTrackingOfChangesInManagedObjectContext: Bool` | may be used when moving rows, explained more in comments in code

Fetching | Description
------------ | -------------
`func performFetch()` | you never have to call this directly, explained more in comments in code


### CoreDataCollectionViewController
`class CoreDataCollectionViewController`

Property | Description
------------ | -------------
`var fetchedResultsController: NSFetchedResultsController?` | you must set this property
`var suspendAutomaticTrackingOfChangesInManagedObjectContext: Bool` | may be used when moving cells, explained more in comments in code

Fetching | Description
------------ | -------------
`func performFetch()` | you never have to call this directly, explained more in comments in code


## Requirements
- Xcode 6.1+
- iOS 7.0+
- AERecord doesn't require any additional libraries for it to work.


## Installation
Just drag AERecord.swift into your project and start using it.


## License
AERecord is released under the MIT license. See [LICENSE](LICENSE) for details.
