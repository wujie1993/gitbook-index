# mermaid

## 项目地址

{% embed url="https://github.com/knsv/mermaid" %}

## 官方文档

{% embed url="https://mermaidjs.github.io/" %}

## 在线编辑器

{% embed url="https://mermaidjs.github.io/mermaid-live-editor" %}

## 示例

### 流程图

```text
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

显示效果：

![](http://knsv.github.io/mermaid/img/flow.png)

### 序列图

```text
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->>John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail!
    John-->>Alice: Great!
    John->>Bob: How about you?
    Bob-->>John: Jolly good!
```

显示效果：

![](http://knsv.github.io/mermaid/img/sequence.png)

### 甘特图

```text
gantt
dateFormat  YYYY-MM-DD
title Adding GANTT diagram to mermaid
excludes weekdays 2014-01-10

section A section
Completed task            :done,    des1, 2014-01-06,2014-01-08
Active task               :active,  des2, 2014-01-09, 3d
Future task               :         des3, after des2, 5d
Future task2               :         des4, after des3, 5d
```

显示效果：

![](http://knsv.github.io/mermaid/img/gantt.png)

### 类图（实验性）

```text
classDiagram
Class01 <|-- AveryLongClass : Cool
Class03 *-- Class04
Class05 o-- Class06
Class07 .. Class08
Class09 --> C2 : Where am i?
Class09 --* C3
Class09 --|> Class07
Class07 : equals()
Class07 : Object[] elementData
Class01 : size()
Class01 : int chimp
Class01 : int gorilla
Class08 <--> C2: Cool label
```

显示效果：

![](../../.gitbook/assets/image.png)

### Git分支图（实验性）

```text
gitGraph:
options
{
    "nodeSpacing": 150,
    "nodeRadius": 10
}
end
commit
branch newbranch
checkout newbranch
commit
commit
checkout master
commit
commit
merge newbranch
```

显示效果：

![](http://knsv.github.io/mermaid/img/git.png)

## VSCode插件

{% embed url="https://marketplace.visualstudio.com/items?itemName=vstirbu.vscode-mermaid-preview" %}

