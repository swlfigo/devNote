

# MTKView



## 概述

MTKView 封装了 Metal 的基本功能，隐藏了配置 Metal 的一些操作，省去了开发者配置的麻烦，开发者只需要将精力花费在图形渲染即数据准备上就可以。

在 iOS, iPadOS, Mac Catalyst, tvOS中，代码定义如下：

```swift
class MTKView : UIView 
```

在 macOS 中定义如下：

```swift
class MTKView : NSView
```



MTKView 使用 CAMetalLayer 来管理 Metal 可绘制对象，通过 MTLRenderPassDescriptor 对象可以将内容渲染到一个纹理中。

此外，通过 MTKView 可以创建深度和模板纹理以及抗锯齿所需的任何中间纹理。该视图使用 CAMetalLayer 来管理 Metal 可绘制对象。



## 指定绘制模式 

**(GPUImage3中RenderView内有设置,不设置会一卡一卡)**

MTKView 支持三种绘制模式：

- 定时更新：视图根据内部计时器重绘其内容。默认情况下使用这个绘制方式，初始化时需要将 isPaused 和 enableSetNeedsDisplay 都设置为 false。游戏和更新的动画内容常用这种模式。
- 绘制通知：当调用 setNeedsDisplay() 或者 当某些内容使其内容无效时，视图会重绘自身。在这种情况下，将 isPaused 和 enableSetNeedsDisplay 设置为 true。这种模式适用于具有更传统工作流程的应用程序，更新只会在数据更改时发生，而不会定期更新。
- 显式绘制：当显式调用 draw() 方法时，视图才会重绘其内容。这种模式下，需要将 isPaused 设置为 true 并将 enableSetNeedsDisplay 设置为 false。一般使用此模式来创建自定义工作流程。



## 绘制

无论使用哪一种绘图模式，当视图需要更新其内容时，在子类重写 draw(:) 方法时，它会调用 draw(:) 方法，否则在视图的委托上调用 draw(in:) 方法。

在绘图方法中，从视图中获取渲染通道描述符，此后的渲染结果都会存在渲染通道描述符中，最后渲染结果呈现在其关联的可绘制对象上。



## 从 MetalKit 视图中获取可绘制对象

CAMetalLayer 提供 MTKView 视图的绘图能力。在渲染器过程中，需要实现 MTKViewDelegate 协议来实现与 MetalKit 视图交互。调用 MetalKit 视图的 currentRenderPassDescriptor 属性可以获取当前帧的渲染通道描述符：

```swift

// BEGIN encoding your onscreen render pass.

// Obtain a render pass descriptor generated from the drawable's texture.

// (`currentRenderPassDescriptor` implicitly obtains the current drawable.)

// If there's a valid render pass descriptor, use it to render to the current drawable.

if let onscreenDescriptor = view.currentRenderPassDescriptor
```

当读取currentRenderPassDescriptor属性时，Core Animation 会隐式获取当前帧的可绘制对象并将其存储在 currentDrawable 属性中。后续深度、模板和抗锯齿纹理会被绘制到 currentDrawable 对象中。视图使用默认的存储和加载操作配置此渲染通道，在创建 MTLRenderCommandEncoder 之前，可以调整描述符。为了提升渲染性能，应该尽可能晚地获取可绘制对象，最好是在编码屏幕渲染通道之前。

## 关联可绘制对象

在渲染内容后，需要将可绘制对象的结果更新到视图中来显示。展示内容最方便的方法是调用命令缓冲区上的 present(:) 方法将视图与命令缓冲区关联起来，然后调用 commit() 方法将命令缓冲区提交给 GPU：

```swift
if let onscreenDescriptor = view.currentRenderPassDescriptor,

let onscreenCommandEncoder = onscreenCommandBuffer.makeRenderCommandEncoder(descriptor: onscreenDescriptor) {

    /* Set render state and resources.

       ...

     */

    /* Issue draw calls.

       ...

     */

    onscreenCommandEncoder.endEncoding()

    // END encoding your onscreen render pass.

    

    // Register the drawable's presentation.

    if let currentDrawable = view.currentDrawable {

        onscreenCommandBuffer.present(currentDrawable)

    }

}


// Finalize your onscreen CPU work and commit the command buffer to a GPU.

onscreenCommandBuffer.commit()
```



当命令队列调度可执行的命令缓冲区时，可绘制对象会在该命令缓冲区中遍历其自身的所有渲染或写入请求。在命令完成执行之前，操作系统不会在屏幕上显示可绘制对象。通过这种调度方式，保证了命令队列调度此命令缓冲区之后才显示渲染结果。





## Reference

https://juejin.cn/post/7030741059999121439