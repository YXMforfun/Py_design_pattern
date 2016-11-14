# 原始工厂模式

## 两个工厂 ： 一个生成纯文本格式示意图， 一个生成SVG格式示意图
### 同样的调用方式
```
def main():
    ...
    txtDiagram = create_diagram(DiagramFactory())
    txtDiagram.save(textFilename)

    svgDiagram = create_diagram(SvgDiagramFactory())
    svgDiagram.save(svgFilename)
```

### 函数不清楚工厂的具体类型，只知道工厂对象具备创建示意图的接口
```
def create_diagram(factory):
    diagram = factory.make_diagram(30, 7)
    rectangle = factory.make_rectangle(4, 1, 22, 5, "yellow")
    text = factory.make_text(7, 3, "Abstract Factory")
    diagram.add(rectangle)
    diagram.add(text)
    return diagram
```

### 文本格式示意图工厂 ，同时该工厂也是其他工厂的基类
```
class DiagramFactory(object):
    def make_diagram(self, width, height):
        return Diagram(width, height)

    def make_rectangle(self, x, y, width, height, fill="white", stroke="black"):
        return Rectangle(x, y, width, height, fill, stroke)

    def make_text(self, x, y, text, fontsize=12):
        return Text(x, y, text, fontsize)
```

### SVG 格式示意图工厂
```
class SvgDiagramFactory(DiagramFactory):
    def make_diagram(self, width, height):
        return SvgDiagram(width, height)

    ...

```

### Text 类
```
class Text(object):
    def __init__(self, x, y, text, fontsize):
        self.x = x
        self.y = y
        self.rows = [list(text)]

```


### SvgText 类
```
SVG_TEXT = """<text x="{x}" y="{y}" text-anchor="left"\
	font-family="sans-serif" font-size="{fontsize}">{text}</text>"""
SVG_SCALE = 20


class SvgText(object):
    def __init__(self, x, y, text, fontsize):
        x *= SVG_SCALE
        y *= SVG_SCALE
        fontsize *= SVG_SCALE
        self.svg = SVG_TEXT.format_map(locals())

```


### Diagram 类
```
class Diagram(object):
    def __init__(self, width, height):
        self.width = width
        self.height = height
        # 填充好一个diagram
        self.diagram = generate_diagram(width, height)

    # 调用该方法时，可能是Text 或 Rectangle
    def add(self, component):
        for y, row in enumerate(component.rows):
            for x, char in enumerate(row):
                self.diagram[y + component.y][x + component.x] = char

```

### SvgDiagram 类
```
class SvgDiagram(object):
    ...

    def add(self, component):
        self.diagram.append(component.svg)

```

### 上述写法有几个缺点
1. 两个工厂都没有各自的状态，不需要创建工厂实例。
2. SvgDiagramFactory 与 DiagramFactory 基本上一模一样，只不过是 make_diagram 返回的实例不一样。
3.  DiagramFactory、 Diagram、 Rectangle、Text类以及SVG 系列中与其对应的那些类都放在了top-level namespace里，而且还需要避免名称冲突。

## Python 工厂模式

### 改动 Pythonic 的工厂模式
```
class DiagramFactory:
    @classmethod
    def make_diagram(cls, width, height):
        return cls.Diagram(width, height)

    @classmethod
    def make_rectangle(cls, x, y, width, height, fill="white", stroke="black"):
        return cls.Rectangle(x, y, width, height, fill, stroke)

    @classmethod
    def make_text(cls, x, y, text, fontsize=12):
        return cls.Text(x, y, text, fontsize)
```


### 改动后，首个参数是类，也就是说哪个类调用，就将哪个类的对象返回。
### 意味着， SvgDiagramFactory 子类只需继承 DiagramFactory，无需重复实现多个make 方法。

### 调用
```
def main():
    ...
    txtDiagram = create_diagram(DiagramFactory)
    txtDiagram.save(textFilename)

    svgDiagram = create_diagram(SvgDiagramFactory)
    svgDiagram.save(svgFilename)
```

### SvgDiagramFactory 实现
### 此时，访问相关常数及非工厂类时，必须在名称前加工厂名字，因为现在都嵌套在工厂类里了。
```
class SvgDiagramFactory(DiagramFactory):
    ...
    SVG_TEXT = """<text x="{x}" y="{y}" text-anchor="left"\
    	font-family="sans-serif" font-size="{fontsize}">{text}</text>"""
    SVG_SCALE = 20

    class Text:
        def __init__(self, x, y, text, fontsize):
            x *= SvgDiagramFactory.SVG_SCALE
            y *= SvgDiagramFactory.SVG_SCALE
            fontsize *= SvgDiagramFactory.SVG_SCALE
            self.svg = SvgDiagramFactory.SVG_TEXT.format(**locals())
```
