# Антипаттерн "Самодокументирующийся код"

1. Класс Vacancy из приложения HHParser:
https://github.com/Paberu/HeadHunterParser/blob/with_gui/Vacancy.py
```Python
TAXES = 0.13  
COURSES = {  
    'USD': 91,  
    'KZT': 0.2,  
    'руб': 1,  
    '₽': 1,  
}  
  
  
class Vacancy:  
    """ Класс Vacancy используется для обработки данных, получаемых со страниц вакансий, опубликованных на портале  
    hh.ru. Выше по файлу определены константы, которые используются при разборе имеющихся вакансий. На данный момент
    это налоговая ставка, принятая в России, и текстовые обозначения валют, как они обозначены на портале.  
    
    Все методы класса Vacancy определены в декларативном стиле. Т.к. информация о вакансии получается единожды, а в
    редактировании отдельных полей нет никакого смысла, информация о вакансии нуждается только в первичной обработке
    (избавление от пустых полей, перевод всех з/п к общего виду - в рублях и после вычета налогов), а потом должна
    храниться без изменений.  
    
    Любая постобработка для передачи пользователю в удобочитаемом виде должна происходить в модуле, отвечающем за
    отображение информации в удобном для пользователя виде.
    """  
    def __init__(self, id, title, salary, experience, detailed_information, key_skills):  
        self.id = id  
        self.title = title  
        self.salary = salary  
        self.experience = experience
    # и т. д.
```

2. Класс Book из приложения BookOpinion:
   https://github.com/Paberu/BookOpinion/blob/main/Book.py
```Python
class Book:  
  
    ''' Класс Book используется в приложении для хранения отзывов и/или мнений о прочитанных книгах. Поскольку  
    все поля являются изменяемыми, принято решение реализовать смену полей через открытый метод set_field, который
    принимает имя поля и его новое значение. Для всех остальных данные поля являются приватными, их изменение из
    любого другого класса, включая классы потомков, невозможно.
    
    Для большего удобство создания новых экземпляров из GUI, а так же сохранения в файл/загрузки из файла в классе  
    организованы методы convert_to_dict() и create_from_dict().
	'''  
  
    BOOK_FIELDS = ('_Book__title', '_Book__author', '_Book__writing_year',  
                   '_Book__publication_year', '_Book__rating', '_Book__opinion')  
  
    def __init__(self, title, author, writing_year=None, publication_year=None, rating=None, opinion=None):  
        self.__title = title  
        self.__author = author  
        self.__writing_year = writing_year  
        self.__publication_year = publication_year  
        self.__rating = rating  
        self.__opinion = opinion
    # и т. д.
```

3. Класс Command из Django-приложения Autopark:
https://github.com/Paberu/Autopark/blob/master/park/management/commands/generate_random_objects.py
```Python
class Command(BaseCommand):  
    """ Модуль, с помощью которого создаётся команда generate_random_objects, которая генерирует объекты для наполнения  
    базы данных тестовыми данными. Запускается по аналогии с запуском Django-сервера:
    python manage.py generate_random_objects  
    Принимает ряд аргументов, основные задаются через пробел, дополнительные с помощью ключевых слов. Пример:
    python manage.py generate_random_objects 1 2 4 --car-number 1000 --driver-number 5000  
    Означает: для каждой компании с уникальным ID 1, 2 или 4 сгенерировать 1000 автомобилей и 5000 водителей.
    """ 
    
    help = 'Generates vehicles and drivers for company/companies.'  
  
    def add_arguments(self, parser):  
        parser.add_argument('enterprise_id', nargs='+', type=int)  
  
        # Optional arguments  
        parser.add_argument('--car-number', type=int, help='Number of cars.')  
        parser.add_argument('--driver-number', type=int, help='Number of drivers.')  
  
    def handle(self, *args, **options):
# и т. д.
```  
