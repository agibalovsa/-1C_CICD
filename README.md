# 1С:CICD

Набор скриптов на 1С:Исполнителе для организации контуров разработки, тестирования, обновления. Для запуска требуется 1С:Исполнитель версии [5.0.8.4](https://releases.1c.ru/version_files?nick=Executor&ver=5.0.8.4).

## Исполнение скриптов

Скрипты исполняются с помощью следующего шаблона:

```cmd
executor <ПутьКСкриптам>main.sbsl <РежимИсполнения> <ОтносительныйПутьККорнюПроекта> <ОстановитьсяВКонце>
```

- **ПутьКСкриптам** - путь к файлу  main.sbsl относительно корня репозитория проекта.

- **РежимИсполнения** - задает конкретную команду для выполнения скриптов.

- **ОтносительныйПутьККорнюПроекта** - указывается относительный путь от рабочего каталога, в котором запускается скрипт (текущий каталог оболочки исполнения), до корня репозитория проекта. Например, если скрипт запускается из каталога bin, то параметр будет иметь следующий вид `..//`.

> Внимание. Не следует путать рабочий каталог запуска скрипта с местом расположения файлов скрипта.

- **ОстановитьсяВКонце** - **1** - перед завершением исполнения скрипта, будет ожидаться ввод от пользователя (необходим для анализа вывода лога исполнения), 0 - скрипт завершится, не дожидаясь пользователя.

## Режимы исполнения скриптов

- **init** - инициализация репозитория проекта для работы со скриптами.
- **build_cf** - сборка файла конфигурации из файловой выгрузки xml.
- **build_cfe** - сборка файлов расширений из файловой выгрузки xml.
- **build_patch** - сборка архива для обновления из файловой выгрузки xml только по полностью или частично снятых с поддержки объектов.
- **build_prod** - сборка файла конфигурации из файловой выгрузки xml с дополнительным конвертацией конфигурацией поставщика с целью устранения шума ложных различий между основной конфигурацией и конфигурацией поставщика.
- **update_ib_cf** - обновление информационной базы собранным файлом cf.
- **update_ib_cfe** - обновление информационной базы собранными файлами cfe.
- **update_ib_patch** - частичное обновление информационной базы файлами из архива с файлами полностью или частично снятых с поддержки объектов.
- **get_configdump** - получение файла ConfigDumpInfo.xml из информационной базы, где идет разработка, для ускоренной выгрузки в файлы xml измененной конфигурации.
- **new_bin** - конвертация файла ParentConfigurations.bin в формат, пригодный для хранения и работы с ним в git хранилище.
- **reapair_bin** - удаление из файла ParentConfigurations.bin битых идентификаторов метаданных.
- **new_configdumpinfo** - конвертация файла ConfigDumpInfo.xml в формат, пригодный для хранения и работы с ним в git хранилище.

## Требуемая структура репозитория проекта

Для корректной работы скриптов проект должен иметь следующую структуру каталогов:

```ini
project
  |- bin
    |- project_user_prop.json
    |- Конфигурация.cf
    |- Расширения.cfe
  |- scripts
    |- 1c_cicd
  |- src
    |- cf
    |- cfe
      |- ИмяРасширения1
      |- ИмяРасширения2
  |- project_config.json
  |- .gitignore
```

- **scripts** - каталог, где хранятся скрипты.

- **src/cf** - каталог с файловой выгрузкой основной конфигурации.

- **src/cfe/*** - каталоги с файловой выгрузкой  расширений.

- **bin** - каталог с собранными файлами конфигураций cf, cfe, patch.

- **project_config.json** - файл с общими настройками проекта.

  ```ini
  {
    "EXT_NAMES" : [
      "ИмяРасширения1",
      "ИмяРасширения2",
    ],
    "BIN_DIR" : "bin",
    "SRC_CFE" : "src\\cfe",
    "SRC_CF" : "src\\cf",
    "PLATFORM_VERSION" : "8.3.22"
  }
  ```

  - EXT_NAMES - перечисление расширений, которые необходимо собирать в файлы cfe при запуске скрипта в режиме build_cfe.
  - BIN_DIR - путь к каталогу bin относительно корня репозитория проекта.
  - SRC_CF- путь к каталогу с файловой выгрузкой xml основной конфигурации проекта.
  - SRC_CFE - путь к каталогу с файловыми выгрузками xml расширений проекта.
  - PLATFORM_VERSION - текущая версия платформы, на которой ведется разработка.

- **project_user_prop.json** - файл с пользовательскими настройками проекта.

  ```ini
  {
    "USE_IBCMD" : false,
    "IB_LOGIN" : "Администратор",
    "IB_PASSWD" : "passw",
    "IB_CONNECTION_STRING" : "Srvr=\"gor\";Ref=\"Arconic_DO_master_agibse\";",
    "HASH" : {
      "cf" : "d1a01a96c07639833d215549eb2ac4bd",
      "ар_БиблиотекаПодсистем" : "e5f121aaf21e584461511fc5f5a9a0bd",
      "ар_ДокументыИФайлы" : "2b2f49331a5c2f2bc03a7f80d30ba06d",
      "ар_ОбменДанными" : "4a70f956be22cab41748e76d7f492d69",
      "ар_Роли" : "de5ba153e8d8ef419c072f3925add5bd",
      "ар_НСИ" : "da6e93d7367780db067c5a3a4e72a352"
    }
  }
  ```

  - HASH - хэши каталогов файловых выгрузок, нужны для того, чтобы сборка файлов проекта выполнялась только в случае изменении в файловой выгрузки относительно предыдущей сборки
  - USE_IBCMD - режим использования автономного сервера для сборки файлов конфигурации
  - IB_CONNECTION_STRING - строка соединения до конфигурации, в которой ведется разработка. (Соединение с конфигурацией выполняется в режимах get_configdump, update*)
  - IB_LOGIN - логин конфигурации, в которой ведется разработка
  - IB_PASSWD - пароль конфигурации, в которой ведется разработка

- **.gitignore** - файл с описанием каталогов, которые не должны версионироваться в git. В данном случае в нем должен содержаться каталог bin.

  ```ini
  /bin
  ```

## Инициализация репозитория проекта

Для инициализации репозитории проекта для работы со скриптами необходимо выполнить несколько шагов:

1. Выбрать место расположение скриптов, например, в каталоге scripts/1c_cicd. Скрипты можно как просто скопировать с родительского репозитория, так и расположить в качестве подмодуля репозитория git.

2. Создать файл инициализации и расположить его в каталоге bin. Например, файл init.bat со следующим кодом

   ```cmd
   executor ..\scripts\1c_cicd\main.sbsl init "..\\" 1
   ```

3. Необходимо запустить команду инициализации, после чего создадутся следующие файлы

   1. Файл настроек проекта ./project_config.json

   2. Файл пользовательских настроек bin/project_user_prop.json

   3. Пакетные файлы для запуски скриптов в разных режимах bin/*

   4. На основе описанной структуры проекта создается файл .git/hooks/precommit с режимами обслуживания репозитория перед фиксацией изменений

      ```bash
      #!/bin/sh
      executor scripts/1c_cicd/main.sbsl new_bin "." 0
      executor scripts/1c_cicd/main.sbsl new_configdumpinfo "." 0
      git add src/cf/Ext/ParentConfigurations.bin src/cf/ConfigDumpInfo.xml src/cfe/*/ConfigDumpInfo.xml
      executor scripts/1c_cicd/main.sbsl build_cfe "." 0
      ```

      Без данных строчек, возникнут проблемы при слиянии изменений, сборки после слияния. Режим build_cfe не только собирает cfe но и проверяет, что перед фиксацией все изменения создаю корректное расширение.

## Особенности использования файла  ConfigDumpInfo.xml

В файловой выгрузке конфигурации содержится очень важный файл ConfigDumpInfo.xml.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ConfigDumpInfo xmlns="http://v8.1c.ru/8.3/xcf/dumpinfo" xmlns:xen="http://v8.1c.ru/8.3/xcf/enums" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" format="Hierarchical" version="2.15">
  <ConfigVersions>
    <Metadata name="AccumulationRegister.КоличествоДействийЗадач" id="fb076a86-f98a-4262-aae6-9620b66ffe56" configVersion="7968057765e97c4b85bbe9cdba2d6add00000000">
      <Metadata name="AccumulationRegister.КоличествоДействийЗадач.Resource.ОжидающихПроверки" id="12ee901e-40ef-403d-97ac-6d417f7f89bc"/>
      <Metadata name="AccumulationRegister.КоличествоДействийЗадач.Resource.Просроченных" id="19149630-16cb-4810-bb20-274f0f690254"/>
****
    </Metadata>
    <Metadata name="AccumulationRegister.КоличествоДействийЗадач.ManagerModule" id="fb076a86-f98a-4262-aae6-9620b66ffe56.2" configVersion="35499d4fcdad694c989da066e77b447600000000"/>
    <Metadata name="AccumulationRegister.КоличествоДействийЗадач.RecordSetModule" id="fb076a86-f98a-4262-aae6-9620b66ffe56.1" configVersion="4ad30263ad581f4daaae5dd125fa9c2c00000000"/>
***
```

- В нем содержится иерархия объектов метаданных. Данной информации нет в файле Configuration.xml.

- Иерархия метаданных помогает быстро обратиться к нужному файлу в файловой выгрузке по идентификатору объекта метаданных.

- Также в нем содержится атрибут configVersion, которые помогает конфигурации выгрузить только измененные объекты относительно предыдущей выгрузки.

- Также, после удалении объектов в конфигурации, при файловой выгрузке платформа по файлу ConfigDumpInfo.xml понимает, какие файлы нужно удалить в каталоге.

> Внимание. Без файла ConfigDumpInfo.xml файлы удаленных метаданных <u>**не удаляются**</u>.

Для того, чтобы файл ConfigDumpInfo.xml в репозитории проекта не конфликтовал в ветками других разработчиков из-за  configVersion, данный атрибут обнуляется с помощью режима скриптов `new_configdumpinfo`.

```ini
configVersion="0000000000000000000000000000000000000000
```

> Внимание. Необходимо убедиться, что в git precommit прописан режим `new_configdumpinfo` ([см.  Инициализация репозитория проекта](#инициализация-репозитория-проекта))

### Ускоренная выгрузка конфигурации в файлы

Для ускоренной выгрузки конфигурации в файлы необходимо <u>**перед**</u> началом разработки актуализировать ConfigDumpInfo.xml. Канонический процесс разработки с использованием режима актуализации ConfigDumpInfo.xml  выглядит следующим образом:

1. Перейти на ветку мастер.
2. Создать новую ветку разработки по задаче разработки.
3. Актуализировать файл ConfigDumpInfo.xml с помощью режима get_configdump.
4. Зайти в конфигуратор, провести разработку.
5. Выгрузить конфигуратор в файлы.
6. Зафиксировать изменения, ConfigDumpInfo.xml вернется к безверсионному формату.

> Внимание. Ускоренная выгрузка <u>не будет</u> работать, в случае удаления или переименования объектов метаданных.
