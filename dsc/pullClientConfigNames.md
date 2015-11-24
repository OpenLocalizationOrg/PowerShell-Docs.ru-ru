#Настройка с использованием имен конфигурации клиента по запросу

> Область применения: Windows PowerShell 5.0

Каждый целевой узел должен быть получено сообщение о необходимости использовать режим по запросу и получает URL-адрес, где его можно связаться с сервером по запросу для получения конфигурации. Чтобы сделать это, необходимо настроить локальный диспетчер конфигурации (НОК) с необходимой информации. Чтобы настроить НОК, создайте специальный тип конфигурации, с **DSCLocalConfigurationManager** атрибута. Дополнительные сведения о настройке НОК см. в разделе [Настройка локальной Configuration Manager](metaConfig.md).

> **Примечание**: этот раздел относится к PowerShell 5.0. Сведения о настройке клиентов по запросу в PowerShell 4.0 см. в разделе [Настройка клиента по запросу, используя код конфигурации в PowerShell 4.0](pullClientConfigID4.md)

Следующий скрипт настраивает НОК конфигураций по запросу с сервера с именем «CONTOSO-PullSrv»:

```powershell
[DSCLocalConfigurationManager()]
configuration PullClientConfigID
{
    Node localhost
    {
        Settings
        {
            RefreshMode = 'Pull'
            RefreshFrequencyMins = 30 
            RebootNodeIfNeeded = $true
        }
        ConfigurationRepositoryWeb CONTOSO-PullSrv
        {
            ServerURL = 'https://CONTOSO-PullSrv:8080/PSDSCPullServer.svc'
            RegistrationKey = '140a952b-b9d6-406b-b416-e0f759c9c0e4'
            ConfigurationNames = @('ClientConfig')

        }      
    }
}
PullClientConfigID
```

В скрипте **ConfigurationRepositoryWeb** блока определяет сервер по запросу.  **ServerURL** определяет конечную точку для сервера по запросу.

**RegistrationKey** свойство является общим ключом между все узлы клиента для сервера по запросу и по запросу сервера. То же значение хранится в файле на сервере по запросу.  **ConfigurationNames** свойство задает имя конфигурации, предназначенной для узла клиента. На сервере по запросу файл конфигурации MOF для этого узла клиента должен называться *ConfigurationNames*.mof, где *ConfigurationNames* совпадает со значением **ConfigurationNames** свойства, заданные в этом metaconfiguration.

После выполнения этого скрипта, он создает новый выходную папку с именем **PullClientConfigID** и помещает metaconfiguration MOF-файл существует. В этом случае с именем файла MOF metaconfiguration `localhost.meta.mof`.

Чтобы применить конфигурацию, вызовите **DscLocalConfigurationManager набор** командлета, с **путь** расположение metaconfiguration MOF-файл. Например: `DSCLocalConfigurationManager набор localhost — путь. \PullClientConfigID – Verbose.`

> **Примечание**: ключи регистрации работают только с веб-серверов по запросу. Необходимо использовать **ConfigurationID** с сервером SMB по запросу. Дополнительные сведения о настройке сервера по запросу с помощью **ConfigurationID**, в разделе [Настройка клиента по запросу, используя код конфигурации](pullClientConfigID.md)

##Ресурс и серверах отчетов

По умолчанию узел клиента получает необходимые ресурсы и отчеты о состоянии сервера конфигурации по запросу. Однако можно указать различные по запросу серверов для ресурсов и создания отчетов.
Чтобы указать сервер ресурсов, либо используйте **ResourceRepositoryWeb** (для веб-сервера по запросу) или **ResourceRepositoryShare** блоком (сервер SMB по запросу).
Чтобы указать сервер отчетов, используйте **ReportRepositoryWeb** блока. Сервер отчетов не может быть сервером SMB.
Настраивает следующие metaconfiguration по запросу клиента для получения его конфигурации из **CONTOSO PullSrv** и его ресурсы из **CONTOSO ResourceSrv**, и отправлять отчеты о состоянии для **CONTOSO ReportSrv**:

```powershell
[DSCLocalConfigurationManager()]
configuration PullClientConfigID
{
    Node localhost
    {
        Settings
        {
            RefreshMode = 'Pull'
            RefreshFrequencyMins = 30 
            RebootNodeIfNeeded = $true
        }

        ConfigurationRepositoryWeb CONTOSO-PullSrv
        {
            ServerURL = 'https://CONTOSO-PullSrv:8080/PSDSCPullServer.svc'
            RegistrationKey = 'fbc6ef09-ad98-4aad-a062-92b0e0327562'
        }

        ResourceRepositoryWeb CONTOSO-ResourceSrv
        {
            ServerURL = 'https://CONTOSO-REsourceSrv:8080/PSDSCPullServer.svc'
            RegistrationKey = '30ef9bd8-9acf-4e01-8374-4dc35710fc90'
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

* [Настройка клиента по запросу с ИД конфигурации](pullClientConfigID.md)
* [Настройка веб-сервера по запросу DSC](pullServer.md)




