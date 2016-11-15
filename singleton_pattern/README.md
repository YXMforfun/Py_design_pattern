# 单例模式
## 如果某个类只应该有一个实例，那么可通过单例模式来保证
### 对于系统中的某些类来说，只有一个实例很重要，例如，一个系统中可以存在多个打印任务，但是只能有一个正在工作的任务；一个系统只能有一个窗口管理器或文件系统；一个系统只能有一个计时工具或ID(序号)生成器。如在Windows中就只能打开一个任务管理器。如果不使用机制对窗口对象进行唯一化，将弹出多个窗口，如果这些窗口显示的内容完全一致，则是重复对象，浪费内存资源；如果这些窗口显示的内容不一致，则意味着在某一瞬间系统有多个状态，与实际不符，也会给用户带来误解，不知道哪一个才是真实的状态。因此有时确保系统中某个对象的唯一性即一个类只能有一个实例非常重要。

### 最简单的实现方法，创建模块时，把全局状态放在私有变量中，并提供用于访问此变量的公开函数。这里尽管没有引入类，但依然把数据做成了"单例数据值"。
```
_URL = "http://xxx.com/gggg.csv"
get.rates = {}

def get(refresh=False):
    if refresh:
        get.rates = {}
    if get.rates:
        return get.rates
    with urllib.request.urlopen(_URL) as file:
        for line in file:
            line = line.rstrip().decode("utf-8")
            if not line or line.startswith(("#", "Date")):
                continue
            name, currency, *rest = re.split(r"\s*, \s*", line)
            key = "{} ({})".format(name, currency)
            try:
                get.rates[key] = float[rest[-1]]
            except ValueError as err:
                print("error {}: {}".format(err, line))
    return get.rates
```
### decorator实现
```
def singleton(cls):
    _instance = {}
    def getinstance(*args, **kwargs):
        if cls not in _instance:
            _instance[cls] = cls(*args, **kwargs)
        return _instance[cls]
    return getinstance

@singleton
def MyClass(BaseClass):
    pass
```
### singleton class实现
```
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kwargs):
        if not isinstance(cls._instance, cls):
            cls._instance = super(singleton, cls).__new__(cls, *args, **kwargs)
        return cls._instance

class MyClass(Singleton, BaseClass):
    pass
```
### metaclass 实现
```
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(cls, *args, **kwargs)
        return cls._instances[cls]

# Python2
class MyClass(BaseClass):
    __metaclass__ = Singleton
    pass

# Python3
class MyClass(BaseClass, metaclass=Singleton):
    pass
```

### 详细可参照
### http://stackoverflow.com/questions/6760685/creating-a-singleton-in-python
