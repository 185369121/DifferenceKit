<h1 align="center">DifferenceKit</h1>
<H4 align="center">
A fast and flexible O(n) difference algorithm framework for Swift collection.</br>
The algorithm is optimized based on the Paul Heckel's algorithm.
</H4>

<p align="center">
<a href="https://developer.apple.com/swift"><img alt="Swift4" src="https://img.shields.io/badge/language-swift4-orange.svg?style=flat"/></a>
<a href="https://developer.apple.com/swift/"><img alt="Platform" src="https://img.shields.io/badge/platform-iOS%20%7C%20tvOS-green.svg"/></a>
<a href="https://github.com/ra1028/DifferenceKit/blob/master/LICENSE"><img alt="Lincense" src="http://img.shields.io/badge/license-MIT-000000.svg?style=flat"/></a>
</br>
<a href="https://travis-ci.org/ra1028/DifferenceKit"><img alt="Build Status" src="https://travis-ci.org/ra1028/DifferenceKit.svg?branch=master"/></a>
<a href="https://cocoapods.org/pods/DifferenceKit"><img alt="CocoaPods" src="https://img.shields.io/cocoapods/v/DifferenceKit.svg"/></a>
<a href="https://github.com/Carthage/Carthage"><img alt="Carthage" src="https://img.shields.io/badge/Carthage-compatible-yellow.svg?style=flat"/></a>
</p>

---

## Features
✅ Automate to calculate commands for batch-updates of UITableView and UICollectionView  
✅ **O(n)** difference algorithm optimized for performance in Swift  
✅ Supports both linear and sectioned collection  
✅ Supports calculating differences with best effort even if elements or section contains duplicates  
✅ Supports **all commands** for animating UI batch-updates including section reloads  

---

<p align="center">
<img src="https://raw.githubusercontent.com/ra1028/DifferenceKit/master/assets/sample.gif" width="250">
</p>

---

