# Генератор паролей на Python с использованием модуля random.

Вообще для обеспечения безопасности не рекомендуют использовать функции модуля random (всё-таки генерируемые там последовательности псевдослучайны). Но как учебный проект, который можно использовать для защиты домашнего ПК, пойдёт.

```Python
class PasswordGenerator:

    def __init__(self, size):
        self.size = size

    def generate_password(self):
        specific_symbols_count = self.size // 6 + 1
        digits_count = self.size // 4
        uppercase_count = (self.size - specific_symbols_count - digits_count) // 2
        lowercase_count = self.size - uppercase_count - specific_symbols_count - digits_count

        password_symbols = [random.choice(string.punctuation) for _ in range(specific_symbols_count)]
        password_symbols.extend([random.choice(string.digits) for _ in range(digits_count)])
        password_symbols.extend([random.choice(string.ascii_uppercase) for _ in range(uppercase_count)])
        password_symbols.extend([random.choice(string.ascii_lowercase) for _ in range(lowercase_count)])

        random.shuffle(password_symbols)

        return ''.join(password_symbols)
```
В конструкторе передаётся только один параметр - длина пароля. 
В самой генерации сначала задаётся количество символов из разных групп: спецсимволы, цифры, малые и большие буквы. Генерируются случайные списки из каждой категории, которые объединяются в один. Потом они перемешиваются случайным образом.

Вкусное в этом коде:
- Группы символов из пакета string. Вместо того, чтобы самому набирать `['1234567890']` достаточно обратиться к string.digits. Остальное по аналогии.
- Генераторы. Нам надо повторить одну и ту же функцию n раз и сформировать список, пожалуйста: `[f(x) for _ in range(n)]`. Здесь прекрасно то, что функция f(x) действительно выполнится n раз, при этом даже не нужно вводить переменную, которая будет выполнять роль счётчика итераций.
- Превращение списка в строку. `''.join(список)` даёт нам конвертацию каждого элемента списка в строку, при этом между ними ставится символ, для которого вызывается join(), в нашем случае - пустая строка, т.е. все элементы будут слиты воедино без разделителей и т.п.