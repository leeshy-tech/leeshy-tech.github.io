# 程序报错总结——HarmonyOS


> 不可转换的类型；无法将 'ohos.agp.components.Component' 转换为 'ohos.ai.cv.text.Text'

```
Text text = (Text) findComponentById(ResourceTable.Id_text1);
```

原因：使用Tab键补全Text时，Studio自动引库引到了错误的库`import ohos.ai.cv.text.Text;`，实际应当是`import ohos.agp.components.Text;`。

![image-20220127223517139](/image/ApplicationError/image-20220127223517139.png)
