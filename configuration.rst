.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov  5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.
  
Структура конфигурации
===============================
  
Конфигурация – это JSON-файл, который задает поведение системы на клиенте и сервере. На сервере – конфигурация это по большей части – набор классов узлов. Узлы хранятся и выполняются по мере обращения к ним со стороны клиентов и внешней системы. А для мобильного клиента в конфигурации задаются многие другие свойства.

Какие разделы есть в конфигурации

 * Общие свойства (Название конфигурации, поставщик, версия и т.д.)
 * Разделы. То, на каких вкладках в интерфейсе располагаются узлы и процессы. Подробнее в разделе Мобильный клиент
 * Классы. Структура классов данных – и серверных и клиентских. Подробнее в разделе Классы
 * Общие события. События, возникающие в системе вне классов узлов, т.е. в приложении в целом. Подробнее в подразделе «Общие события мобильного клиента»
 * Разделы с python-обработчиками для классов и общих событий. Это python-файлы, преобразованные в base64-формат, хранящиеся в структуре конфигурации. Доставляются на целевое устройство вместе с обновлением конфигурации и преобразуются в исполняемые временные файлы.
 * Датасеты. Хранение и доставка справочников и ссылок внешней системы на устройство. Подробнее в разделе Датасеты
 * Серверы – раздел для регистрации псевдонимов серверов, использующихся в функциях API

Описание формата файла
-----------------------------
  
Некоторые из полей этого раздела формируются автоматически конструктором, но в случае формирования файла вне контрактора (например LLM) все поля необходимо будет заполнять согласно принципам, описанным ниже

Общие свойства конфигурации 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**"name"** – имя конфигурации

**"uid"** – ИД установки на сервере, через который организуется доступ к объектам конфигурации для API и хранение

**"url"** – полный адрес хостинга, с которого клиенты будут скачивать конфигурацию(конфигурация изначально может быть попасть на клиент с файла, но для обновления он будет обращаться по этому адресу)

**"content_uid"** – уникальный ID конфигурации независимо от ИД уставновки. Одна и та же конфигурация может быть развернута под разными uid, но content_uid останется тем же

**"vendor"** имя поставщика конфигурации

**"version"** – номер версии. 

**"NodaLogicFormat"** – версия формата. Анализируется приложением. Если приложение не поддерживает этот формат, будет предложено обновить приложение

**"NodaLogicType": "ANDROID_SERVER"** – константа

Обработчики
~~~~~~~~~~~~~~~~~~

**"nodes_handlers"** и **"nodes_server_handlers"** – base64 закодированные файлы обработчиков. Они должны быть оформлены по определенному стандарту – верхняя часть содержим импорты и константы, которые автоматически генерируются системой. Далее идут классы и пользовательский код. В current_module_name прописывается url инстанса, в current_configuration_url – путь к API инстанса. Т.е. поля uid и url

Для андроид это:

.. code-block:: Python

 from nodesclient import RefreshTab,SetTitle,CloseNode,RunGPS,StopGPS,UpdateView,Dialog,ScanBarcode,GetLocation,AddTimer,StopTimer,ShowProgressButton,HideProgressButton,ShowProgressGlobal,HideProgressGlobal,Controls,SetCover,getBase64FromImageFile,convertImageFilesToBase64Array,saveBase64ToFile,convertBase64ArrayToFilePaths,UpdateMediaGallery
 from android import *
 from nodes import NewNode, DeleteNode, GetAllNodes, GetNode, GetAllNodesStr, GetRemoteClass, CreateDataSet, GetDataSet, DeleteDataSet,to_uid, from_uid
 from com.dv.noda import DataSet
 from com.dv.noda import DataSets
 from com.dv.noda import SimpleUtilites as su
 from datasets import GetDataSetData
 
 # Константы конфигурации
 current_module_name="296f962e-bd0d-4b32-8ac8-3bc9a1162a56"
 current_configuration_url="http://nmaker.pw/api/config/296f962e-bd0d-4b32-8ac8-3bc9a1162a56"
 _data_dir = su.get_data_dir(current_module_name)
 _downloads_dir = su.get_downloads_dir(current_module_name)
 
 
 from nodes import Node
 
Для server это:

.. code-block:: Python

 from nodes import Node

