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
