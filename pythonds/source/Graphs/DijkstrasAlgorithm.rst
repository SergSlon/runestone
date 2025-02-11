..  Copyright (C)  Brad Miller, David Ranum, Jeffrey Elkner, Peter Wentworth, Allen B. Downey, Chris
    Meyers, and Dario Mitchell.  Permission is granted to copy, distribute
    and/or modify this document under the terms of the GNU Free Documentation
    License, Version 1.3 or any later version published by the Free Software
    Foundation; with Invariant Sections being Forward, Prefaces, and
    Contributor List, no Front-Cover Texts, and no Back-Cover Texts.  A copy of
    the license is included in the section entitled "GNU Free Documentation
    License".

Алгоритм Дейкстры
~~~~~~~~~~~~~~~~~

Для поиска кратчайшего пути мы собираемся использовать так называемый "алгоритм Дейкстры". Он является итеративным и возвращает кратчайшее расстояние от конкретного стартового узла до всех прочих узлов графа - очень похоже на результат поиска в ширину.

Чтобы отслеживать общие затраты на продвижение от начального узла до каждого конечного пункта, мы используем поле ``dist`` класса ``Vertex``. Оно будет содержать текущий общий вес кратчайшего пути от старта до запрашиваемой вершины. Алгоритм повторяется для каждого узла в графе, однако, последовательность итераций задаётся очередью с приоритетом. Для определения порядка объектов в ней используется значение ``dist``. Когда вершина только создаётся, ``dist`` устанавливается в очень большую величину. Теоретически ею должна быть бесконечность, но на практике мы просто выбираем число, большее любого реального расстояния, которое будет использоваться при решении задачи.

Код алгоритма Дейкстры показан в :ref:`листинге 1 <lst_shortpath>`. Результатом его работы станут правильно установленные расстояния, а также ссылки на предшественника для каждой вершины графа.

.. _lst_shortpath:

**Листинг 1**

::

    from pythonds.graphs import PriorityQueue, Graph, Vertex
    def dijkstra(aGraph,start):
        pq = PriorityQueue()
        start.setDistance(0)
        pq.buildHeap([(v.getDistance(),v) for v in aGraph])        
        while not pq.isEmpty():
            currentVert = pq.delMin()
            for nextVert in currentVert.getConnections():
                newDist = currentVert.getDistance() \
                        + currentVert.getWeight(nextVert)
                if newDist < nextVert.getDistance():
                    nextVert.setDistance( newDist )
                    nextVert.setPred(currentVert)
                    pq.decreaseKey(nextVert,newDist)

Алгоритм Дейкстры использует очередь с приоритетом. Вы можете помнить, что она основывается на куче, которую мы создали в главе, посвящённой деревьям. Однако, между тем простым вариантом и реализацией для алгоритма Дейкстры есть несколько отличий. Во-первых, класс ``PriorityQueue`` сохраняет кортежи пар ключ-значение. Это очень важно для алгоритма Дейкстры, поскольку ключи в очереди с приоритетом должны быть связаны с ключами вершин графа. Во-вторых, значение используется для расстановки приоритетов, а следовательно и для определения позиции ключа в очереди. В этой реализации в качестве индикатора приоритета используется расстояние до вершины, покольку (как будет показано далее) для исследования мы всегда будем стремиться найти узел с наименьшим расстоянием. Следующее отличие между предыдущей и новой версиями - в дополнительном методе ``decreaseKey``. Как вы можете видеть, он используется, когда нужно уменьшить расстояние до вершины, уже стоящей в очереди, т.е. переместить её ближе к началу.

Давайте пройдём по алгоритму Дейкстры для одного из узлов, используя в качестве путеводителя нижеследующие рисунки. Начнём с вершины :math:`u`. С нею смежны три узла: :math:`v, w` и :math:`x`. Поскольку начальные расстояния для них инициализированы ``sys.maxint``, то расстояниями пути к ним от стартового узла станут их непосредственные веса. Вместе с обновлением издержек мы устанавливаем предшественником каждого узел :math:`u` и добавляем их в очередь. Расстояние будет использоваться как ключ приоритета. Состояние алгоритма на этот момент показано на :ref:`рисунке 3 <fig_dija>`.

