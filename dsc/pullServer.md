#Настройка веб-сервера по запросу DSC

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

Веб-сервер по запросу DSC является веб-службы в IIS, которое использует интерфейс OData для предоставления файлов конфигурации DSC целевые узлы при их запрашивать эти узлы.

Требования для использования сервера по запросу:

* Серверы:
   - WMF-PowerShell 4.0 или выше
   - Роль сервера IIS
   - Служба DSC
* В идеальном случае некоторые означает создание сертификата, для защиты учетных данных, переданных на целевые узлы для локального диспетчера конфигурации (НОК)

Роль сервера IIS и служба DSC можно добавить с помощью мастера добавления ролей и компонентов в диспетчере сервера или с помощью PowerShell. В этом разделе содержатся образцы скриптов будет обрабатывать оба эти действия можно также.

##Использование ресурсов xWebService

Для настройки веб-сервера по запросу проще всего использовать xWebService ресурсов, включенных в модуле xPSDesiredStateConfiguration. Ниже объясняется, как использовать ресурс в конфигурации, которая настраивает веб-службы.

1. Вызовите [установки модуля](https://technet.microsoft.com/en-us/library/dn807162.aspx) командлет, чтобы установить **xPSDesiredStateConfiguration** модуля. **Примечание**: **установки модуля** включается в **PowerShellGet** модуль, который включается в PowerShell 5.0. Можно загрузить **PowerShellGet** модуля PowerShell 3.0 и 4.0 на [предварительного просмотра модули PowerShell PackageManagement](https://www.microsoft.com/en-us/download/details.aspx?id=49186).
1. Создать самозаверяющий сертификат с субъектом `"CN = PSDSCPullServerCert"` в `CERT: \LocalMachine\MY\` хранения. Это можно сделать с помощью команды `SelfSignedCertificate Нью - CertStoreLocation «CERT: \LocalMachine\MY»-субъекта "CN = PSDSCPullServerCert"`.
1. В PowerShell ISE запустить (F5) следующий скрипт конфигурации (включен в распаковать файлы как Sample_xDscWebService.ps1). Этот скрипт настраивает сервер по запросу и сервер соответствия.

```powershell
Configuration Sample_xDSCService
{
    param (
    [ValidateNotNullOrEmpty()]
    [String] $certificateThumbPrint
    )
    Import-DSCResource –ModuleName DSCServic
    Node localhost
    {
        WindowsFeature DSCServiceFeature
        {
            Ensure = “Present”
            Name = “DSC-Service”
        }

        DSCService PSDSCPullServer
        {
            Ensure   = "Present"
            Name     = “PSDSCPullServer”
            Port     = 8080
            PhysicalPath = "$env:SystemDrive\inetpub\wwwroot\PSDSCPullServer"
            EnableFirewallException = $true
            CertificateThumbprint = $certificateThumbPrint
            ModulePath = “$env:PROGRAMFILES\WindowsPowerShell\DscService\Modules”
            ConfigurationPath = “$env:PROGRAMFILES\WindowsPowerShell\DscService\Configuration”
            State = “Started”
            DependsOn = “[WindowsFeature]DSCServiceFeature             
        }
    }
}
```

1. Конфигурации запуска, передав отпечаток самостоятельно подписанный сертификат был создан как **certificateThumbPrint** параметр:

```powershell
PS:\>$myCert = Get-ChildItem CERT: | Where {$_.Subject -eq 'CN=PSDSCPullServerCert'}
PS:\>Sample_xDSCService -certificateThumbprint $myCert.Thumbprint 
```

##Ключ регистрации

Чтобы разрешить клиентским узлы, чтобы зарегистрировать на сервере, чтобы они могли использовать имена конфигурации вместо кода конфигурации, ключ регистрации (являющийся идентификатором GUID известно, что узел клиента и сервера) необходимо поместить в файл с именем `RegistrationKeys.txt`. По умолчанию сервер по запросу, созданный в данном примере ожидает расположены в этот файл `C:\Program Files\WindowsPowerShell\DscService`. Создайте текстовый файл с только одну строку, состоящую из регистрационный ключ и сохранить его в эту папку.
> **Примечание**: регистрация ключи не поддерживаются в PowerShell 4.0.

##Размещение конфигураций и ресурсы

После завершения установки сервера по запросу, является новой папки в `$env: PROGRAMFILES\WindowsPowerShell` с именем «DscService». В этой папке существует две папки с именем «Модули» и «Конфигурация». В папке «Модули» поместите все ресурсы, необходимые для конфигурации, которые будет извлекать узлы с этого сервера. В папке «Конфигурация» установите MOF-файлов конфигурации для всех конфигураций, которые будут извлечены узлами. Имена файлов MOF зависят от типа клиента по запросу. В следующих разделах описывается настройка клиентов по запросу подробно:

* [Настройка клиента DSC по запросу, используя код конфигурации](pullClientConfigID.md)
* [Настройка клиента DSC по запросу, используя имена конфигурации](pullClientConfigNames.md)
* [Частичное конфигураций](partialConfigs.md)

##См. также:

* [Общие сведения о состоянии конфигурации требуемого Windows PowerShell](overview.md)
* [Введение в действие конфигурации](enactingConfigurations.md)
* [Как получить сведения об узле из DSC по запросу сервера](retrieveNodeInfo.md)



