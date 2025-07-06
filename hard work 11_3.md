
Как избегать увлечения примитивными типами данных, если кажется, что ты уже матёрый программист и тебе такая беда не грозит. Открываю Spear and Spirit - проект, который я начал, когда изучал тонкости ООАП. Он представляет собой попытку начинающего геймдевелопера создать HoMM-подобную пошаговую стратегию. Смотрю код TestHero.py, смотрю на такой участок кода:

```Python
class TestHero(unittest.TestCase):  
  
    def setUp(self):  
    # ...
        self.knight = Knight(3, 2, 1, 1, knight_sprite)  
        self.barbarian = Barbarian(4, 0, 1, 1, barbarian_sprite)  
        self.green_dragon = GreenDragon()  
        self.thunderbird = ThunderBird()  
        self.boots = Artifact('Crazy Boots', {'movement': 1000})
	# ...
```

Что такое GreenDragon? Открываю Unit.py:

```Python
class GreenDragon(Unit):  
  
    def __init__(self):  
        super().__init__(20, 20, 300, 10)
```

Что значат все эти числа? Если посмотреть конструктор Unit, там найдётся упоминание о них:

```Python
def __init__(self, attack, defence, health, movement):  
    self._attack = attack  
    self._defence = defence  
    self._health = health  
    self._movement = movement
```

Как и у рыцаря с варваром в конструкторе родительского класса указано следующее: 

```Python
def __init__(self, attack: int, defence:int, knowledge:int, power:int, sprite:GraphicObject):  
    self.__attack = attack  
    self.__defence = defence  
    self.__knowledge = knowledge  
    self.__power = power  
  
    self.__movement = Hero.BASIC_MOVEMENT  
  
    self.artifacts = []  
    self.active_artifacts = []  
  
    self.graphic_object = DynamicGraphicObject(sprite)
```

Я вижу такое решение. Создаю два модуля - HeroParameter и UnitParameter:
https://github.com/Paberu/Spear_and_Spirit/blob/main/HeroParameter.py
https://github.com/Paberu/Spear_and_Spirit/blob/main/UnitParameter.py

Каждый из которых содержит целый ряд родственных классов, оборачивающих параметры, так или иначе характеризующие героя или боевой юнит (впоследствии семейство этих модулей может быть расширено).

Новый конструктор юнита:
```Python
class GreenDragon(Unit):  
  
    def __init__(self):  
        super().__init__(UnitAttack(20), UnitDefence(20), UnitHealth(300), UnitMovement(10))
```

Новое создание героя:
```Python
self.knight = Knight(HeroAttack(3), HeroDefence(2), HeroKnowledge(1), HeroPower(1), knight_sprite)
```
а объявление конструктора выглядит, соответственно, так:
```Python
def __init__(self, attack: HeroAttack, defence: HeroDefence, knowledge: HeroKnowledge, power: HeroPower, sprite: GraphicObject):
```