Классы
~~~~~~~~

**"classes"** – массив классов

Каждый класс должен присутствовать и в массиве classes и также быть в обработчике, в зависимости от того где он будет выполняться. Либо в "nodes_handlers" либо в     "nodes_server_handlers"

Класс имеет ключи:

 * **"name"** – имя, оно же идентифкатор класса. Должно быть python-совместимым (так как на этот класс в python также существует класс)
 * **"section_code"** и **"section"** – код раздела и название раздела (из раздела конфигурации sections) к которому отностится класс
 * **"has_storage"** – автосохранение данных при вводе в поля ввода
 * **"display_name"** – отображаемое имя, елси не используется «обложка»
 * **"cover_image"** – обложка в формате разметки, таком же как для экранов и люых мест где используется разметка в Андроид- клиенте
 * **"hidden"** – класс (если это класс-процесс) будет скрыт для отображения в интерфейсе)
 * **"class_type"** – доступно либо "custom_process" – узел-процесс, существующий в единственно экземпляре, создается вместе с загрузкой конфигурации либо "data_node" – узел данных, т.е. объект данных создаваемый или загружаемый в систему

  **"methods"** – массив методов класса. Методы должны быть описаны и в массиве и в обработчиках.

Ключи метода:

 * **"name"** и тоже самое в **"code"** (для совместимости)– имя метода
 * **"source": "internal"**
 * **"engine"** : либо "android_python" либо "engine": "server_python" в зависимости от того, где планируется выполнять метод
 * **"events"** – массив событий, которые возникают в системе – открытие формы, действия пользователя

Ключи события:

 * **"event"** – может быть "onShow" (при открытии формы узла), "onResume" ,"onInput" – любое событие ввода от пользователя или внешнее событие
 * **"listener"** – фильтр по "listener". События ввода как правило содержат дополнительное поле "listener", уточняющее событие. Например, id нажатой кнопки. Если не использовать "listener" то все события будут попадать на этот обработчик
 * **"actions"** – массив действий, который надо выполнить на событие. На одно и то же событие можно повесить несколько действий, но обычно оно одно.
   
Ключи действия: 
   
 * **"action"** -"run"(синхронное выполнение), "runasync" – асинхронное выполнение, "runprogress" – синхронное с прогрессбаром.
 * **"source"** : "internal"
 * **"method"** : имя метода из массива "methods"
 * **"postExecuteMethod"** – имя метода, который может быть выполнен после окончания выполнения основного метода. Актуально для асинхронного выполнения и выполнение с прогрессбаром.

Помимо прописывания в classes класс узла также необходимо разместить в обработчиках, используя то же имя и те же имена методов и соблюдая определенный формат описания – класс родитель, метод __init__.
   
Для Андроид пример:
   
.. code-block:: Python

 class MyClass(Node):
    def __init__(self, modules, jNode, modulename, uid, _data):
        super().__init__(modules, jNode, modulename, uid, _data)

    """Class MyClass"""

    def Open(self, input_data=None):
        self.Show(
          [
            [{"type":"Input","id":"input1","caption":"input 1","value":"@input1"}],
            [{"type":"Button","id":"button1","caption":"Get result"}]
          ]
        )

        return True,{}
    def Input(self, input_data=None):
        toast(self._data["input1"])
        
        return True,{}
             
Для сервера:

.. code-block:: Python
              
 class MyClass(Node):
    
    def __init__(self, node_id=None, config_uid=None):
        super().__init__(node_id, config_uid)

Также при оформлении методов класса нужно придерживаться единого формата , описанного в разделах Классы и Общие обработчики. Это параметры метода и структура возвращаемого кортежа. 
Для методов класса:
 
.. code-block:: Python

 def Input(self, input_data=None):
        
        return True,{}

Для общих обработчиков:
            
.. code-block:: Python

 def onStartConfiguration(input_data):

  return True,{}

Секция Разделы
~~~~~~~~~~~~~~~~            
            
**"sections"** – массив разделов
            
**"name"** – отображаемое имя раздела
            
**"code"** – ИД раздела
            
**"commands"** – список команд  в виде списка через запятую <Заголовок команды>|<ид команды>. При нажатии на кнопку генерируется общее событие onStartMenuCommand в котором в качестве параметра передается ИД команды
            
