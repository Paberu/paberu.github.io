# Три правила простого проектного дизайна. Правило 2. Отказаться от дефолтных конструкторов без параметров.

Как избегать увлечения примитивными типами данных, если кажется, что ты уже матёрый программист и теКак отказаться от дефолтных конструкторов без параметров? У меня есть проект три-в-ряд, консольная игра, т.к. хотелось понять внутреннюю логику старых игр, а на графику отвлекаться пока не хотелось.

В модуле GameMaster есть такой кусок кода: 
```Python
if __name__ == '__main__':  
    game_master = GameMaster()  
    game_board = GameBoard()
```
Размеры игрового поля заданы константами в классе GameBoard. Если захочется сделать размеры поля настраиваемыми, то придётся ковырять код программы и исправлять целочисленные константы. Да, прямо в теле файла. Если хранить настройки игры в файле, то вообще наличие жёстко прописанных констант не просто непонятно, а даже вредно.

Если избавиться от пустого конструктора для GameBoard? Было:
```Python
COLUMNS: Final = 8  
ROWS: Final = 8  
BOARD_FIELDS: Final = COLUMNS * ROWS  
FAILED_ATTEMPTS: Final = 5  
  
class GameBoard:  
    # docstring
    MATCH_NIL: Final = 0  
    MATCH_OK: Final = 1  
    MATCH_NOT: Final = 2  
    HAS_NOT_EMPTY_CELL: Final = 0  
    HAS_EMPTY_CELL: Final = 1  
  
    def __init__(self):  
        # docstring  
        self._match_status = self.MATCH_NIL  
        self._has_empty_cell = self.HAS_NOT_EMPTY_CELL  
        self._counters = {'moves': 0,  
                          'score': 0,  
                          'failed_attempts': 0}  
        self._board = [[Cell() for _ in range(COLUMNS)] for _ in range(ROWS)]
```

Стало:
```Python
class GameBoard:  
    # docstring
    MATCH_NIL: Final = 0  
    MATCH_OK: Final = 1  
    MATCH_NOT: Final = 2  
    HAS_NOT_EMPTY_CELL: Final = 0  
    HAS_EMPTY_CELL: Final = 1  

	def __init__(self, columns: int, rows: int, failed_attempts: int):  
	    # docstring
	    self.COLUMNS = columns  
	    self.ROWS = rows  
	    self.BOARD_FIELDS = self.COLUMNS * self.ROWS  
	    self.FAILED_ATTEMPTS = failed_attempts  
	  
	    self._match_status = self.MATCH_NIL  
	    self._has_empty_cell = self.HAS_NOT_EMPTY_CELL  
	    self._counters = {'moves': 0,  
	                      'score': 0,  
	                      'failed_attempts': 0}  
	    self._board = [[Cell() for _ in range(self.COLUMNS)] for _ in range(self.ROWS)]  # переход на двумерный массив
```

Исправив некоторое количество функций, использовавших псевдоконстанты (настоящих констант в Python всё равно нет), получился такой вариант исправленного кода:
```Python
rows, columns, failed_attempts = 8, 12, 4  # сделать чтение из конфигурационного файла  
game_master = GameMaster()  
game_board = GameBoard(columns, rows, failed_attempts)
```

Специально оставил себе пометку для создания конфигурационного экрана, сейчас это не к спеху. GameMaster на данный момент задействован слабо, поэтому его пока можно оставить с пустым конструктором. Но в дальнейшем (как только у игры появится честный графический модуль) это будет незамедлительно исправлено.