## Algorithm
The algorithm used in DifferenceKit is optimized based on the Paul Heckel's algorithm.  
See also his paper ["A technique for isolating differences between files"](https://dl.acm.org/citation.cfm?id=359467) released in 1978.  
[RxDataSources](https://github.com/RxSwiftCommunity/RxDataSources) and [IGListKit](https://github.com/Instagram/IGListKit) are also implemented based on his algorithm.  
This allows all types of differences to be computed in linear time **O(n)**.  

However, in `performBatchUpdates` of UITableView and UICollectionView, there are combinations of commands that cause crash when applied simultaneously.  
To solve this problem, DifferenceKit takes an approach of split the set of differences at the minimal stages that can be perform batch-updates with no crashes.

Implementation is [here](https://github.com/ra1028/DifferenceKit/blob/master/Sources/Algorithm.swift).

---

## Documentation
See docs in [GitHub Pages](https://ra1028.github.io/DifferenceKit/).  
Documentation is generated by [jazzy](https://github.com/realm/jazzy).

---

## Getting Started
- [Example app](https://github.com/ra1028/DifferenceKit/blob/master/Examples)
- [Playground](https://github.com/ra1028/DifferenceKit/blob/master/DifferenceKit.playground/Contents.swift)

### Example codes
The type of the element that to take the differences must be conform to the `Differentiable` protocol:
```swift
struct User: Differentiable {
    let id: Int
    let name: String

    var differenceIdentifier: Int {
        return id
    }

    func isUpdated(from source: User) -> Bool {
        return name != source.name
    }
}
```

In the case of definition above, `id` uniquely identifies the element and get to know the user updated by comparing `name` of the elements in source and target.

There are default implementations of `Differentiable` for the types that conformed to `Equatable` or `Hashable`：
```swift
extension String: Differentiable {}
```

You can calculate the differences by creating `StagedChangeset` from two collections of elements conforming to `Differentiable`:
```swift
let source = [
    User(id: 0, name: "Vincent"),
    User(id: 1, name: "Jules")
]
let target = [
    User(id: 1, name: "Jules"),
    User(id: 0, name: "Vincent"),
    User(id: 2, name: "Butch")
]

let changeset = StagedChangeset(source: source, target: target)
```

If you want to include multiple types conformed to `Differentiable` in the collection, use `AnyDifferentiable`:
```swift
let source = [
    AnyDifferentiable("A"),
    AnyDifferentiable(User(id: 0, name: "Vincent"))
]
```

In the case of sectioned collection, the section itself must have a unique identifier and be able to compare whether there is an update.  
So each section must conform to `DifferentiableSection` protocol, but in most cases you can use `Section` that general type conformed to it.  
`Section` requires a model conforming to `Differentiable` for differentiate from other sections:
```swift
enum Model: Differentiable {
    case a, b, c
}

let source: [Section<Model, String>] = [
    Section(model: .a, elements: ["A", "B"]),
    Section(model: .b, elements: ["C"])
]
let target: [Section<Model, String>] = [
    Section(model: .c, elements: ["D", "E"]),
    Section(model: .a, elements: ["A"]),
    Section(model: .b, elements: ["B", "C"])
]

let changeset = StagedChangeset(source: source, target: target)
```

You can incremental updates UITableView and UICollectionView using the created `StagedChangeset`.  
**Don't forget** to update the data referenced by the dataSource in `setData` closure, as the differences is applied in stages:
```swift
tableView.reload(using: changeset, with: .fade) { data in
    dataSource.data = data
}
```

Batch-updates using too large amount of differences may adversely affect to performance.  
Returning `true` with `interrupt` closure then falls back to `reloadDate`:
```swift
collectionView.reload(using: changeset, interrupt: { $0.changeCount > 100 }) { data in
    dataSource.data = data
}
```

---

## Comparison with Other Frameworks
Made a fair comparison as much as possible in features and performance with other **popular** and **awesome** frameworks.  
The frameworks and its version that compared is below.  

- [DifferenceKit](https://github.com/ra1028/DifferenceKit) - 0.1.0
- [RxDataSources](https://github.com/RxSwiftCommunity/RxDataSources) ([Differentiator](https://github.com/RxSwiftCommunity/RxDataSources/tree/master/Sources/Differentiator)) - 3.0.2
- [IGListKit](https://github.com/Instagram/IGListKit) - 3.4.0
- [ListDiff](https://github.com/lxcid/ListDiff) - 0.1.0
- [DeepDiff](https://github.com/onmyway133/DeepDiff) - 1.2.0
- [Differ](https://github.com/tonyarnold/Differ) ([Diff.swift](https://github.com/wokalski/Diff.swift)) - 1.2.3
- [Dwifft](https://github.com/jflinter/Dwifft) - 0.8

### Features comparison
#### - Supported collection
|             |Linear|Sectioned|Duplicate Element/Section|
|:------------|:----:|:-------:|:-----------------------:|
|DifferenceKit|✅    |✅       |✅                        |
|RxDataSources|❌    |✅       |❌                        |
|IGListKit    |✅    |❌       |✅                        |
|ListDiff     |✅    |❌       |✅                        |
|DeepDiff     |✅    |❌       |✅                        |
|Differ       |✅    |✅       |✅                        |
|Dwifft       |✅    |✅       |✅                        |

`Linear` means 1-dimensional collection.  
`Sectioned` means 1-dimensional collection.  

#### - Supported element differences
|             |Delete|Insert|Move|Reload  |
|:------------|:----:|:----:|:--:|:------:|
|DifferenceKit|✅    |✅     |✅  |✅      |
|RxDataSources|✅    |✅     |✅  |✅      |
|IGListKit    |✅    |✅     |✅  |✅      |
|ListDiff     |✅    |✅     |✅  |✅      |
|DeepDiff     |✅    |✅     |✅  |✅ / ❌ |
|Differ       |✅    |✅     |✅  |❌      |
|Dwifft       |✅    |✅     |❌  |❌      |

#### - Supported section differences
|             |Delete|Insert|Move|Reload|
|:------------|:----:|:----:|:--:|:----:|
|DifferenceKit|✅    |✅     |✅  |✅    |
|RxDataSources|✅    |✅     |✅  |❌    |
|IGListKit    |❌    |❌     |❌  |❌    |
|ListDiff     |❌    |❌     |❌  |❌    |
|DeepDiff     |❌    |❌     |❌  |❌    |
|Differ       |✅    |✅     |✅  |❌    |
|Dwifft       |✅    |✅     |❌  |❌    |

### Performance comparison
Performance was measured using `XCTestCase.measure` with `-O -whole-module-optimization`.  
Use `Foundation.UUID` as an element.  

*DeepDiff may had increased the processing speed by misuse of Hashable in algorithm.*

#### - From 5,000 elements to 500 deleted and 500 inserted
|             |Time(second)|
|:------------|:-----------|
|DifferenceKit|0.00425     |
|RxDataSources|0.00784     |
|IGListKit    |0.0412      |
|ListDiff     |0.0388      |
|DeepDiff     |0.015       |
|Differ       |0.326       |
|Dwifft       |33.6        |

#### - From 10,000 elements to 1,000 deleted and 1,000 inserted
|             |Time(second)|
|:------------|:-----------|
|DifferenceKit|0.0079      |
|RxDataSources|0.0143      |
|IGListKit    |0.0891      |
|ListDiff     |0.0802      |
|DeepDiff     |0.030       |
|Differ       |1.345       |
|Dwifft       |❌          |

#### - From 100,000 elements to 10,000 deleted and 10,000 inserted
|             |Time(second)|
|:------------|:-----------|
|DifferenceKit|0.098       |
|RxDataSources|0.179       |
|IGListKit    |1.329       |
|ListDiff     |1.026       |
|DeepDiff     |0.334       |
|Differ       |❌          |
|Dwifft       |❌          |

---

## Requirements
- Swift4.1+
- iOS 10.0+
- tvOS 10.0+

---

## Installation

### [CocoaPods](https://cocoapods.org/)
Add the following to your `Podfile`:
```ruby
use_frameworks!

target 'TargetName' do
  pod 'DifferenceKit'
end
```
To use only algorithm without extensions for UI, add the following:
```ruby
use_frameworks!

target 'TargetName' do
  pod 'DifferenceKit/Core'
end
```
And run
```sh
pod install
```

### [Carthage](https://github.com/Carthage/Carthage)
Add the following to your `Cartfile`:
```ruby
github "ra1028/DifferenceKit"
```
And run
```sh
carthage update
```

---

## Contribution
Welcome to fork and submit pull requests.  
Before submitting pull request, please ensure you have passed the included tests.  
If your pull request including new function, please write test cases for it.  

---

## License
DifferenceKit is released under the [MIT License](https://github.com/ra1028/DifferenceKit/blob/master/LICENSE).