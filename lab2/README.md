# Лабораторная 2

1. Чтение файлов

programming_languages = sc.read.csv("programming-languages.csv")
programming_languages = [str(x[0]).lower() for x in programming_languages.collect()]

post_sample = sc.read.format("xml").options(rowTag="row").load("posts_sample.xml")


Первые 10 языков:

['a# .net',
 'a# (axiom)',
 'a-0 system',
 'a+',
 'a++',
 'abap',
 'abc',
 'abc algol',
 'abset',
 'absys']

Первый пост:

Row(_AcceptedAnswerId=7, _AnswerCount=13, _Body="<p>I want to use a track-bar to change a form's opacity.</p>\n\n<p>This is my code:</p>\n\n<pre><code>decimal trans = trackBar1.Value / 5000;\nthis.Opacity = trans;\n</code></pre>\n\n<p>When I build the application, it gives the following error:</p>\n\n<blockquote>\n  <p>Cannot implicitly convert type <code>'decimal'</code> to <code>'double'</code></p>\n</blockquote>\n\n<p>I tried using <code>trans</code> and <code>double</code> but then the control doesn't work. This code worked fine in a past VB.NET project.</p>\n", _ClosedDate=None, _CommentCount=2, _CommunityOwnedDate=datetime.datetime(2012, 10, 31, 20, 42, 47, 213000), _CreationDate=datetime.datetime(2008, 8, 1, 2, 42, 52, 667000), _FavoriteCount=48, _Id=4, _LastActivityDate=datetime.datetime(2019, 7, 19, 5, 39, 54, 173000), _LastEditDate=datetime.datetime(2019, 7, 19, 5, 39, 54, 173000), _LastEditorDisplayName='Rich B', _LastEditorUserId=3641067, _OwnerDisplayName=None, _OwnerUserId=8, _ParentId=None, _PostTypeId=1, _Score=630, _Tags='<c#><floating-point><type-conversion><double><decimal>', _Title='Convert Decimal to Double?', _ViewCount=42817)

Описание: csv файл можно прочитать с использованием методов read.csv, а xml файл можно прочитать с использованием того же метода read, но с указанием формата - format("xml), построчно - options(rowTag="row"), указание файла - load("path").


2. Обработка данных

from pyspark.sql.functions import col

def find_language(x):
    for l in programming_languages:
        if "<" + l + ">" in x._Tags.lower():
            return (x._Id, l)
    return None

def isValid(x, y):
    if x._Tags is None:
        return False
    return x._CreationDate>=datetime(year=y, month=1, day=1) and datetime(year=y, month=12, day=31)>=x._CreationDate
    

res = {}
for y in range(2010,2020):
    res[y]=(post_sample.rdd.filter(lambda x: isValid(x,y))\
               .map(find_language).filter(lambda x: x is not None).keyBy(lambda x: x[1])\
               .aggregateByKey(0,lambda x,y:x+1, lambda d,z:d+z).sortBy(lambda x:x[1],ascending=False).toDF())\
               .select(col("_1").alias("Language"), col("_2").alias("Year_{0}".format(y))).limit(10)
    res[y].show()



Описание: для каждого года в промежутке от 2010 до 2019 исходная выборка постов проходит фильтрацию по валидности даты (с использованием вспомогательной функции isValid), затем с помощью метода map отбираем пары (id, language), создаем tuple  с помощью keyBy, затем подсчитываем общее число постов для каждого языка и сортируем по названию языка. 
С помощью метода toDF и с использованием col форматируем полученные результаты.

Примеры вывода:

+-----------+---------+     +-----------+---------+     +-----------+---------+    +-----------+---------+       
|   Language|Year_2010|     |   Language|Year_2011|     |   Language|Year_2012|    |   Language|Year_2013| 
+-----------+---------+     +-----------+---------+     +-----------+---------+    +-----------+---------+ 
|       java|       52|     |        php|       97|     |        php|      136|    | javascript|      196| 
| javascript|       44|     |       java|       92|     | javascript|      129|    |       java|      191| 
|        php|       42|     | javascript|       82|     |       java|      124|    |        php|      173| 
|     python|       25|     |     python|       35|     |     python|       65|    |     python|       87| 
|objective-c|       22|     |objective-c|       33|     |objective-c|       45|    |objective-c|       40| 
|          c|       20|     |          c|       24|     |          c|       27|    |          c|       36| 
|       ruby|       11|     |       ruby|       17|     |       ruby|       25|    |       ruby|       30| 
|     delphi|        7|     |       perl|        8|     |       bash|        9|    |          r|       25| 
|applescript|        3|     |     delphi|        8|     |          r|        9|    |       bash|       11| 
|          r|        3|     |       bash|        7|     |     matlab|        6|    |      scala|       10| 
+-----------+---------+     +-----------+---------+     +-----------+---------+    +-----------+---------+ 

3. Отчёт сохранить в формате Apache Parquet

for i in result.keys():
    result[i].write.format("parquet").save("{0}".format(i))

Описание: для каждого года сохраняем результат в формате parquet.