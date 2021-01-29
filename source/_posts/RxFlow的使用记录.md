---
title: RxFlow使用记录
---
1、在官方demo中的可以看到是在AppDelegate中创建了一个```FlowCoordinator```，一个全局的```Coordinator```, 通过这个```Coordinator```传入的```AppFlow```是最顶层的，在目前的使用中发现，这个最顶层的```AppFlow```只会响应```AppStepper```中的事件。如果在下一层中传入的```presentable```是一个```Controller```，对应的```Stepper```是一个```ViewModel```或是这个```Controller```自身，则在```AppFlow```中是收不到这个```Stepper```的事件的，具体的解决方法是创建一个对应的```Flow```去处理事件。如果不处理可以转发到```ParentFlow```既```AppFlow```中去处理。

2、在项目中使用到一个```WrapViewController```去包裹一个```controller```, 如果```presentable```传入这个```controller```而不是外层的```WrapViewController```，则在```popViewController```时，这个```controller```不会释放。
```swift
let controller = UIViewContrller()
let wrapController = WrapViewController(controller)
_rootViewController.pushViewController(wrapController)
FlowContributors.one(flowContributor: .contribute(withNextPresentable: controller, withNextStepper: stepper))
```
此时的```controller```, 在```popViewController```后是不会被释放掉的，只有传入```wrapController```（ ```.contribute(withNextPresentable: controller, withNextStepper: stepper)```），才可以释放。
