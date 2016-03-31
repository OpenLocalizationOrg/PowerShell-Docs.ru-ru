#DSC для Linux nxArchive ресурсов

**NxArchive** ресурсов в PowerShell требуемой состояние конфигурации (DSC) предоставляет механизм для распаковки файлов архива (.tar, .zip) на конкретный путь на узле Linux.

##Синтаксис

```
nxArchive <string> #ResourceName
{
    SourcePath = <string>
    DestinationPath = <string>
    [ Checksum = <string> { ctime | mtime | md5 }  ]
    [ Force = <bool> ]
    [ DependsOn = <string[]> ]
    [ Ensure = <string> { Absent | Present }  ]
}
```

##Свойства

| Свойство| Описание|
|---|---|
| SourcePath| Задает исходный путь для файла архива.Это должно быть .tar .zip, или. tar.gz файл.|
| DestinationPath| Указывает расположение, где необходимо убедиться, что содержимое архива будет извлечено.|
| Контрольная сумма| Определяет тип, используемый при определении, обновляется ли архив источника.Значения: «ctime», «mtime» или «md5».Значение по умолчанию — «md5».|
| Force| Определенных операций с файлами (перезаписи файла или при удалении каталога, который не является пустым) приведет к ошибке.С помощью **принудительно** такие ошибки переопределения свойств.Значение по умолчанию — **$false**.|
| DependsOn| Указывает, что конфигурация другого ресурса должна выполняться перед настройкой этого ресурса.Например если **идентификатор** ресурса блок скрипта конфигурации, необходимо запустить сначала — **ResourceName** и его тип является **типа ресурса**, синтаксис для использования этого свойства `DependsOn = "[типа ресурса] ResourceName"`.|
| Убедитесь| Определяет, следует проверить, существует ли содержимое архива в **назначения**.Установите это свойство для «Отсутствует», чтобы убедиться, что существует содержимое.Задайте для него значение «Отсутствует», чтобы убедиться, что они не существуют.Значение по умолчанию — «Отсутствует».|

##Пример

В следующем примере показано, как использовать **nxArchive** ресурсов, чтобы убедиться, что содержимое архивный файл с именем `website.tar` существует и извлекаются в заданный конечный.

```
Import-DSCResource -Module nx 

nxFile SyncArchiveFromWeb
{
   Ensure = "Present"
   SourcePath = “http://release.contoso.com/releases/website.tar”
   DestinationPath = "/usr/release/staging/website.tar"
   Type = "File"
   Checksum = “mtime”
}

nxArchive SyncWebDir
{
   SourcePath = “/usr/release/staging/website.tar”
   DestinationPath = “/usr/local/apache2/htdocs/”
   Force = $false
   DependsOn = "[nxFile]SyncArchiveFromWeb"
} 
```



