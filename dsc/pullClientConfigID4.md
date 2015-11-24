#Настройка клиента по запросу, используя код конфигурации в PowerShell 4.0

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

Каждый целевой узел должен быть получено сообщение о необходимости использовать режим по запросу и получает URL-адрес, где его можно связаться с сервером по запросу для получения конфигурации. Чтобы сделать это, необходимо настроить локальный диспетчер конфигурации (НОК) с необходимой информации. Чтобы настроить НОК, создайте специальный тип конфигурации называется «metaconfiguration». Дополнительные сведения о настройке НОК см. в разделе [Windows PowerShell 4.0 требуемой состояние конфигурации локальной Configuration Manager](metaConfig4.md)

Следующий скрипт настраивает НОК конфигураций по запросу с сервера с именем «PullServer».

```powershell
Configuration SimpleMetaConfigurationForPull 
{ 
    LocalConfigurationManager 
    { 
        ConfigurationID = "1C707B86-EF8E-4C29-B7C1-34DA2190AE24";
        RefreshMode = "PULL";
        DownloadManagerName = "WebDownloadManager";
        RebootNodeIfNeeded = $true;
        RefreshFrequencyMins = 15;
        ConfigurationModeFrequencyMins = 30; 
        ConfigurationMode = "ApplyAndAutoCorrect";
        DownloadManagerCustomData = @{ServerUrl = "http://PullServer:8080/PSDSCPullServer/PSDSCPullServer.svc"; AllowUnsecureConnection = “TRUE”}
    } 
} 
SimpleMetaConfigurationForPull -Output "."
```

В скрипте **DownloadManagerCustomData** передает URL-адрес сервера по запросу и (в этом примере) позволяет небезопасного соединения.

После выполнения этого скрипта, он создает новую папку с именем выходного **SimpleMetaConfigurationForPull** и помещает metaconfiguration MOF-файл существует.

Для применения к конфигурации, используйте **набора DscLocalConfigurationManager** с параметрами для **имя_компьютера** (использовать "localhost") и **путь** (путь к расположению файла localhost.meta.mof целевой узел). Например:
```powershell
Set-DSCLocalConfigurationManager –ComputerName localhost –Path . –Verbose.
```

##Код конфигурации

Задает скрипт **ConfigurationID** свойство НОК идентификатор GUID, который был ранее создан для этой цели (идентификатор GUID можно создать с помощью **New Guid** командлет).  **ConfigurationID** — НОК используется для поиска соответствующих настроек на сервере по запросу. На сервере по запросу MOF-файл конфигурации должен называться `ConfigurationID.mof`, где *ConfigurationID* значение **ConfigurationID** свойство НОК целевой узел.




