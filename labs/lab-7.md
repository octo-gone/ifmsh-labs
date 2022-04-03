---
layout: default_no_header
title: Лабораторная работа 7
---

> В сессию студент ничего не делает, потому что некогда, а в остальное время — потому что впадлу. Не ругайте, сделайте за него.

## Теория

**Редакторы диаграмм** — специальные инструменты, позволяющие с помощью визуально простого интерфейса и с 
минимального взаимодействия с кодом создать диаграммы, так еще и практически на любую тематику. Возникают вопросы, 
как они работают? 

Естественно разные редакторы работают по-разному, однако исторически сложилось, что данные редакторы
рассматривались для работы с UML (Unified Modeling Language) и с другими похожими. Они унаследовали как принципы, так и общий
формат представления данных для таких диаграмм/моделей. 

Один из таких подходов — использование XML (Extensible Markup Language) формата. **XML** — это не просто формат с фиксированным набором
тегов для данных, нет, это расширяемый язык разметки описывающий грамматику написания документов. Это значит, любой разработчик
может создать собственный тег с аргументами и всем прочим, и при этом документ будет правильным и рабочим. Сочетание отношений Родитель-Ребенок, 
расширяемости языка и простоты синтаксиса позволяет составлять практически любые данные, в том числе и диаграммы. 
Похожий пример, но с фокусом на определенную специфику — HTML.

Перейдем к сути лабораторной. Что если, использовать диаграммы не только для визуализации, но и для выполнения каких-либо действий?
Например, что если мы опишем в формате диаграмм визуальный язык программирования? Что требуется для такой работы?

