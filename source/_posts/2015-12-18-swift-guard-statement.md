title: Swift的guard语句
categories: iOS
date: 2015-12-18 08:15:33
tags: Swift
---

Swift 2引入了`guard`关键字，特性可以概括为以下两点：

- 仅关注需要满足的条件，一旦条件不满足，快速返回（fast return）；
- 对于optional的变量，使用`guard let`解绑后，在包含`guard`的作用域内（guard语句已结束），该解绑后的值仍然可用；

<!-- more -->

## 1. 失败快速返回

使用`if`语句，如果关注点是需要满足的条件，则：

	if firstName != "" && lastName != "" {
		// do stuff with firstName and lastName
	} else {
		// fast return
	}

> 如果条件满足后要执行的逻辑很长，则当条件不满足时，无法快速返回；

如果失败快速返回，则关注点是不满足的条件，如果不满足的条件很多，则这是不合理的：

	if firstName == "" || lastName == "" {
		// fast return
	} else {
		// do stuff with firstName and lastName
	}

使用`guard`语句，关注需要满足的条件，不满足快速返回：

	guard firstName != "" && lastName != "" else {
		// fast return
	}
	// do stuff with firstName and lastName

## 2. optional解绑

使用`if let`解绑optional变量，在if作用域外，解绑的值是不可用的：

	func checkAge(ageOfUser:Int?) {
		if let age = ageOfUser where age > 17 {
			// do stuff with age
		} 
		// cannot use age anymore
		//	print("you age is: \(age), continue.")
	}

使用`guard let`解绑optional变量后，在包含guard的作用域内，解绑后的值是可用的：

	func checkAge(ageOfUser: Int?) {
		guard let age = ageOfUser where age > 17 else {
			// fast return
			return
		}
		// do stuff with age
		print("you age is: \(age), continue.")
	}

鉴于`guard`语句带来的好处，如果可以，尽量使用`guard`语句。

### 参考

- [Statements](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/Statements.html)
- [The guard keyword in Swift 2: early returns made easy](https://www.hackingwithswift.com/new-syntax-swift-2-guard)

