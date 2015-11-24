#DSC для Linux nxFileLine ресурсов

**NxFileLine** ресурс в PowerShell требуемой состояние конфигурации (DSC) предоставляет механизм для управления строки в файле конфигурации на узле Linux.

##Синтаксис

```
nxFileLine <string> #ResourceName
{
    FilePath = <string>
    ContainsLine = <string>
    [ DoesNotContainPattern = <string> ]
    [ DependsOn = <string[]> ]

}
```

##Свойства

| Свойство| Описание|
|---|---|
| FilePath| Полный путь к файлу для управления строк на целевом узле.|
| ContainsLine| Строки, чтобы обеспечить существует в файле.Если не существует в файле этой строки будут добавлены в файл.**ContainsLine** является обязательным, но можно задать пустую строку ("ContainsLine =" "), если он не является обязательным.|
| DoesNotContainPattern| Шаблон регулярного выражения для строк, которые не должны присутствовать в файле.Для любой строки, которые содержатся в файле, соответствовать этому регулярному выражению строки будут удалены из файла.|
| DependsOn| Указывает, что конфигурация другого ресурса должна выполняться перед настройкой этого ресурса.Например если **идентификатор** ресурса блок скрипта конфигурации, необходимо запустить сначала — **ResourceName** и его тип является **типа ресурса**, синтаксис для использования этого свойства `DependsOn = "[типа ресурса] ResourceName"`.|

##Пример

В этом примере демонстрируется использование **nxFileLine** ресурсов для настройки `/etc/sudoers` файл, убедиться, что пользователь: monuser requiretty не настроено.

```
Import-DSCResource -Module nx 

nxFileLine DoNotRequireTTY
{
   FilePath = “/etc/sudoers”
   ContainsLine = 'Defaults:monuser !requiretty'
   DoesNotContainPattern = "Defaults:monuser[ ]+requiretty"
} 
```





