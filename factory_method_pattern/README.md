# 工厂方法模式 Factory method pattern

## 如果子类的某方法要根据情况来决定用什么类去实例化相关对象，那么可以考虑工程方法模式。此模式可单独使用，也可在无法预知对象类型时使用。
## 例子为两种棋盘生成程序，一共为四个版本，从一至四不断改进。
## 先设计出抽象的棋盘类，然后用其子类创建特定的棋盘。每个子类都会生成相应的棋盘，并把棋子摆放好。
### 顶层代码，实例化棋盘对象。
```
def main():
    checkers = CheckersBoard()
    print(checkers)

    chess = ChessBoard()
    print(chess)
```
### 抽象棋盘类
```
# 按惯例可写成 BLACK, WHITE = range(2)，
# 但是使用字符串来定义常量在调试时更容易看出错误信息的定义，
# 而且Python 还会自动把内容相同的字符串规整起来，保留一份。
BLACK, WHITE = ("BLACK", "WHITE")

class AbstractBoard:

    def __init__(self, columns, rows):
        # 二维列表表示棋盘
        self.board = [[None for _ in range(columns)] for _ in range(rows)]
        self.populate_board()

    # 虽然可以通过把AbstractBoard 类的 metaclass 设置成 abc.ABCMeta
    # 可以使类成为抽象基类，但是在这里，改用:
    # 凡是需要由子类重新实现的方法都抛出 NotImplementError 异常。
    def populate_board(self):
        raise NotImplementError()

    def __str__(self):
        squares = []
        for y, row in enumerate(self.board):
            for x, piece in enumerate(row):
                square = console(piece, BLACK if (y + x) % 2 else WHITE)
                squares.append(square)
            squares.append("\n")
        return "".join(squares)
```
## 版本1
### 国际跳棋棋盘- CheckersBoard(版本1) 硬编码实现
```
class CheckersBoard(AbstractBoard):

    def __init__(self):
        super().__init__(10, 10)

    def populate_board(self):
        for x in range(0, 9, 2):
            for row in range(4):
                column = x + ((row + 1) % 2)
                self.board[row][column] = BlackDraught()
                self.board[row + 6][column] = WhiteDraught()

```
### 国际象棋棋盘- ChessBoard(版本1) 硬编码实现
```
class ChessBoard(AbstractBoard):

    def __init__(self):
        super().__init__(8, 8)

    def populate_board(self):
        self.board[0][0] = BlackChessBoard()
        self.board[0][1] = BlackChessKnight()
        ...
        self.board[7][7] = WhiteChessRook()
        for column in range(8):
            self.board[1][column] = BlackChessPawn()
            self.board[6][column] = WhiteChessPawn()
```
### 所有棋子的基类
```
class Piece(str):
    # __slots__ = () 保证实例中不会有任何数据，
    # __slots__ = ('xx','yy')表示类只能添加'xx','yy'的属性，
    # 且对继承的子类是不起作用的。
    __slots__ = ()
```
### 所有棋子类的范本
```
class BlackDraught(Piece):
    __slots__ = ()
    def __new__(cls):
        return super().__new__(cls, "\N{black draughts man}")

class WhiteChessKing(Piece):
    __slots__ = ()
    def __new__(cls):
        return super().__new__(cls, "\N{white chess king}")

# 棋子类的实现类似，只是类名和所含的字符串不同
...

```
### 以上的 populate_board 都不是工厂方法，因为都是以硬编码实现，不会根据类的不同实现不同的实例。

