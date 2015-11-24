#Настройка клиента по запросу, используя код конфигурации

> Область применения: Windows PowerShell 5.0

Каждый целевой узел должен быть получено сообщение о необходимости использовать режим по запросу и получает URL-адрес, где его можно связаться с сервером по запросу для получения конфигурации. Чтобы сделать это, необходимо настроить локальный диспетчер конфигурации (НОК) с необходимой информации. Чтобы настроить НОК, создайте специальный тип конфигурации, снижен с **DSCLocalConfigurationManager** атрибута. Дополнительные сведения о настройке НОК см. в разделе [Настройка локальной Configuration Manager](metaConfig.md).

> **Примечание**: этот раздел относится к PowerShell 5.0. Сведения о настройке клиентов по запросу в PowerShell 4.0 см. в разделе [Настройка клиента по запросу, используя код конфигурации в PowerShell 4.0](pullClientConfigID4.md)

Следующий скрипт настраивает НОК конфигураций по запросу с сервера с именем «CONTOSO-PullSrv».

```powershell
[DSCLocalConfigurationManager()]
configuration PullClientConfigID
{
    Node localhost
    {
        Settings
        {
            RefreshMode = 'Pull'
            ConfigurationID = 1d545e3b-60c3-47a0-bf65-5afc05182fd0'
            RefreshFrequencyMins = 30 
            RebootNodeIfNeeded = $true
        }
        ConfigurationRepositoryWeb CONTOSO-PullSrv
        {
            ServerURL = 'https://CONTOSO-PullSrv:8080/PSDSCPullServer.svc'

        }      
    }
}
PullClientConfigID
```

В скрипте **ConfigurationRepositoryWeb** блока определяет сервер по запросу.  **ServerURL**

После выполнения этого скрипта, он создает новый выходную папку с именем **PullClientConfigID** и помещает metaconfiguration MOF-файл существует. В этом случае с именем файла MOF metaconfiguration `localhost.meta.mof`.

Чтобы применить конфигурацию, вызовите **DscLocalConfigurationManager набор** командлета, с **путь** расположение metaconfiguration MOF-файл. Например: `DSCLocalConfigurationManager набор localhost — путь. \PullClientConfigID – Verbose.`

##Код конфигурации

Задает скрипт **ConfigurationID** свойство НОК идентификатор GUID, который был ранее создан для этой цели (идентификатор GUID можно создать с помощью **New Guid** командлет).  **ConfigurationID** — НОК используется для поиска соответствующих настроек на сервере по запросу. На сервере по запросу MOF-файл конфигурации должен называться _ConfigurationID_.mof, где _ConfigurationID_ значение **ConfigurationID** свойство НОК целевой узел.

##По запросу сервер SMB

Для настройки клиента с конфигурациями по запросу с сервера SMB, используйте **ConfigurationRepositoryShare** блока. В **ConfigurationRepositoryShare** блока, укажите путь к серверу, задав **SourcePath** свойства. Целевой узел для извлечения с сервера по запросу SMB с именем настраивает следующие metaconfiguration **SMBPullServer**.

```powershell
[DSCLocalConfigurationManager()]
configuration PullClientConfigID
{
    Node localhost
    {
        Settings
        {
            RefreshMode = 'Pull'
            ConfigurationID = '1d545e3b-60c3-47a0-bf65-5afc05182fd0'
            RefreshFrequencyMins = 30 
            RebootNodeIfNeeded = $true
        }
        ConfigurationRepositoryWeb SMBPullServer
        {
            SourcePath = '\\SMBPullServer\PullSource'

        }     
    }
}
PullClientConfigID
```

##Ресурс и серверах отчетов

По умолчанию узел клиента получает необходимые ресурсы и отчеты о состоянии сервера конфигурации по запросу. Однако можно указать различные по запросу серверов для ресурсов и создания отчетов.
Чтобы указать сервер ресурсов, либо используйте **ResourceRepositoryWeb** (для веб-сервера по запросу) или **ResourceRepositoryShare** блоком (сервер SMB по запросу).
Чтобы указать сервер отчетов, используйте **ReportRepositoryWeb** блока. Сервер отчетов не может быть сервером SMB.
Настраивает следующие metaconfiguration по запросу клиента для получения его конфигурации из **CONTOSO PullSrv** и его ресурсы из **CONTOSO ResourceSrv**, и отправлять отчеты о состоянии для **CONTOSO ReportSrv**.

```powershell
[DSCLocalConfigurationManager()]
configuration PullClientConfigID
{
    Node localhost
    {
        Settings
        {
            RefreshMode = 'Pull'
            ConfigurationID = '1d545e3b-60c3-47a0-bf65-5afc05182fd0'
            RefreshFrequencyMins = 30 
            RebootNodeIfNeeded = $true
        }

        ConfigurationRepositoryWeb CONTOSO-PullSrv
        {
            ServerURL = 'https://CONTOSO-PullSrv:8080/PSDSCPullServer.svc'

        }

        ResourceRepositoryWeb CONTOSO-ResourceSrv
        {
            ServerURL = 'https://CONTOSO-REsourceSrv:8080/PSDSCPullServer.svc'
        }

        ReportServerWeb CONTOSO-ReportSrv
        {
            ServerURL = 'https://CONTOSO-REsourceSrv:8080/PSDSCPullServer.svc'
        }
    }
}
PullClientConfigID
```

##См. также

* [Настройка клиента по запросу с именами конфигурации](pullClientConfigNames.md)



