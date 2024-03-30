# Константы в Python

Практически в каждом языке программирования есть константы - переменные, значение которым задаётся один раз и больше не меняется. А в Python их нет.
Любой переменной можно присвоить значение после того, как она была проинициализирована каким-то исходным значением. Иногда это недопустимо, и хочется, чтобы инструменты работы с исходным кодом хотя бы намекали об ошибке программисту, раз уж интерпретатор молчит (в компилируемых языках такой код просто не скомпилируется).

Есть способ, как заставить хотя бы среду разработки нервно отреагировать предупреждениями в ситуации с изменением значения константы.

Модуль typing, добавленный в версию 3.5, позволяет добавить в код инструкцию Final:
```Python
from typing import Final

PUT_STATUS_OK: Final = 1
```
В указанном коде используется указание для линтера (программы или библиотеки, которая проверяет код на соответствие правилам) - эту переменную менять нельзя, её значение финализировано. Если прогонять код через анализатор кода, он заметит несоответствие ожидаемому обращению с переменной с реальным поведением кода (может быть человек потерял одно = и вместо сравнения использует присвоение).

Это единственный способ создания псевдоконстанты в Python:
1. Следование соглашению об именование переменных - слова пишутся прописными буквами с разделением символом `_`.
2. Использование Final из модуля typing (в принципе, использование аннотации типов - хорошая практика).
3. Использование линтеров для проверки кода.