##  版本2
### 此时的 populate_board() 为工厂方法，而其中的 create_piece()，也就是工厂函数会根据参数返回适当类型的对象。工厂函数使用 Python 语言内置的 eval() 函数来创建对应类的实例，但不建议使用 eval()。使用 eval() 其中可能会有风险，因为任何字符串都会作为 Python 代码交给 eval() 函数执行。
### 同时实现了以一段代码块为模板，将所有棋子类全都创建好的代码，在该代码中使用了 exec()，但是实际上使用 exec() 所带来的风险比用 eval() 还要高。
```
class CheckersBoard(AbstractBoard):

    def __init__(self):
        super().__init(10, 10)

    def populate_board(self):
        for x in range(0, 9, 2):
            for y in range(4):
                column = x + ((y + 1) % 2)
                for row, color in ((y, "black"), (y + 6, "white")):
                    self.board[row][column] = create_piece("draught", color)

def create_piece(kind, color):
    if kind == "draught":
        return eval("{}{}()".format(color.title(), kind.title()))
    return eval("{}Chess{}()".format(color.title(), kind.title()))

class Piece(str):
    __slots__ = ()

for code in itertools.chain((0x26c0, 0x26c2), range(0x2654, 0x2660)):
    char = chr(code)
    name = unicodedata.name(char).title().replace(" ", "")
    if name.endswith("sMan"):
        name = name[:-4]
    exec("""\
class {}(Piece):
    __slots__ = ()

    def __new__(cls):
        return super().__new__(cls, "{}")""".format(name, char))

```
## 版本3
### 将字符串字面值替换为常量，降低错误的发生，改动 eval() 与 exec()，使用 type() 来代替类的生成。
```
DRAUGHT, PAWN, ROOK, KINGHT, BISHOP, KING, QUEEN = ("DRAUGHT", "PAWN",\
    "ROOK", "KINGHT", "BISHOP", "KING", "QUEEN")

class AbstractBoard:
    __classForPiece = {(DRAUGHT, BLACK): BlackDraught,
                       (PAWN, BLACK): BlackChessPawn,
                       ...
                       (QUEEN, WHITE): WhiteChessQueen}
    ...
    # 在类里的字典中找到对应的类，返回"类对象"，调用() 实例化
    def create_piece(self, kind, color):
        return AbstractBoard.__classForPiece[kind, color]()

class CheckersBoard(AbstractBoard):
    ...
    def populate_board(self):
        for x in range(0, 9, 2):
            for y in range(4):
                column = x + ((y + 1) % 2)
            for row, color in ((y, BLACK), (y + 6, WHITE)):
                self.board[row][column] = self.create_piece(DRAUGHT, color)

for code in itertools.chain((0x26c0, 0x26c2), range(0x2654, 0x2660)):
    char = chr(code)
    name = unicodedata.name(char).title().replace(" ", "")
    if name.endswith("sMan"):
        name = name[:-4]
    # 改变 exec() 为以下代码
    new = make_new_method(char)
    Class = type(name, (Piece,), dict(__slots__ = (), __new__ = new))
    setattr(sys.modules[__name__], name, Class)
```
### 在知道棋子相应的字符及类名之后，使用自定义的 make_new_method() 函数来创建 new() 函数，然后再使用 type() 函数来创建新的类。用 type() 创建类的时候，必须传入类型的名称、含有基类名称的元祖(这里只有一个，可多个)以及含有类属性的字典。最后，调用 setattr() 函数，把 Class 当作属性添加到当前的模块。
### make_new_method() 函数
### 在创建 new() 函数时不能调用 super()，因为此处没有并没有 super() 函数所需的"类环境"。而这里 Piece 类没有 \__new\__() 方法，但是其基类 str 有，所以调用的是 str.\__new\__()。
```
def make_new_method(char):
    def new(Class):
        return Piece.__new__(Class, char)
    return new

# 实际上，可以将上面的 new = make_new_method(char) 用 lambda 表示 (比较tricky)
    new = (lambda char : lambda Class: Piece.__new__(Class, char))(char)
# setattr(sys.modules[__name__], name, Class) 也可改为：
    globals()[name] = Class
```
## 版本4
### 将 \__classForPiece 封装在 create_piece()中，并把 CheckersBoard.populate_board()、ChessBoard.populate_board() 改进。
```
class CheckersBoard(AbstractBoard):

    def __init__(self):
        self.populate_board()

    def populate_board(self):
        def black():
            return create_piece(DRAUGHT, BLACK)
        def white():
            return create_piece(DRAUGHT, WHITE)
        rows = ((None, black()), (black(), None), (None, black()),
                (black(), None),
                (None, None), (None, None), (None, white()),
                (white(), None), (None, white()),
                (white(), None))
        self.board = [list(itertools.islice(
            itertools.cycle(squares), 0, len(rows))) for squares in rows]

class ChessBoard(AbstractBoard):

    def __init__(self):
        super().__init__(8, 8)

    def populate_board(self):
        for row, color in ((0, BLACK), (7, WHITE)):
            for columns, kind in (((0, 7), ROOK), ((1, 6), KNIGHT),
                        ((2, 5), BISHOP),((3,), QUEEN), ((4,), KING)):
                for column in columns:
                    self.board[row][column] = create_piece(kind, color)
        for column in range(8):
            for row, color in ((1, BLACK), (6, WHITE)):
                self.board[row][column] = create_piece(PAWN, color)

def create_piece(kind, color):
    color = "White" if color == WHITE else "Black"
    reflect = {DRAUGHT: "Draught", PAWN: "ChessPawn", ROOK: "ChessRook",
        KNIGHT: "ChessKnight", BISHOP: "ChessBishop",
        KING: "ChessKing", QUEEN: "ChessQueen"}
    name = reflect[kind]
    return globals()[color + name]()
```
