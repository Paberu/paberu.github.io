# Ясный код-2

1.1 Методы, используемые только в тестах, я нашёл у себя в проекте Spear&Spirit (учебный клон HoMM3, нужный для мотивации и вдохновения).

В модуле Artifact.py создавались классы-потомки, никак друг с другом не связанные. Да и с родительским классом очевидных связок тоже не было (на тот момент он был пуст, только в `__init__()` передавалось имя артефакта). Он нужен был лишь для того, чтобы потом, по возможности, создать интерфейс экипирования себя; впоследствии он реализовывался бы потомками. У каждого артефакта был один модифицируемый параметр, и в результате сомнительные классы обрастали сомнительными методами. И классы, и их тесты не дожили до фазы **commit - push**.

Сейчас всю эту странность заменяет один неабстрактный класс Artifact
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
	
	# дальше так и будут связки has_attack_modifier(), get_attack_modifier(), has_defence_modifier(), has_defence_modifier() и т.д.
```
Такой класс позволяет задавать артефакты не с одним модификатором (как тот щит, например, с +12 к Защите и -3 к Атаке), создавать новые категории артефактов и т.д., т.к. в проекте пока нет и не будет никаких категорий - класс самодостаточен. Благодаря этому ушли лишние тесты с позорными вызовами. В целом, отрадно, что это не пережило критического взгляда доброго утра и теперь не пачкает историю моего GitHub.

Так выглядит тестирование класса Artifact:
```Python
def test_creation(self):  
    artifact = Artifact('Crazy Boots', {'movement': 1000})  
    self.assertEqual(artifact.name, 'Crazy Boots')  
    self.assertTrue(artifact.has_movement_modifier())  
    self.assertEqual(artifact.get_movement_modifier(), 1000)
```


1.3. У метода слишком большой список параметров. Видел подобную ситуацию в отечественном старом учебнике по C. 
Текст не нашёл, воспроизвожу по памяти:
```C
void get_next_char(int x, int y, bool need_next, char* current, char* next) {
    //в данной функции есть 3 символа, x и y символизируют координаты на плоскости, булева переменная определяет необходимость смены символа
    //ссылки на переменные типа char позволяют нам не возвращать какое-то конкретное значение из функции, а передать в саму функцию несколько переменных, в которые можно сохранить результаты вычислений
}
```
С моей колокольни, это - сущий ад. Если так нужно передать несколько значений из функции, давайте создадим array из char. Если переменные отличаются типами, давайте создавать structure. Но засыпать вход аргументами, половина из которых - выходные... До сих пор вздрагиваю.


1.2 (он же 1.5). Т.е. случилась цепочка методов, которая произошла из-за того, что первая функция отдавала больше, чем нужно. Вот она:
```Python
def parse_salary(salary):
    money_template = re.compile(r'(\d{1,3}).(\d{3})')
    country = re.compile(r'[USD|руб].*')
    if not salary.endswith('не указана'):
        salary_delta = []
        value = country.findall(salary)
        for money_tuple in money_template.findall(salary):
            salary_delta.append(int(''.join(d for d in money_tuple)))
        return salary_delta, value
    return None
```
Функция получает информацию о зарплате для данной вакансии, проводит две проверки по регуляркам: получает валюту страны из двух возможных (доллары США и российский рубли) и формирует список из двух значений (если есть зарплатная вилка) или из одного (если конкретное значение).
Потом вызывает функция, которая проверяет валюту и переводит, если надо, в рубли. Потом функция, которая ещё раз ковыряет salary на предмет наличия фразы "до уплаты налогов". И ещё какой-то мусор.

Исправил я так:
```Python
# для начала я сразу задал ключевые параметры: налоги и обозначения валют, курс пока задаётся руками, но на данный момент это не так важно
TAXES = 0.13  
COURSES = {  
    'USD': 94.15,  
    'KZT': 0.21,  
    'руб': 1,  
    '₽': 1,  
    '$': 94.15,  
    '€': 100.08,  
    '': 1  
}

