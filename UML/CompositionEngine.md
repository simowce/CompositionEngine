CompositionEngine 在 R 上经历了一次比较大的重构，真是这次重构让我决定要为 CompositionEngine 写点东西。因为在经历了这次重构以后，所有跟合成相关的内容全部都被打包进 CompositionEngine 里，这样其实非常便于理解和分析。我们可以先看一下整个重构的过程：

1. 剥离出一个 CompositionEngine 的接口类和实现类，就只有两个成员函数——构造函数和析构函数：
   **注意： `createCompositionEngine()` 是一个类的非成员函数，我这里将其处理为静态方法**

   ```plantuml
   @startuml
      namespace android {
        namespace compositionengine {
        interface CompositionEngine {
            {abstract} ~CompositionEngine()
        }
        namespace impl {
        class CompositionEngine {
            + CompositionEngine()
            + ~CompositionEngine()
            {static} createCompositionEngine()
        }
        android.compositionengine.CompositionEngine <|.. CompositionEngine
       }
   }
   @enduml
   ```
2. 将 HWComposer 的所有权移入 CompositionEngine（在 SurfaceFlinger 只能通过 CompositionEngine 来获取 HWComposer）

   ```plantuml
   @startuml
   namespace android {
   namespace compositionengine {
       interface CompositionEngine {
           {abstract} ~CompositionEngine()
           
           {abstract} getHwComposer()
           {abstract} setHwComposer()
       }
       namespace impl {
       class CompositionEngine {
           - std::unique_ptr<HWComposer> mHwComposer
           + CompositionEngine()
           + ~CompositionEngine()

           + getHwComposer()
           + setHwComposer()
           {static} createCompositionEngine()
       }
       android.compositionengine.CompositionEngine <|.. CompositionEngine
       }
   }

   interface HWComposer {}

   namespace impl {
       class HWComposer {}
       android.HWComposer <|.. HWComposer
   }
   }
   @enduml
   ```

3. 将 RenderEngine 的所有权转移到 CompositionEngine（在 SurfaceFlinger 里，只能通过 CompositionEngine 来获取 RenderEngine）

    ```plantuml
    @startuml
    namespace android {
        namespace compositionengine {
            interface CompositionEngine {
                {abstract} ~CompositionEngine()
                
                {abstract} getHwComposer()
                {abstract} setHwComposer()

                {abstract} getRenderEngine()
                {abstract} setRenderEngine()
            }
            namespace impl {
                class CompositionEngine {
                    - mHwComposer
                    - mRenderEngine
                    + CompositionEngine()
                    + ~CompositionEngine()

                    + getHwComposer()
                    + setHwComposer()

                    + getRenderEngine()
                    + setRenderEngine()

                    {static} createCompositionEngine()
                }
                android.compositionengine.CompositionEngine <|.. CompositionEngine
            }
        }

        namespace renderengine {
            class RenderEngine
        }

        interface HWComposer {}

        namespace impl {
            class HWComposer {}
            android.HWComposer <|.. HWComposer
        }
    }
    @enduml
    ```

4. 在 CompositionEngine 增加 Display 类

    ```plantuml
    @startuml
    namespace android {
        namespace compositionengine {
            interface CompositionEngine {
                {abstract} ~CompositionEngine()
                
                {abstract} getHwComposer()
                {abstract} setHwComposer()

                {abstract} getRenderEngine()
                {abstract} setRenderEngine()

                {abstract} createDisplay()
            }
            namespace impl {
                class CompositionEngine {
                    - mHwComposer
                    - mRenderEngine
                    + CompositionEngine()
                    + ~CompositionEngine()

                    + getHwComposer()
                    + setHwComposer()

                    + getRenderEngine()
                    + setRenderEngine()

                    + createDisplay()

                    {static} createCompositionEngine()
                }
                android.compositionengine.CompositionEngine <|.. CompositionEngine
                class Display {
                    {static} createDisplay();
                }
                android.compositionengine.Display <|.. Display
            }
            interface Display {
                {abstract} getId();
                {abstract} isSecure();
                {abstract} isVirtual();
                {abstract} disconnect();
                # ~Display();
            }
        }

        namespace renderengine {
            class RenderEngine
        }

        interface HWComposer {}

        namespace impl {
            class HWComposer {}
            android.HWComposer <|.. HWComposer
        }
    }
    @enduml
    ```

5. 从 `DisplayDevice` 里剥离出一个 Output 类：

    ```plantuml
    @startuml
    namespace android {
        namespace compositionengine {
            interface CompositionEngine {
                {abstract} ~CompositionEngine()
                
                {abstract} getHwComposer()
                {abstract} setHwComposer()

                {abstract} getRenderEngine()
                {abstract} setRenderEngine()

                {abstract} createDisplay()
            }
            interface Display {
                {abstract} getId();
                {abstract} isSecure();
                {abstract} isVirtual();
                {abstract} disconnect();
                # ~Display();
            }
            interface Output {}
            Output <|-- Display

            namespace impl {
                class CompositionEngine {
                    - mHwComposer
                    - mRenderEngine
                    + CompositionEngine()
                    + ~CompositionEngine()

                    + getHwComposer()
                    + setHwComposer()

                    + getRenderEngine()
                    + setRenderEngine()

                    + createDisplay()

                    {static} createCompositionEngine()
                }
                class Display {
                    {static} createDisplay();
                }
                class Output {}

                android.compositionengine.CompositionEngine <|.. CompositionEngine
                android.compositionengine.Display <|.. Display
                android.compositionengine.Output <|.. Output
                Output <|-- Display
            }
        }

        namespace renderengine {
            class RenderEngine
        }

        interface HWComposer {}

        namespace impl {
            class HWComposer {}
            android.HWComposer <|.. HWComposer
        }
    }
    @enduml
    ```

6. 将 `DisplaySurface` 移到 CompositionEngine 里：

    ```plantuml
    @startuml
    namespace android {
        namespace compositionengine {
            interface CompositionEngine {
                {abstract} ~CompositionEngine()
                
                {abstract} getHwComposer()
                {abstract} setHwComposer()

                {abstract} getRenderEngine()
                {abstract} setRenderEngine()

                {abstract} createDisplay()
            }
            interface Display {
                {abstract} getId();
                {abstract} isSecure();
                {abstract} isVirtual();
                {abstract} disconnect();
                # ~Display();
            }
            interface Output {}
            class DisplaySurface {}

            Output <|-- Display

            namespace impl {
                class CompositionEngine {
                    - mHwComposer
                    - mRenderEngine
                    + CompositionEngine()
                    + ~CompositionEngine()

                    + getHwComposer()
                    + setHwComposer()

                    + getRenderEngine()
                    + setRenderEngine()

                    + createDisplay()

                    {static} createCompositionEngine()
                }
                class Display {
                    {static} createDisplay();
                }
                class Output {}
                
                android.compositionengine.CompositionEngine <|.. CompositionEngine
                android.compositionengine.Display <|.. Display
                android.compositionengine.Output <|.. Output
                Output <|-- Display
            }
        }

        namespace renderengine {
            class RenderEngine
        }

        interface HWComposer {}

        namespace impl {
            class HWComposer {}
            android.HWComposer <|.. HWComposer
        }
    }
    @enduml
    ```

7. 