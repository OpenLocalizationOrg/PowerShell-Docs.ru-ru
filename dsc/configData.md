#Разделение данных среды и конфигурации

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

В Windows PowerShell требуемой состояние конфигурации (DSC), можно разделить данные конфигурации из логики конфигурации. Рассмотрим другим способом является возможность структурных конфигурации (например, конфигурация может потребоваться установить службы IIS) как отдельные из среды конфигурации (ли ситуации является тестовой среде и производства, один — имена узлов, будет отличаться, но конфигурация применяется к ним будет таким же). Это дает следующие преимущества:

* Он позволяет повторно использовать данные конфигурации для различных ресурсов, узлов и конфигураций.
* Логика конфигурации является более многократного использования, если он не содержит жестко данных. Это похоже на хороший сценариев рекомендации для функций.
* Если некоторые данные требуется изменить, можно внести изменения в одном месте, тем самым сэкономить время и уменьшить ошибки.

##Основные понятия и примеры

Для указания среды частью конфигурации, использует DSC **ConfigurationData** параметра, который является хэш-таблицы (или psd1-файле, который содержит хэш-таблицы может занять). Этот хэш-таблицы должен иметь по крайней мере один ключ **AllNodes**, который имеет структурированного значения. Например:

```powershell
$MyData = 
@{
    AllNodes = @();
    NonNodeData = ""   
}
```

Запишите ключ AllNodes, значение которого представляет собой массив. Каждый элемент этого массива также является хэш-таблицы, в которой требуется имя_узла как ключ:

```powershell
$MyData = 
@{
    AllNodes = 
    @(
        @{
            NodeName = "VM-1"
        },


        @{
            NodeName = "VM-2"
        },


        @{
            NodeName = "VM-3"
        }
    );

    NonNodeData = ""   
}
```

Каждая запись в таблице хэш в AllNodes соответствует данные конфигурации для узла в конфигурации. Помимо необходимый ключ имя_узла можно добавить другие ключи для хэш-таблицы, например:

```powershell
$MyData = 
@{
    AllNodes = 
    @(
        @{
            NodeName = "VM-1";

        },


        @{
            NodeName = "VM-2";
            Role     = "SQLServer"
        },


        @{
            NodeName = "VM-3";
            Role     = "WebServer"
        }
    );

    NonNodeData = ""   
}
```

DSC предлагает три специальные переменные для использования в сценарий настройки:

* **$AllNodes**: ссылается на все узлы. Можно использовать фильтрацию с **. WHERE()** и **. ForEach()**, поэтому вместо жестко запрограммированных имена узлов для выбора конкретного узлов для действия, можно написать примерно так, чтобы выбрать VM-1 и 3 виртуальной Машины в приведенном выше примере:

```powershell
configuration MyConfiguration
{
    node $AllNodes.Where{$_.Role -eq "WebServer"}.NodeName
    {
    }
}
```

* **$Node**: после фильтрации набора узлов, $Node можно использовать для обращения к определенной записи. Например:

```powershell
configuration MyConfiguration
{
    Import-DscResource -ModuleName xWebAdministration -Name MSFT_xWebsite
    node $AllNodes.Where{$_.Role -eq "WebServer"}.NodeName
    {
        xWebsite Site
        {
            Name         = $Node.SiteName
            PhysicalPath = $Node.SiteContents
            Ensure       = "Present"
        }
    }
}
```

Чтобы применить свойство для всех узлов, можно задать имя_узла = "*". Например чтобы предоставить свойство LogPath каждый узел, может этого:

```
$MyData = 
@{
    AllNodes = 
    @(
        @{
            NodeName           = "*"
            LogPath            = "C:\Logs"
        },


        @{
            NodeName = "VM-1";
            Role     = "WebServer"
            SiteContents = "C:\Site1"
            SiteName = "Website1"
        },


        @{
            NodeName = "VM-2";
            Role     = "SQLServer"
        },


        @{
            NodeName = "VM-3";
            Role     = "WebServer";
            SiteContents = "C:\Site2"
            SiteName = "Website3"
        }
    );
}
```

* **$ConfigurationData**: эта переменная в конфигурации можно использовать для доступа к таблице хэш данных конфигурации, переданный в качестве параметра. Например:

```powershell
$MyData = 
@{
    AllNodes = 
    @(
        @{
            NodeName           = "*"
            LogPath            = "C:\Logs"
        },


        @{
            NodeName = "VM-1";
            Role     = "WebServer"
            SiteContents = "C:\Site1"
            SiteName = "Website1"
        },


        @{
            NodeName = "VM-2";
            Role     = "SQLServer"
        },


        @{
            NodeName = "VM-3";
            Role     = "WebServer";
            SiteContents = "C:\Site2"
            SiteName = "Website3"
        }
    );

    NonNodeData = 
    @{
        ConfigFileContents = (Get-Content C:\Template\Config.xml)
     }   
} 

configuration MyConfiguration
{
    Import-DscResource -ModuleName xWebAdministration -Name MSFT_xWebsite

    node $AllNodes.Where{$_.Role -eq "WebServer"}.NodeName
    {
        xWebsite Site
        {
            Name         = $Node.SiteName
            PhysicalPath = $Node.SiteContents
            Ensure       = "Present"
        }

        File ConfigFile
        {
            DestinationPath = $Node.SiteContents + "\\config.xml"
            Contents = $ConfigurationData.NonNodeData.ConfigFileContents
        }
    }
}
```

Можно найти полный пример, включенный в [модуль xWebAdministration](https://powershellgallery.com/packages/xWebAdministration).



