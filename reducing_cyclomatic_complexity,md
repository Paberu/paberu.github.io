# Снижение цикломатической сложности
Цикломатическая сложность - это метрика, которая измеряет количество путей выполнения в программе и может быть использована для оценки сложности кода. Снижение цикломатической сложности улучшает читаемость, тестируемость и поддерживаемость кода. Недавно узнал и решил показать на примерах: один из библиотеки, которую использую, второй - из собственного проекта.

## Избавление от else и отказ от None.
Самое начало замечательной библиотеки DearPyGui вызывает сильнейшее недоумение. Служебное слово `pass` в Python - это заглушка: нельзя оставлять функцию пустой, и если уж мы начали прописывать интерфейс класса, то сначала можно натолкать заглушек, а потом их постепенно реализовывать.
[https://github.com/hoffstadt/DearPyGui/blob/master/dearpygui/dearpygui.py](https://github.com/hoffstadt/DearPyGui/blob/master/dearpygui/dearpygui.py)
```Python
def run_callbacks(jobs):
    """ New in 1.2. Runs callbacks from the callback queue and checks arguments. """

    if jobs is None:
        pass
    else:
        for job in jobs:
            if job[0] is None:
                pass
            else:
                sig = inspect.signature(job[0])
                args = []
                for arg in range(len(sig.parameters)):
                    args.append(job[arg+1])
                job[0](*args)
```
Мой вариант данной функции:
```Python
def run_callbacks(jobs):
    """ New in 1.2. Runs callbacks from the callback queue and checks arguments. """

    if not jobs:
	    return
	    
	for job in jobs:
		if not job[0]:
			continue
			
		sig = inspect.signature(job[0])
		args = []
		for arg in range(len(sig.parameters)):
			args.append(job[arg+1])
		job[0](*args)
```

## Избавление от else и "табличная" логика.

Класс, созданный мной в рамках дипломного проекта по Django:
```Python
class VehicleInfoView(APIView):
    # некоторое количество адекватного удобочитаемого кода
    
	# функция get(), в которой стоит убрать if..else
    def get(self, request):
        if request.user.is_superuser:
            vehicles = Vehicle.objects.all()
        elif Manager.objects.filter(user=request.user):
            mngr = Manager.objects.filter(user=request.user)[0]
            enterprises = mngr.enterprise.all()
            vehicles = Vehicle.objects.filter(enterprise__in=enterprises)
        else:
            print('Error in server logic, should have failed on "check_permissions" stage.')
            self.permission_denied(request, message='Error code: 401', code=401)

        page = request.GET.get('page', 1)
        paginator = Paginator(vehicles, 20)
        try:
            vehicles = paginator.page(page)
        except PageNotAnInteger:
            vehicles = paginator.page(1)
        except EmptyPage:
            vehicles = paginator.page(paginator.num_pages)
        serialized_vehicles = VehicleSerializer(instance=vehicles, many=True)
        return Response(serialized_vehicles.data)
```

Я решил выделить проверку доступности автомобилей в отдельную функцию.
```Python
class VehicleInfoView(APIView):
    # некоторое количество адекватного удобочитаемого кода
    def check_vehicle_access(request, object_type):
		if request.user.is_superuser:
	        return Vehicle.objects.all()
	    if Manager.objects.filter(user=request.user):
			mngr = Manager.objects.filter(user=request.user)[0]
			enterprises = mngr.enterprise.all()
			return Vehicle.objects.filter(enterprise__in=enterprises)
		print('Error in server logic, should have failed on "check_permissions" stage.')
		self.permission_denied(request, message='Error code: 401', code=401)

	# функция get(), в которой стоит убрать if..else
    def get(self, request):
        vehicles = check_vehicle_access(request, Vehicle)
        page = request.GET.get('page', 1)
        paginator = Paginator(vehicles, 20)
        try:
            vehicles = paginator.page(page)
        except PageNotAnInteger:
            vehicles = paginator.page(1)
        except EmptyPage:
            vehicles = paginator.page(paginator.num_pages)
        serialized_vehicles = VehicleSerializer(instance=vehicles, many=True)
        return Response(serialized_vehicles.data)
```

И это не предел. Классов, работа с которыми доступна через APIView, много, поэтому функцию `check_vehicle_access()` стоит превратить функцию `check_object_access()` (и вынести за пределы всех классов). Я передаю ссылку на сам класс `VehicleInfoView` (или любой другой, использующий `check_object_access`), чтобы грамотно отработать `permission_denied` (которая, в свою очередь, является обёрткой на `raise exceptions...`). Я передаю в эту функцию ссылку на `request`, чтобы взять оттуда текущего пользователя. Я передаю тип объекта `object_type`, который позволяет через `objects` обращаться к `all(), filter()` и прочим функциям.
```Python
def check_object_access(self, request, object_type):  
    check_list = [Vehicle, Driver, Enterprise,] # и далее список всего, что подлежит проверке. Зачем? Вот за этим:  
    if object_type not in check_list:  
        print('[ERR] Wrong class, check your code.')  
        self.permission_denied(request, message='Error code: 401', code=401)  
    if request.user.is_superuser:  
        return object_type.objects.all()  
    if Manager.objects.filter(user=request.user):  
        mngr = Manager.objects.filter(user=request.user)[0]  
        enterprises = mngr.enterprise.all()  
        return object_type.objects.filter(enterprise__in=enterprises)  
    print('Error in server logic, should have failed on "check_permissions" stage.')  
    self.permission_denied(request, message='Error code: 401', code=401)

class VehicleInfoView(APIView):
    # некоторое количество адекватного удобочитаемого кода
    
    def get(self, request):
        vehicles = check_object_access(self, request, Vehicle)
        page = request.GET.get('page', 1)
        paginator = Paginator(vehicles, 20)
        try:
            vehicles = paginator.page(page)
        except PageNotAnInteger:
            vehicles = paginator.page(1)
        except EmptyPage:
            vehicles = paginator.page(paginator.num_pages)
        serialized_vehicles = VehicleSerializer(instance=vehicles, many=True)
        return Response(serialized_vehicles.data)
```
Как таковой, таблицы с классами нет, есть банальный список. Но принцип действия понятен: при наличии схожих (пусть и не одинаковых) методов в разных классах полезно вынести их во внешнюю функцию. Не только с точки зрения удобства и снижения объема кода, но и с точки зрения логики проектирования - все проверки собраны в одном месте, где их легче сопровождать, расширять и т.д.