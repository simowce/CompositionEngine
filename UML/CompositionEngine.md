CompositionEngine 在 R 上经历了一次比较大的重构，真是这次重构让我决定要为 CompositionEngine 写点东西。因为在经历了这次重构以后，所有跟合成相关的内容全部都被打包进 CompositionEngine 里，这样其实非常便于理解和分析。我们可以先看一下整个重构的过程：

1. 剥离出一个 CompositionEngine 的接口类和实现类，就只有两个成员函数——构造函数和析构函数：
   // `createCompositionEngine()` 这个定义在类外部的方法应该怎么处理？

   ```plantuml
   @startuml
   namespace compositionengine {
       interface CompositionEngine {
           + CompositionEngine()
           + ~CompositionEngine()
       }
   }
   namespace compositionengine::impl {
       class CompositionEngine {
           + CompositionEngine()
           + ~CompositionEngine()
       }
       compositionengine.CompositionEngine <|-- CompositionEngine
   }
   @enduml
   ```
2. 将 HWComposer 并入
   ```plantuml
   @startuml
   namespace compositionengine {
       interface CompositionEngine {
           + CompositionEngine()
           + ~CompositionEngine()

           + HWComposer& getHwComposer()
           + void setHwComposer()
       }
   }
   namespace compositionengine::impl {
       class CompositionEngine {
           - mHwComposer
           + CompositionEngine()
           + ~CompositionEngine()

           + HWComposer& getHwComposer()
           + void setHwComposer()
       }
       compositionengine.CompositionEngine <|-- CompositionEngine
   }
   @enduml
   ```