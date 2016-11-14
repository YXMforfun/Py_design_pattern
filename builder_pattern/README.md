# 建造者模式 (Builder Pattern)
### 建造者模式 (Builder Pattern) 于抽象工厂模式类似，都可以创建那种需要由其他对象组合而成的复杂对象。而两者的区别在于，建造者模式不仅提供了创建复杂对象所需的方法，而且保存了复杂对象里各个部分的细节。

### 以下例子为，创建一段表单 form。

### 顶层调用语句
```
htmlForm = create_login_form(HtmlFormBuilder())
with open(htmlFilename, "w", encoding="utf-8") as file:
    file.write(htmlForm)

tkForm = create_login_form(TkFormBuilder())
with open(tkFilename, "w", encoding="utf-8") as file:
    file.write(tkForm)


def create_login_form(builder):
    builder.add_title("Login")
    builder.add_label("Username", 0, 0, target="username")
    builder.add_entry("username", 0, 1)
    builder.add_label("Password", 1, 0, target="password")
    builder.add_entry("password", 1, 1, kind="password")
    builder.add_button("Login", 2, 0)
    builder.add_button("Cancel", 2, 1)
    return builder.form()

```
## 抽象基类
### metaclass 为 abc.ABCMeta，该类无法初始化。继承自AbstractFormBuilder 的类必须实现所有抽象方法。这种情况在将C++ 或 Java 代码移植到 Python 时，比较有用。 实际上在 Python 里，有比较宽松的做法，就是根本不使用 metaclass, 而是直接在文档中说明该类只能作为抽象基类。
```
import abc
class AbstractFormBuilder(metaclass=abc.ABCMeta):
    @abc.abstractmethod
    def add_title(self, title):
        self.title = title

    @abc.abstractmethod
    def form(self):
        pass

    @abc.abstractmethod
    def add_label(self, text, row, column, **kwargs):
        pass

    @abc.abstractmethod
    def add_entry(self, variable, row, column, **kwargs):
        pass

    @abc.abstractmethod
    def add_button(self, id, row, column, **kwargs):
        pass
```
## HTML Builder
```
class HtmlFormBuilder(AbstractFormBuilder):
    def __init__(self):
        self.title = "HtmlFormBuilder"
        self.items = {}

    def add_title(self, title):
        super().add_title(escape(title))

    def add_label(self, text, row, column, **kwargs):
        self.items[(row, column)] = ('<td><label for="{}">{}:</label></td>'
                                     .format(kwargs["targets"], escape(text)))

    def add_entry(self, variable, row, column, **kwargs):
        html = """<td><input name="{}" type="{}" /></td>""".format(
            variable, kwargs.get("kind", "text")
        )
        self.items[(row, column)] = html

    def add_button(self, id, row, column, **kwargs):
        html = """<button id="{}" type="{}"></button>""".format(
            id, kwargs.get("type", "submit")
        )
        self.items[(row, column)] = html

    def form(self):
        html = ["<!doctype html>\n<html><head><title>{}</title></head><body>"
                    .format(self.title), '<form><table border="0">']
        thisRow = None
        for key, value in sorted(self.items.items()):
            row, column = key
            if thisRow is None:
                html.append("  <tr>")
            elif thisRow != row:
                html.append("  </tr>\n <tr>")
                thisRow = row
                html.append("    " + value)
            html.append("  </tr>\n</table></form></body></html>")
        return "\n".join(html)
```

## Tk Builder
```
class TkFormBuilder(AbstractFormBuilder):

    # TEMPLATE
    # 根据内容的不同，模板化运行
    TEMPLATE = """
    #!/usr/bin/env python3
    import tkinter as tk
	import tkinter.ttk as ttk

    class {name}Form(tk.Toplevel):

        def __init__(self, master):
    		super().__init__(master)
			self.withdraw()
			self.title("{title}")
			{statements}
			self.bind("<Escape>", lambda *args: self.destory())
			self.deiconify()
			if self.winfo_viewable():
				self.transient(master)
			self.wait_visibility()
			self.grab_set()
			self.wait_window(self)

	if __name__ == "__main__":
		application = tk.Tk()
    	window = {name}Form(application)
		application.protocol("WM_DELETE_WINDOW", application.quit)
		application.mainloop()
    """

    def __init__(self):
        self.title = "TkFormBuilder"
        self.statements = []

    #_canonicalize 是用于取得控件的"规范名称"
    def _canonicalize(self, text, startLower=True):
        text = re.sub(r"\W+", "", text)
        if text[0].isdigit():
            return "_" + text
        return text if not startLower else text[0].lower() + text[1:]

    def add_title(self, title):
        super().add_title(title)

    def add_label(self, text, row, column, **kwargs):
        name = self._canonicalize(text)
        create = """self.{} Label = ttk.Label(self, text="{}:")""".format(
            name, text)
        layout = """self.{} Label.grid(row={}, column={}, sticky=tk.W, \
            padx="0.75m", pady="0.75m")""".format(name, row, column)
        self.statements.extends((create, layout))

    # add_* 实现都差不多，具体看源码
    ...

    def form(self):
		return TkFormBuilder.TEMPLATE.format(title=self.title,
			name=self._canonicalize(self.title, False),
			statements="\n        ".join(self.statements))



```
