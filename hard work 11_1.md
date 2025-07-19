# Три правила простого проектного дизайна. Правило 1. Избавляться от точек генерации исключений.

Как избавляться от точек генерации исключений? У меня есть проект Spear and Spirit - проект, который я начал, когда изучал тонкости ООАП. Он представляет собой попытку начинающего геймдевелопера создать HoMM-подобную пошаговую стратегию.

1) Вот класс, объекты которого представляют собой все полезные артефакты в игре:
```Python
class Artifact:  
  
    def __init__(self, name, parameters):  
        self.name = name  
        for key in parameters.keys():  
            self.__setattr__(key, parameters[key])  
  
    def __str__(self):  
        return self.name  
  
    def has_movement_modifier(self):  
        return 'movement' in self.__dict__  
  
    def get_movement_modifier(self):  
        return self.__getattribute__('movement')
```

Последние две функции созданы для проведения тестов создания предметов, повышающих перемещение. Интерес вызывает конструктор: представленный в таком виде, он может послужить причиной постоянных внезапных AttributeError и разнообразных Exception.

С одной стороны, такой конструктор позволяет создавать артефакт с любыми свойствами, с другой стороны, возникает вопрос, с какими? Решением будет атрибут класса, в котором хранятся имена всех возможных параметров.

```Python
class Artifact:

    PARAMETER_NAMES = ['attack', 'defence', 'power', 'knowledge', 'movement', 'unit_health', 'unit_speed', 'morale', 'luck']

    def __init__(self, name, parameters):
        self.name = name
        for parameter in self.PARAMETER_NAMES:
            self.__setattr__(parameter, 0)
            if parameter in parameters.keys():
                self.__setattr__(parameter, parameters[parameter])

    def __str__(self):
        return self.name

    def has_modifier(self, parameter):
        if parameter in self.PARAMETER_NAMES:
            return self.__getattribute__(parameter) != 0
        return False

    def get_modifier(self, parameter):
        return self.__getattribute__(parameter)

    @classmethod
    def check_parameter_name(cls, parameter_name):
        return parameter_name in cls.PARAMETER_NAMES

    @classmethod
    def get_parameter_names(cls):
        return cls.PARAMETER_NAMES

    def get_object_modifiers(self):
        return [mod for mod in self.PARAMETER_NAMES if self.__getattribute__(mod) != 0]
```

Два теста выявляют защищённость класса от опечаток и прочего:
```Python

class TestArtifact(unittest.TestCase):

    def setUp(self):
        self.artifact = Artifact('Crazy Boots', {'movement': 1000})
        self.corrupted_artifact = Artifact('Bad Boots',{'movment': 5000})  # опечатка

    def test_correct_artifact(self):
        self.assertEqual(self.artifact.name, 'Crazy Boots')
        self.assertTrue(self.artifact.has_modifier('movement'))
        self.assertEqual(self.artifact.get_modifier('movement'), 1000)
        self.assertEqual(self.artifact.get_object_modifiers(), ['movement'])

    def test_corrupted_artifact(self):
        self.assertEqual(self.corrupted_artifact.name, 'Bad Boots')
        self.assertFalse(self.corrupted_artifact.has_modifier('movement'))
        self.assertNotEqual(self.corrupted_artifact.get_modifier('movement'), 5000)
        self.assertEqual(self.corrupted_artifact.get_object_modifiers(), [])
```

2) В случае добавления классу Hero из того же проекта новых атрибутов следует добавить возможность создавать артефакты, дающие прибавку по тому же параметру. Учёт возможных параметров параллельно в двух модулях - рассадник ошибок и исключений. Я считаю, что достаточно перенести `PARAMETER_NAMES` в класс Hero, т.к. это параметры, описывающие именно героя. А в Artefact достаточно сделать import.