На следующей итерации цикла ``while`` мы проверяем вершины, смежные с :math:`x`. Она выбрана следующей, поскольку имеет наименьшее значение ``dist`` и, следовательно, оказалась наверху очереди с приоритетом. Для :math:`x` мы рассматриваем её соседей :math:`u, v, w` и :math:`y`. Для каждого из них расчитываем путь через :math:`x` и смотрим, у кого он будет меньше существующей величины ``dist``. Очевидно, что это вариант :math:`y`, поскольку его расстояние - ``sys.maxint``, а не :math:`u` или :math:`v`, поскольку их значения 0 и 2 соответственно. Как бы то ни было, теперь мы знаем, что расстояние до :math:`w` будет наименьшим, если идти к ней через :math:`x`, а не непосредственно от :math:`u` к :math:`w`. Таким образом, мы обновляем расстояние :math:`w` и изменяем её предшественника с :math:`u` на :math:`x`. См. :ref:`рисунок 4 <fig_dijb>`, где показано состояние всех вершин на данный момент.

Следующим шагом станет рассмотрение соседей :math:`v` (см. :ref:`рисунок 5 <fig_dijc>`). Его результаты не изменят граф, так что мы переходим к узлу :math:`y`. Для него (см. :ref:`рисунок 6 <fig_dijd>`) мы находим, что дешевле пройти через :math:`w` и :math:`z`, в связи с чем регулируем соответствующие расстояния и ссылки на предшественников. Наконец, проверяем узлы :math:`w` и :math:`z` (см. :ref:`рисунок 7 <fig_dije>` и :ref:`рисунок 8 <fig_dijf>`). Однако, никаких дополнительных изменений это не вносит, очередь с приоритетом пуста и алгоритм Дейкстры заканчивает свою работу.

.. _fig_dija:

.. figure:: Figures/dijkstraa.png
   :align: center

   Рисунок 3: Трассировка алгоритма Дейкстры      
   
.. _fig_dijb:

.. figure:: Figures/dijkstrab.png
   :align: center

   Рисунок 4: Трассировка алгоритма Дейкстры      
   
.. _fig_dijc:

.. figure:: Figures/dijkstrac.png
   :align: center

   Рисунок 5: Трассировка алгоритма Дейкстры       
   
.. _fig_dijd:

.. figure:: Figures/dijkstrad.png
   :align: center

   Рисунок 6: Трассировка алгоритма Дейкстры      
   
.. _fig_dije:

.. figure:: Figures/dijkstrae.png
   :align: center

   Рисунок 7: Трассировка алгоритма Дейкстры       
   
.. _fig_dijf:

.. figure:: Figures/dijkstraf.png
   :align: center

   Рисунок 8: Трассировка алгоритма Дейкстры 

Важно отметить, что алгоритм Дейкстры работает только в том случае, когда веса рёбер положительны. Вы можете самостоятельно проверить, что если хотя бы у одного из них будет отрицательный вес, то алгоритм никогда не завершится.

Отметим, что для прокладывания маршрута сообщения через интернет используются и другие алгоритмы поиска кратчайшего пути. Одной из проблем с алгоритмом Дейкстры для Всемирной паутины является необходимость иметь полное представление графа. Только в этом случае он может быть запущен. Следствием этого будет то, что каждый маршрутизатор должен обладать полной картой всех маршрутизаторов в интернете. На практике такое не встречается, поэтому существуют дргие варианты алгоритма, позволяющие маршрутизатору открывать граф по ходу дела. Один из них, о котором вы, возможно, захотите почитать, называется "дистанционно-векторный алгоритм маршрутизации".
