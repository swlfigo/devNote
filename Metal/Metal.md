# Metal

## 概念

Metal的简要流程图如下：

![img](https://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/webp.jpg)



`MTLCommandQueue`由device创建，是整个app绘制的队列，而command buffer存放每次渲染的指令,即包含了每次渲染所需要的信息，直到指令被提交到GPU执行。Command queue用于创建和组织MTLCommandBuffer，其内部存在着多个command buffer，并且保证指令（command buffer）有序地发送到GPU。

*render pass*是指绘制一组渲染目标（纹理）的命令序列。我们用`MTLRenderPassDescriptor`来配置*render pass*。

编码器`MTLCommandEncoder`，将我们描述的高级指令，编码转换成GPU可以理解的低级指令(GPU commands)，写入command buffer中，我们用到的`MTLRenderCommandEncoder`是用于图形渲染任务的编码器。

*render pipeline*是指处理绘制命令，并把数据写入到*render pass*的渲染目标中，着色器方法需要添加到*render pipeline*中。*render pipeline*对应的对象是`MTLRenderPipelineState`，我们需要用一个`MTLRenderPipelineDescriptor`对象来配置它。

当渲染好纹理后并不会自动更新到屏幕上，**在Metal中，通过显示*drawable*对象来显示其纹理。**



在Metal中支持并行运算，在这里我们可以从两个部分来实现GPU的并行。从下图可知，在Metal中Encoder和CommandBuffer这两个操作层级都是支持并行的。我们可以在多个线程中提交编码命令缓冲区(CommandBuffer)和命令编码器(CommandEncoder)。

<img src="https://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/15432216340130.jpg" alt="img" style="zoom:50%;" />

## 渲染流程

首先我们从CommandQueue中取出一个命令缓冲区(CommandBuffer)，然后编写数个同步或异步的渲染命令编码器(RenderEncoder)完成渲染操作。

[CommandQueue commandBuffer] -> RenderEncoder -> Draw-Call…Draw-Call…Draw-Call -> [RenderEncoder endEncoding] -> [CommandBuffer Commit]



注:其实CommandBuffer就是类似一个一帧绘制任务，任务中需要配置顶点，片段函数，colorattachment，纹理之类信息。



## 顶点坐标

关于传入的顶点坐标顺序，答案来自 claude

```swift
/*
在 Metal 中使用 triangleStrip 绘制图片时，顶点传入顺序非常重要，它决定了三角形的连接方式以及最终渲染结果。直接传入左上，右上，右下，左下顺序通常不会得到预期的结果。

triangleStrip 的工作原理是，第一个三角形由前三个顶点构成，后续的每个三角形都与前一个三角形共享一条边，并由前一个三角形的最后两个顶点和当前顶点构成。

为了正确绘制一个矩形（图片通常被视为矩形），你需要以逆时针或顺时针的顺序提供顶点，并且确保三角形带状图覆盖整个矩形区域。以下两种顶点顺序都可以绘制一个矩形：

左上，右上，左下，右下: 这种顺序下，第一个三角形由左上、右上、左下构成，第二个三角形由右上、左下、右下构成。

左上，左下，右上，右下: 这种顺序下，第一个三角形由左上、左下、右上构成，第二个三角形由左下、右上、右下构成。

以下是一个简单的示例，展示了如何使用 triangleStrip 绘制一个矩形：

关键点：

triangleStrip 绘制需要至少三个顶点。
矩形绘制需要四个顶点，按特定顺序排列。
顶点顺序决定了三角形的绘制方向，进而影响背面剔除等渲染效果。
纹理坐标 (texCoord) 的设置与顶点顺序对应，确保纹理正确映射到矩形上。
如果你使用左上，右上，右下，左下 的顺序，绘制的两个三角形会重叠，导致渲染结果不正确。 因此，请务必按照上面提到的两种顺序之一来组织你的顶点数据。
*/

import MetalKit

struct Vertex {
    var position: SIMD2<Float>
    var texCoord: SIMD2<Float>
}

func createVertices(width: Float, height: Float) -> [Vertex] {
    let vertices: [Vertex] = [
        // 左上
        Vertex(position: [-width/2, height/2], texCoord: [0, 0]),
        // 右上
        Vertex(position: [width/2, height/2], texCoord: [1, 0]),
        // 左下
        Vertex(position: [-width/2, -height/2], texCoord: [0, 1]),
        // 右下
        Vertex(position: [width/2, -height/2], texCoord: [1, 1]),
    ]
    return vertices
}

// 在渲染编码器中绘制
renderEncoder.setVertexBytes(vertices, length: vertices.count * MemoryLayout<Vertex>.stride, index: 0)
renderEncoder.drawPrimitives(type: .triangleStrip, vertexStart: 0, vertexCount: 4)
```



所以，顶点坐标顺序传入需要按照逆时针或者顺时针传入，如果平面2D这种，以顺时针为例，需要顺序为  ***左上，右上，左下，右下***  , 纹理坐标也应该如此



## Shader

shader中传入的顶点坐标，定义数据类型偏移量要对齐

```
内存对齐的基本概念：

在Metal中，结构体和数据类型都需要遵循特定的内存对齐规则，这对于数据在CPU和GPU之间的高效传输非常重要。
内存对齐会影响数据在内存中的布局和访问效率 。
Metal中的主要对齐规则：

基本类型对齐：
float：4字节对齐
float4：16字节对齐
float3：16字节对齐（这点需要特别注意）
float4x4：16字节对齐
结构体对齐：
结构体的对齐遵循C/C++的标准对齐算法 
结构体的总体对齐值取决于其成员中最大的对齐要求
packed类型：
Metal shader语言中的packed类型（如packed_float4）的对齐要求较为宽松
packed_float4的对齐要求是4字节（与单个float相同）
示例：

swift
// Swift端
struct Vertex {
    var position: SIMD4<Float>  // 16字节对齐
    var color: SIMD4<Float>     // 16字节对齐
}

// Metal Shader端
struct Vertex {
    float4 position [[position]]; // 16字节对齐
    float4 color;                 // 16字节对齐
};

```



如上 **float4 position [[position]]; // 16字节对齐** ，在shader中，需要标注postion，告诉shader返回的这个字段属性是关于顶点的坐标

详细可以查看Demo

[Metal绘制Demo](https://github.com/swlfigo/someDemo/tree/master/MetalTriangle)





