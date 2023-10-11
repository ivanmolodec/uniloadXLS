![uniloadXLS](https://github.com/Bayselonarrend/uniloadXLS/raw/main/uniloadxls.png)
# uniloadXLS
Универсальная загрузка XLS с приведением типов для 1С:Предприятие. <br>
Приведение типов происходит при помощи прогона полученной из файла таблицы через запрос. На данный момент приводятся документы, справочники и перечисления.

Установка:
В решении присутствует код для двух модулей:

||КлиентСерверный|Серверный|
|-|-|-|
|Клиент|+||
|Сервер|+|+|
|Внешнее соединение|+|+|
|Вызов сервера||+|

Этот код необходимо поместить в подходящие модули или создать новые. Функция для вызова - *КлиентСерверный.ЗагрузитьИзXLS*
<Br><br>

### Принцип работы:

```
!!! Пример вызова на форме  !!!

&НаКлиенте
Процедура ЗагрузитьXLS(Команда)
	
	СтруктураКолонок = Новый Структура;
	СтруктураКолонок.Вставить("Спр2"          , "СправочникСсылка.Спр2");
	СтруктураКолонок.Вставить("Перечисление1" , "ПеречислениеСсылка.ТипыСообществ");
	СтруктураКолонок.Вставить("ЛюбоеИмя"      , "");
	СтруктураКолонок.Вставить("Док1"          , "ДокументСсылка.Док1");
	СтруктураКолонок.Вставить("Спр1"          , "СправочникСсылка.Спр1");

	
	ИД = КлиентСерверный.ЗагрузитьИзXLS(СтруктураКолонок,,2);
	
	Если ЗначениеЗаполнено(ИД)  Тогда
		ЗагрузкаНаСервере(ИД);
	КонецЕсли; 
	
КонецПроцедуры


&НаСервере
Процедура ЗагрузкаНаСервере(ИД)	
	ТЗ = ПолучитьИзВременногоХранилища(ИД);
	Реквизит1.Загрузить(ТЗ);
КонецПроцедуры


```

1. Необходимо сформировать структуру колонок, где **Ключ** - название колонки, которое необходимо получить в финальной ТЗ и **Значение** - тип данных строкой (как оно выглядит в поле Тип реквизита). Колонки должны идти в том **порядке** и **количестве**, в котором они идут внутри XLS файла. Если вам необходимо пропустить колонку (например, первую), то достаточно просто вставить в структуру колонок запись с любым ключем и пустым типом. Такие записи обрабатываются как пропуск колонки.

   ||Поле1||Поле2|Поле3|
   |-|-|-|-|-|
   ||Значение||Значение|Значение|
   ||Значение||Значение|Значение|
   ||Значение||Значение|Значение|
  ```
  СтруктураКолонок = Новый Структура;
	СтруктураКолонок.Вставить("Любое"           , "");
	СтруктураКолонок.Вставить("Поле1"           , "ПеречислениеСсылка.Поле1");
	СтруктураКолонок.Вставить("ДругоеЛюбое"     , "");
	СтруктураКолонок.Вставить("Поле2"           , "ДокументСсылка.Поле2");
	СтруктураКолонок.Вставить("Поле3"           , "СправочникСсылка.Поле3");
   ```

2. Вызваемая функция - *КлиентСерверный.ЗагрузитьИзXLS* <br>
     Параметры
   - Структура колонок - из предыдущего пункта
   - Название листа - чтение идет из первого, если не указан
   - Номер первой строки - для пропуска шапки
     
При вызове метода открывается окно выбора файла. Вызов должен быть НаКлиенте. После обработки возвращает адрес временного хранилища, в котором хранится полученная таблица. <br><br>
Обработка происходит следующим образом: по-очереди считываются колонки из файла и записываются в ТЗ по именам из структуры. Затем эта таблица попадает через параметр в запрос, динамически сформированный на основе типов из значений структуры. Документы приводятся по номерам, справочники - по кодам и наименованиям, перечисления - по синониму. Справочник Номенклатура отдельно приводится по реквизиту Артикул. Строковые значения, загруженные из документа, так же сохраняются в финальной таблице на всякий случай.

<br><br>
Пример полученного запроса:
```
ВЫБРАТЬ
	ВходнаяТаблица.Спр2 КАК Спр2,
	ВходнаяТаблица.Спр2КодЧислом КАК Спр2КодЧислом,
	ВходнаяТаблица.Перечисление1НеОбрабатывать КАК Перечисление1,
	ВходнаяТаблица.Док1 КАК Док1,
	ВходнаяТаблица.Док1КодЧислом КАК Док1КодЧислом,
	ВходнаяТаблица.Спр1 КАК Спр1,
	ВходнаяТаблица.Спр1КодЧислом КАК Спр1КодЧислом,
	ВходнаяТаблица.ПорядковыйНомер КАК ПорядковыйНомер
ПОМЕСТИТЬ ВходнаяТаблица
ИЗ
	&ВходнаяТаблица КАК ВходнаяТаблица
;

////////////////////////////////////////////////////////////////////////////////
ВЫБРАТЬ
	Произведения1.Ссылка КАК Спр2,
	Пост1.Ссылка КАК Док1,
	Люди1.Ссылка КАК Спр1,
	ВходнаяТаблица.Спр2 КАК Спр2Служебный,
	ВходнаяТаблица.Перечисление1 КАК Перечисление1,
	ВходнаяТаблица.Док1 КАК Док1Служебный,
	ВходнаяТаблица.Спр1 КАК Спр1Служебный
ИЗ
	ВходнаяТаблица КАК ВходнаяТаблица
		ЛЕВОЕ СОЕДИНЕНИЕ Справочник.Произведения КАК Произведения1
		ПО (ВходнаяТаблица.Спр2 <> "")
			И (ВходнаяТаблица.Спр2 <> 0)
			И (ВходнаяТаблица.Спр2 = Произведения1.Наименование
				ИЛИ ВходнаяТаблица.Спр2КодЧислом = Произведения1.Код
				ИЛИ ВходнаяТаблица.Спр2 = Произведения1.Код)
		ЛЕВОЕ СОЕДИНЕНИЕ Документ.Пост КАК Пост1
		ПО (ВходнаяТаблица.Док1 <> "")
			И (ВходнаяТаблица.Док1 <> 0)
			И ВходнаяТаблица.Док1 = Пост1.Номер
		ЛЕВОЕ СОЕДИНЕНИЕ Справочник.Люди КАК Люди1
		ПО (ВходнаяТаблица.Спр1 <> "")
			И (ВходнаяТаблица.Спр1 <> 0)
			И (ВходнаяТаблица.Спр1 = Люди1.Наименование
				ИЛИ ВходнаяТаблица.Спр1КодЧислом = Люди1.Код
				ИЛИ ВходнаяТаблица.Спр1 = Люди1.Код)

УПОРЯДОЧИТЬ ПО
	ВходнаяТаблица.ПорядковыйНомер
```
Запрос выполняется, выгружается в ТЗ и помещается во временное хранилище, адрес которого возвращается в результате выполнения функции.


>![Infostart](https://github.com/Bayselonarrend/uniloadXLS/raw/main/infostart.svg)
>
>Статья на Инфостарте: [https://infostart.ru/1c/articles/1954271/](https://infostart.ru/1c/articles/1954271/)
