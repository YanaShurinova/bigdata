# Лабораторная 1

1. Найти велосипед с максимальным временем пробега

bike_max_duration_id = trip_internal.map(lambda x: (x.bike_id, x.duration)) \
                                    .reduceByKey(lambda a,b:a+b) \
                                    .reduce(lambda a,b: a if a[1]>b[1] else b)[0]

Результат: 535

Описание: с помощью метода map получаем из исходной выборки пары типа (bike_id, duration) - (id, продолжительность),
затем складываем все поездки по bike_id, получая в итоге пары (id, суммарная продолжительность),
затем сравниваем все пары и выбираем ту, у которой суммарная по всем поездкам продолжительность больше, таким образом определяя пару из id и максимальной суммарной продолжительности.

2. Найти наибольшее геодезическое расстояние между станциями.

def distance(x,y):
    return ((x.lat-y.lat)**2+(x.long-y.long)**2)**0.5

max_distance = station_internal.cartesian(station_internal).map(lambda x:  distance(x[0], x[1])).max()

Результат: 0.705848

Описание: с помощью метода cartesian выбираем все возможные комбинации станций и затем с помощь метода map и функции distance, вычисляем расстояние для каждой пары и выбираем максимальное

3. Найти путь велосипеда с максимальным временем пробега через станции.

path_max_duration = trip_internal.filter(lambda x:x.bike_id==bike_max_duration_id) \ 
                                .sortBy(lambda x: x.start_date) \
                                .map(lambda x: x.end_station_name).distinct().collect()

Результат: ['San Francisco Caltrain 2 (330 Townsend)', 'Market at Sansome', '2nd at South Park', 'Davis at Jackson', 'Post at Kearney', 'Embarcadero at Sansome', 'Clay at Battery', 'Harry Bridges Plaza (Ferry Building)', 'Steuart at Market', 'Townsend at 7th', 'Powell at Post (Union Square)', 'Market at 4th', 'Beale at Market', 'Powell Street BART', 'San Francisco City Hall', 'Embarcadero at Vallejo', 'Yerba Buena Center of the Arts (3rd @ Howard)', 'Howard at 2nd', 'Commercial at Montgomery', 'Grant Avenue at Columbus Avenue', 'Broadway St at Battery St', 'Post at Kearny', 'San Francisco Caltrain (Townsend at 4th)', 'Spear at Folsom', 'Temporary Transbay Terminal (Howard at Beale)', '5th at Howard', 'Civic Center BART (7th at Market)', 'Market at 10th', '2nd at Folsom', 'South Van Ness at Market', 'Mechanics Plaza (Market at Battery)', 'Embarcadero at Folsom', '2nd at Townsend', 'Embarcadero at Bryant', 'Golden Gate at Polk', 'Washington at Kearny', 'Washington at Kearney']


Описание: из исходной выборки выбираем поездки, в которых идентификатор велосипеда совпадает с подсчитанным ранее - идентификатором велосипеда с максимальным временем пробега, затем сортируем по дате начала поездки и с помощью метода map выбираем название конечной станции.

4. Найти количество велосипедов в системе.

bike_count = trip_internal.map(lambda x: x.bike_id).distinct().count()

Результат: 700

Описание: отбираем из исходной выборки все идентификаторы велосипедов, затем убираем повторы и с помощью метода count подсчитываем их общее число.

5. Найти пользователей потративших на поездки более 3 часов.

clients = trip_internal.map(lambda x: (x.zip_code, x.duration)) \ 
                        .reduceByKey(lambda a,b: a+b) \ 
                        .filter(lambda x: x[1]>3*60*60) \ 
                        .map(lambda x:x[0])

Первые 10 пользователей: 
['95138',
 '95060',
 '95112',
 '94041',
 '94122',
 '94117',
 '95819',
 '94114',
 '94102',
 '94612']

Число таких пользователей: 3661

Описание: с помощью метода map из исходной выборки отбираем пары (zip_code, duration) - почтовый индекс пользователя и продолжительность поездки; затем складываем все поездки для каждого пользователя и с помощью метода filter находим пары, в которых суммарная продолжительность больше трех часов, затем из пар отбираем только почтовый индекс пользователя.