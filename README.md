# Проект Оптимизация SQL-запроса

## Необходимо в одном запросе выгрузить следующую информацию о группах пользователей:
ARPU   
ARPAU   
CR в покупку   
СR активного пользователя в покупку   
CR пользователя из активности по математике в покупку курса по математике  

**Данные**  

Таблица **peas** - содержит данные о решении карточек учениками  
| Название атрибута | Тип атрибута | Смысловое значение |
|-------------------|--------------|--------------------|
| `st_id`           | int          | ID ученика |
| `timest`          | timestamp    | Время решения карточки |
| `correct`         | bool         | Правильно ли решена горошина (`true`/`false`) |
| `subject`         | text         | Дисциплина, в которой находится горошина |



Таблица **studs** - таблица студентов  
| Атрибут    | Тип       | Описание                     |
|------------|-----------|------------------------------|
| st_id      | int       | ID ученика                   |
| test_grp   | text      | Метка группы в эксперименте  |



Таблица **final_project_check** - таблица покупок  
| Атрибут    | Тип         | Описание                              |
|------------|-------------|---------------------------------------|
| st_id      | int         | ID ученика                            |
| sale_time  | timestamp   | Время покупки                         |
| money      | int         | Цена курса                            |
| subject    | text        | Дисциплина курса                      |


### SQL-запрос для расчета метрик

За основную таблицу берем studs.  
Джоинами подтягиваем все столбцы из final_project_check и расчетные количества студентов успешно сдавших все предметы и математику из peas.  
Производим расчеты ARPU, ARPAU и CR в покупку.  
Производим расчеты CR активных студентов и CR активных студентов по математике с учетом пустых значений (NULLIF).  

'''SELECT s.test_grp AS test_grp,  
       AVG(COALESCE(fpc.money, 0)) AS arpu,  
AVG(CASE WHEN COALESCE(p.total_correct, 0) > 10 THEN COALESCE(fpc.money, 0) END) AS arpau,  
100.0 * AVG(CASE WHEN fpc.money > 0 THEN 1 ELSE 0 END) AS cr_purchase,  
100.0 * AVG(CASE WHEN COALESCE(p.total_correct, 0) > 10 AND fpc.money > 0 THEN 1 ELSE 0 END) / NULLIF(AVG(CASE WHEN COALESCE(p.total_correct, 0) > 10 THEN 1 ELSE 0 END), 0) AS cr_active,  
100.0 * AVG(CASE WHEN COALESCE(m.math_correct, 0) >= 2 AND fpc.money > 0 THEN 1 ELSE 0 END) / NULLIF(AVG(CASE WHEN COALESCE(m.math_correct, 0) >= 2 THEN 1 ELSE 0 END), 0) AS cr_math_active   
FROM studs AS s  
LEFT JOIN final_project_check AS fpc ON s.st_id = fpc.st_id  
LEFT JOIN (SELECT st_id, COUNT() AS total_correct FROM peas WHERE correct = true GROUP BY st_id) p ON s.st_id = p.st_id  
LEFT JOIN (SELECT st_id, COUNT() AS math_correct FROM peas WHERE correct = true AND subject = 'Math' GROUP BY st_id) m ON s.st_id = m.st_id  
GROUP BY s.test_grp
'''  
