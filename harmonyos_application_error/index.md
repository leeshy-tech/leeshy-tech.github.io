# 程序报错总结——HarmonyOS

### 1

{{< admonition type=bug title="Bug" open=true >}}
不可转换的类型；无法将 'ohos.agp.components.Component' 转换为 'ohos.ai.cv.text.Text'
{{< /admonition >}}

原因：使用Tab键补全Text时，Studio自动引库引到了错误的库`import ohos.ai.cv.text.Text;`，实际应当是`import ohos.agp.components.Text;`。

![image-20220127223517139](/image/ApplicationError/image-20220127223517139.png)
### 2

{{< admonition type=bug title="Bug" open=true >}}
java.lang.NullPointerException: Attempt to invoke virtual method 'void ohos.agp.components.Button.setClickedListener(ohos.agp.components.Component$ClickedListener)' on a null object reference
{{< /admonition >}}

原因：在写xml文件时，误把注释写成了//，应该是`<!-- -->`。这里它不会直接报错，而是说我获得的按键对象为空，说明xml构建出了问题。

![image-20220130212423710](/image/ApplicationError/image-20220130212423710.png)

### 3

{{< admonition type=bug title="Bug" open=true >}}
ohos.agp.window.wmc.AGPWindowManager$O000000: Can't show toast, Looper.don't prepare
{{< /admonition >}}

![image-20220220171756254](/image/ApplicationError/image-20220220171756254.png)

原因：一般是因为在子线程中直接操作UI导致的，这个主要是指导致UI重绘的操作，指上图中的.show()，.setText()是可以在子线程中正常使用的。

解决办法：![image-20220220172446640](/image/ApplicationError/image-20220220172446640.png)

利用getUITaskDispatcher把UI操作返回到主线程中。
