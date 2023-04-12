# Задание на аналитику новой корзины

### Контекст

Интернет-магазин федеральной сети. Гипотеза – увеличить количество заказов на сайте, благодаря новой корзине. Корзина была полностью переделана и предлагает новый опыт оформления доставки (новый ui способов получения, новая карта выбора точки получения заказа и т.п.). 

Ваша задача понять какие изменения произошли с экономикой. Заказчика интересует как изменились верхнеуровневые экономические показатели сайта. И если они изменились, то объяснить их изменение с помощью метрик уровнями ниже.

### Сетап эксперимента

- A1 25% (0) / A2 25% (1) / Б 33% (2) / С 33% (3)
    - в скобках указан id варианта
- Уровень значимости и мощности берите 95% и 80%, соответственно

### Задача

- Какие бы метрики вы предложили для анализа этого эксперимента? Составьте иерархию метрик и посчитайте их
- Если какие-то метрики не показали стат. значимый результаты, то сколько еще требуется ждать?

### Мое решение

**Определяем метрики** 

Главная потребность бизнеса сформулирована как “увеличить количество заказов на сайте”. Но в качестве ключевой метрики я предлагаю все же выбрать средние траты на пользователя, потому что она лучше описывает экономическую составляющую и в большинстве случаев коррелирует со средним количеством заказов на пользователя.

Чтобы погрузиться глубже в экономику сайта я предлагаю изучить конверсию в покупку, среднее количество заказов на пользователя и среднюю стоимость заказа каждого пользователя в группе контроля и воздействия. Из поведенческих метрик также думаю стоит посомтреть на retention в покупку. Также стоит изучить количество ошибок при оформлении заказа в тестовой и контрольной группах.

Итого иерархию метрик я представляю следующим образом:

----

Экономика:

Cредние траты на пользователя

- Метрики эффективности
    - количество заказов на пользователя
    - средняя стоимость заказа
- Метрики мониторинга ошибок
    - количество ошибок при оформлении заказа
- Поведенческие метрики
    - retention


----

**Перейдем к рассчетам**

Изучим заполненность колонок данными, и увидим что есть абсолюно пустые колонки - удалим поля, которые не используются, а также 'timestamp', 'payment_type', 'charity_points’ из за низкого наполнения + experiment_name, тк оно содержит только одно значение.

Проверим группы на пересечения - пересечений нет. Теперь определим какие величины будут нужны для расчета метрик и расчитаем их для теста и контроля.


#### **Средние траты на пользователя**
- смотрим на все event’ы purchase, где pg_status = ‘REALIZATION’ и считаем сумму revenue в расчете на одного покупателя. Для проверки на статзначимость будем использовать t-test, поскольку нам нужно сравнить средние значения прибыли, распределение которых при большом количестве наблюдений будет стремиться к нормальному (по ЦПТ).

Заметим что для второй вариации различия являются статзначимыми, средняя стоимость заказов выросла с 2681 до 2759 (на 2,9%)

```
Различия между тестом (2759) и контролем (2681) в средних тратах на пользователя являются статзначимыми.
p-value: 0.01
```

а для третьей не являются статзначимыми

```
Различия между тестом (2677) и контролем (2681) в средних тратах на пользователя не являются статзначимыми.
p-value: 0.89
```
Причем группа Б статзначимо лучше группы С:

```
Различия между тестом (2759) и контролем (2677) в средних тратах на пользователя являются статзначимыми.
p-value: 0.01
```

Верхнеуровнево видим, что изменения корзины положительно повлияли на метрику средних трат на клиента в группе Б, чтобы убедиться в том, что это является индикатором роста экономики сайта, изучим более низкоуровневые метрики. В группе C статистически значимых изменений не произошло.

----

#### **Количество заказов на пользователя**
- смотрим на все event’ы purchase, где pg_status = ‘REALIZATION’ и считаем количество покупок в расчете на одного покупателя.

В группе Б различия оказались статзначимыми - клиенты стали покупать на 1,3% чаще

```
Различия между тестом (1.56) и контролем (1.54) в среднем количестве заказов на пользователя являются статзначимыми.
p-value: 0.01
```
В группе C различия оказались не статзначимыми:

```
Различия между тестом (1.53) и контролем (1.54) в среднем количестве заказов на пользователя не являются статзначимыми.
p-value: 0.25
```

Здесь тоже группа Б статзначимо опережает группу С:

```
Различия между тестом (1.56) и контролем (1.53) в среднем количестве заказов на пользователя являются статзначимыми.
p-value: 0.0
```

Далее изучим изменение средней стоимости заказа.

----

#### **Средняя стоимость заказа**
- смотрим на все event’ы purchase, где pg_status = ‘REALIZATION’ и считаем стоимость заказов.

Оказалось что в группе Б выросла средняя сумма заказа с 1743 до 1767 (на 1,4%)

```
Различия между тестом (1768) и контролем (1743) в средней стоимости заказов являются статзначимыми.
p-value: 0.05
```
А в группе С опять нет статзначимости:

```
Различия между тестом (1753) и контролем (1743) в средней стоимости заказов не являются статзначимыми.
p-value: 0.45
```

Здесь тестовые группы статзначимо не отличаются:

```
Различия между тестом (1768) и контролем (1753) в средней стоимости заказов не являются статзначимыми.
p-value: 0.24
```

Итак, в группе Б средние траты выросли как за счет увеличения чиса заказов, так и за счет увеличения средней стоимости одного заказа.

----

#### **Среднее число ошибок**
- смотрим на все event’ы ‘purchase_error’ и считаем среднее число ошибок на пользователя.

Оказалось что в группе Б среднее число ошибок статзначимо меньше чем в группе С:

```
Различия между тестом (1.48) и контролем (2.41) в среднем числе ошибок являются статзначимыми.
p-value: 0.0
```

Таким образом, версия сайта а варианте Б показала наибольшие приросты в средних тратах пользователя по сравнению с контролем (средняя стоимость заказов выросла с 2681 до 2759 (на 2,9%)) и вариантом С (средняя стоимость заказов выросла с 2759 до 2677)). Это произошло как за счет повышения среднего числа заказов в группе Б (с 1.54 до 1.56), так и за счет повышения средней стоимости заказа (с 1743 до 1768). Помимо этого пользователи версии Б реже сталкивались с ошибками сайта при заказе.
Версия С не показала статзначимые улучшения по сравнению с контрольной группой в вопросе экономики.