# И вот новая версия parse_salary. Получаем кусок нужного html'а и уже из него добываем всю информацию
@staticmethod
def parse_salary(soup):  
    salary_tag = soup.find('div', attrs={'data-qa': 'vacancy-salary'})  
    if not salary_tag:  
        return []  # если у вакансии нет тэга с запрлатой, возвращаем пустой список
    salary = ''.join(salary_tag.find('span').stripped_strings)  # получили строковое значение зарплаты 
    salary = salary.replace('\xa0', '')  # заменяем разделитель разряда пустой строкой  
    currency_template = r'(' + '|'.join(COURSES.keys()) + ')(.*)'  # собираем шаблон из всех значимых валют  
    value_template = re.compile(currency_template)  # компилируем шаблон поиска  
    try:  
        currency, tax_flag = value_template.findall(salary)[0]  # выполняем поиск, группируем результат  
    except IndexError:  
        return []  # если поиск не вернул ничего, значит тэг есть, но он пустой
  
    money_template = re.compile(r'(\d{3,7})')  
    salary_delta = list(map(int, money_template.findall(salary))) # переводим все найденные значения в int, формируем список  
    for i in range(len(salary_delta)):  # нельзя воспользоваться итератором, надо отредактировать каждое значение в массиве  
        if tax_flag == 'до вычета налогов':  # высчитываем налог, чтобы не тешить себя иллюзиями  
            salary_delta[i] = round(salary_delta[i] * (1 - TAXES))  
        salary_delta[i] = salary_delta[i] * COURSES[currency]  
    return salary_delta
```
В последних строках функции происходит высчитывание налогов, приведение к рублю и возврат вилки или конкретного значения. Сейчас я понимаю, что в вакансии мало кого интересует начало "от 100 000 рублей". В этом нет интриги, как во фразе "от Сида Мейера". Интрига в том, до 150 000 или до 200 000. Ну и не будем забывать о любви HR-ов к преуменьшению: "Мы натолкали избыточных требований, чтобы соискатель признался, что чего-то не умеет, и можно было поторговаться". И в итоге, три вакансии с з/п 150 000, с з/п до 150 000 и с з/п от 120 000 до 150 000 - это по сути своей три вакансии с одинаковым уровнем зарплаты. Примерно ниже 150 000. Считаю это озарением

Решено, возвращать надо `max(salary_delta)`. Всё. Спасибо за инсайт.

1.4. В моём проекте HeadHunterParser я использую библиотеку BeautifulSoup, которая прекрасно парсит html-разметку, полученную с помощью библиотеки requests. Объект BeautifulSoup создаётся следующим образом:
`soup = BeautifulSoup(request_result, parser)`, где `request_result` - это как раз и есть та html-разметка, а `parser` - это тип парсера. На выбор можно использовать: 'html.parser', 'lxml', 'lxml-xml', 'xml', 'html5lib'. И есть проблема, когда реально создаются одинаковые функции, с небольшой разницей в начале, например:
```Python
def parse_categories_lxml(tag):
	soup = BeautifulSoup(tag, 'lxml')
	# ...
	
def parse_categories_html5lib(tag):
	soup = BeautifulSoup(tag, 'html5lib')
	# ...
```
... и небольшой разницей где-то в середине метода.

Видимо, для разных задач, подходят разные парсеры, или ещё что-то. Я бы или провёл исследование перед использованием BeautifulSoup, чтобы заранее выбрать парсер, либо сделал бы так:
```Python
def parse_categories(tag):
	# получение параметров из tag
	parser = choose_parser(tag_parameters)
	soup = BeautifulSoup(tag, parser)
	# ...

def choose_parser(tag_parameters):
	if tag_parameters_meets_lxml_expectations:
		return 'lxml'
	if tag_parameters_meets_html5lib_expectations:
		return 'html5lib'
	# etc.

def unstable_of_parse_categories(parser):
	# сюда попадает весь, зависимый от выбранного парсера код
	# это позволяет хранить исходный parse_categories чистым

```