На занятии мы рассмотрели редактор диаграмм draw.io или же [приложение](https://github.com/jgraph/drawio) для визуализации диаграмм/досок. Основой
файла, хранящего диаграммы оказался XML формат. Однако, точного представления диаграммы не было найдено. Есть непонятная зашифрованная часть и xml-теги.

```xml
<mxfile host="Electron" modified="2022-03-25T16:55:16.297Z"
        agent="5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) draw.io/13.4.5 Chrome/83.0.4103.122 Electron/9.1.0 Safari/537.36"
        etag="HPf6owBOrpI0fvbF3CKK" compressed="true" version="13.4.5" type="device" pages="2">
    <diagram id="HivCTnQHfRtn4HRhNKg_" name="Page-1">
        7Rxpc6...
        ...D+Gw==
    </diagram>
</mxfile>
```

Присмотревшись к наполнению `diagram` мы поняли, что это закодированная информация в формате Base64, 
похожий вид принимала картинка с прошлых заданий, которая тоже была закодирована в Base64. Далее показан код для декодирования: 

```python
import base64

def decode_base64(d):
    return base64.b64decode(d)
```

Однако декодировав с помощью веб-сервисов или программы мы узнали, что этого не достаточно, возможно используется еще 
какая-то кодировка или трансформация данных. Этим кодированием оказался deflate, 
алгоритм сжатия без потерь, это можно было узнать на официальном сайте ~(￣▽￣)~ или из опыта, так как для сжатия Base64 
часто используется данный алгоритм. Для декомпресии требуется использовать библиотеку ориентированную на алгоритмы 
сжатия zlib и соответственно программу:

```python
import zlib

def inflate(d):
    return zlib.decompress(d, -15).decode()
```

```xml
...DAiIHN0cm9rZS13aWR0aD0iMS42IiB3aWR0aD0iMTAwJSIgeD0iMCIgeT0iMCIvPjwvc3ZnPg%3D%3D%3Bpoints%3D%5B%5B0%2C%200.5%5D%2C%20
%5B1%2C%200.5%5D%5D%3BverticalLabelPosition%3Dtop%3BlabelBackgroundColor%3Dnone%3BverticalAlign%3Dbottom%3Baspect
%3Dfixed%3BimageAspect%3D0%3BfontFamily%3DCourier%3BfontSize%3D8%3BlabelPosition%3Dcenter%3Balign%3Dcenter%3Bspacing%3D-1
%3BfontStyle%3D1%3BsyncNodeName%3Dprint%20ctrl%3Bresizable%3D0%3B%22%20parent%3D%221%22%20vertex%3D%221%22%3E%3CmxGeometry
%20x%3D%22240%22%20y%3D%2260%22%20width%3D%2240%22%20height%3D%2240%22%20as%3D%22geometry%22%2F%3E%3C%2FmxCell%3E%3C%2FUserObject
%3E%3C%2Froot%3E%3C%2FmxGraphModel%3E
```

Расшифровав (декодировав) мы опять не получили нужный результат. Какие-то странные символы, ну хотя бы уже есть слова. 
Заметив закономерность, что все специальные символы, как например `<` и `>` были заменены на `%3C` и `%3E`. Такое 
кодирование является экранированием ненадежных символов или же процентное кодирование. Такой подход часто встречается в
интернете, для защиты от неправильного чтения данных. Существуют и аналоги, как например, кодирование через символ `&`.

```python
from urllib import parse

def decode_percent_encoding(d):
    return parse.unquote(d)
```

```xml
<mxGraphModel dx="434" dy="322" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1"
              pageScale="1" pageWidth="827" pageHeight="1169" math="0" shadow="0">
    <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
    ...
    </root>
</mxGraphModel>
```

В итоге мы опять получили XML, значит мы заново можем использовать xml.minidom для парсинга и других взаимодействий.

### Визуальный язык программирования

Раз было решено сделать язык программирования, то воспользуемся уже готовой библиотекой для draw.io с блоками для визуального ЯП. 

<a class="btn-download" href="{{site.baseurl}}/resources/labs/lab-7/base.drawio">Скачать библиотеку</a>  

Допустим левая сторона блока будет определять входы, а правая — выходы (как в логических функциях по ГОСТ). 
На одинарные входы можно подключить только один провод, выходы не ограничены подключениями. 
Работу программы начинает блок Run, конец программы Stop. Функционал остальных блоков будет выбран вами. 

Основной принцип ЯП, что активный блок после завершения вычисления отправляет сигнал на следующий блок. Активный блок пытается
получить данные с предыдущих блоков, если они были вычислены ранее, иначе ожидает. Блоки имеют несколько типов входов: целочисленный (голубой),
дробный (бирюзовый), логический (фиолетовый), строковый (желтый), сигнальный (красный) и любой (серый). 
В остальном принцип аналогичен простым языкам программирования.

## Задача

Требуется реализовать минимальный функционал, описанный на картинке ниже.

<img src="{{site.baseurl}}/resources/labs/lab-7/01_nodes.jpg"/>

Получение диаграммы:

```python
from xml.dom import minidom
import base64
import zlib
from urllib import parse


def decode_base64(d):
    return base64.b64decode(d)


def inflate(d):
    return zlib.decompress(d, -15).decode()


def decode_percent_encoding(d):
    return parse.unquote(d)


def load_structure(filename="scripts.drawio"):
    file = minidom.parse(filename)
    d = file.getElementsByTagName("diagram")[0]
    data = d.firstChild.nodeValue

    decoded_diagram = decode_percent_encoding(inflate(decode_base64(data)))

    return minidom.parseString(decoded_diagram)
```

Выгрузка блоков и связей из диаграммы:

```python
def parse_diagram(d):
    root = diagram.getElementsByTagName("root")[0]

    links = []
    for link in root.getElementsByTagName("mxCell"):

        if link.getAttribute("id") in ["0", "1"]:
            pass
        elif link.parentNode == root:
            link_data = {
                "exitX": None,
                "exitY": None,
                "entryX": None,
                "entryY": None
            }

            for style in str(link.getAttribute("style")).split(";"):
                for key in link_data.keys():
                    if style.startswith(key):
                        link_data[key] = float(style.split("=")[-1])

            link_data.update({
                "id": link.getAttribute("id"),
                "source": link.getAttribute("source"),
                "target": link.getAttribute("target")
            })

            if None in link_data.values():
                print(f"error {link_data['id']}")
                continue

            elif int(link_data["exitX"]) == 0:
                tmp = link_data["source"]
                link_data["source"] = link_data["target"]
                link_data["target"] = tmp

                tmp = link_data["exitY"]
                link_data["exitY"] = link_data["entryY"]
                link_data["entryY"] = tmp

            link_data.pop("exitX")
            link_data.pop("entryX")

            links.append(link_data)

    nodes = []
    for node in diagram.getElementsByTagName("UserObject"):
        inner_data = node.getElementsByTagName("mxCell")[0]

        node_data = {
            "id": node.getAttribute("id"),
            "label": node.getAttribute("label"),
            "name": None
        }

        for style_attribute in str(inner_data.getAttribute("style")).split(";"):
            if style_attribute.startswith("syncNodeName"):
                node_data["name"] = style_attribute.split("=")[-1]

        if node_data["name"] is None:
            continue

        nodes.append(node_data)

    return nodes, links
```

Пример структур данных для блоков:

```python
class Node:
    ids = {}

    def __init__(self, node_data):
        Node.ids[node_data["id"]] = self
        # 0 - not started
        # 1 - working
        # 2 - start in next iteration
        # 3 - evaluated
        self.state = 0
        self.value = None
        self.label = node_data["label"]
        self.inputs = []  # <object Node>, y
        self.outputs = []

    def add_input(self, obj, y):
        self.inputs.append((obj, y))

    def add_output(self, obj, y):
        self.outputs.append((obj, y))

    def __str__(self):
        return f"{self.__class__}: {self.state}"

    def evaluate(self):
        pass


class NodeRun(Node):
    def __init__(self, node_data):
        super(NodeRun, self).__init__(node_data)
        self.state = 2

    def evaluate(self):
        self.state = 3
        for output, y in self.outputs:
            if y == 0.5 and output.state != 1:
                output.state = 2


class NodeConstCtrl(Node):
    def __init__(self, node_data):
        super(NodeConstCtrl, self).__init__(node_data)


class NodePrintCtrl(Node):
    def __init__(self, node_data):
        super(NodePrintCtrl, self).__init__(node_data)


class NodeStop(Node):
    def __init__(self, node_data):
        super(NodeStop, self).__init__(node_data)
```

Пример программы:

```python
if __name__ == "__main__":
    diagram = load_structure()
    nodes, links = parse_diagram(diagram)

    for node in nodes:

        if node["name"] == "run":
            n = NodeRun(node)

        ...

    for link in links:
        source = Node.ids[link["source"]]
        target = Node.ids[link["target"]]

        source.add_output(target, link["exitY"])
        target.add_input(source, link["entryY"])
    
    iter_max = 1000
    for i in range(iter_max):
        pass
        # здесь описывается работа программы на визуальном ЯП
```