Псевдонимы серверов
~~~~~~~~~~~~~~~~~~~~~~~~~            
            
Если на клиенте используется такая конструкция как RemoteClass (по сути обертка вокруг API сервера) то надо завести хотябы один сервер с "is_default": true. Если еще есть сервера то они также используются тут. В таком случае в функциях используются их псевдонимы. В случае одного сервера, псевдоним можно не использовать.
            
**"servers"** – массив псевдонимов серверов
            
Ключи объекта сервера:
            
 * **"alias"** – псевдоним сервера, который используется в фунциях работы с удаленными серверами
 * **"url"** – адрес сервера
 * **"is_default"** – признак сервера по умолчанию
            
Раздел «Датасеты»
~~~~~~~~~~~~~~~~~~~~~
            
**"datasets"** – массив объектов типа «Датасет»

 * **"name"** – имя датасета, по которому к нему обращаться в прикладном решении
 * **"hash_indexes"** – список ключей записей для hash-индексов в виде массива. Пример: [
                "barcode",
                "article"
            ]
* **"text_indexes"** – список полей полнотекстового поиска в виде массива. Пример: [
                "name"
            ]
* **"view_template"** – шаблон отображения записи датасеты в поле ввода формы. Пример: "{name}, {article}"  
** **"api_url"** – прямой url доступа к датасету. Откуда он будет грузится. Обязательно поле. Принцип формирования видно на примере. 
          
Пример конфигурации целиком
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
          
.. code-block:: JSON

 {
    "name": "New configuration",
    "server_name": "",
    "uid": "296f962e-bd0d-4b32-8ac8-3bc9a1162a56",
    "url": "http://nmaker.pw/api/config/296f962e-bd0d-4b32-8ac8-3bc9a1162a56",
    "content_uid": "1dd8f18c-fd58-4c12-b610-0595fe573429",
    "vendor": "Dmitry Vorontsov",
    "nodes_handlers": "base64 encoded python file",
    "nodes_handlers_meta": null,
    "nodes_server_handlers": "base64 encoded python file",
    "nodes_server_handlers_meta": null,
    "version": "00.00.01",
    "NodaLogicFormat": "1.1",
    "NodaLogicType": "ANDROID_SERVER",
    "last_modified": "2025-12-02T13:39:08.917962+03:00",
    "provider": "Dmitry Vorontsov",
    "classes": [
        {
            "name": "MyClass",
            "section": "Documentation samples",
            "section_code": "Documentation",
            "has_storage": true,
            "display_name": "MyClass",
            "cover_image": "[[\"MyClass example\"]]",
            "class_type": "custom_process",
            "hidden": false,
            "methods": [
                {
                    "name": "Open",
                    "source": "internal",
                    "engine": "android_python",
                    "code": "Open"
                },
                {
                    "name": "Input",
                    "source": "internal",
                    "engine": "android_python",
                    "code": "Input"
                }
            ],
            "events": [
                {
                    "event": "onShow",
                    "listener": "",
                    "actions": [
                        {
                            "action": "run",
                            "source": "internal",
                            "server": "",
                            "method": "Open",
                            "postExecuteMethod": ""
                        }
                    ]
                },
                {
                    "event": "onInput",
                    "listener": "",
                    "actions": [
                        {
                            "action": "run",
                            "source": "internal",
                            "server": "",
                            "method": "Input",
                            "postExecuteMethod": ""
                        }
                    ]
                }
            ]
        }
    ],
    "datasets": [
        {
            "name": "goods",
            "hash_indexes": [
                "barcode",
                "article"
            ],
            "text_indexes": [
                "name"
            ],
            "view_template": "{name}, {article}",
            "autoload": false,
            "created_at": "2025-12-01T12:01:01.072524",
            "updated_at": "2025-12-01T12:01:01.072554",
            "api_url": "http://nmaker.pw/api/config/296f962e-bd0d-4b32-8ac8-3bc9a1162a56/dataset/goods/items",
            "item_count": 0
        }
    ],
    "sections": [
        {
            "name": "Documentation samples",
            "code": "Documentation",
            "commands": ""
        }
    ],
    "servers": [
        {
            "alias": "main",
            "url": "https://nmaker.pw",
            "is_default": true
        }
    ]
 }
