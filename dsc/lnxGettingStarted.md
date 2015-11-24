#Приступая к работе с требуемой конфигурации состояния (DSC) для Linux

В этом разделе объясняется, как приступить к работе с помощью PowerShell требуемой состояние конфигурации (DSC) для Linux. Общие сведения о DSC см. в разделе [Приступая к работе с требуемой конфигурации состояния Windows PowerShell](overview.md).

##Поддерживаемые версии системы Linux операции

Следующие версии операционной системы Linux поддерживаются для DSC для Linux.
- CentOS 5, 6 и 7 (x 86 и x 64)
- Debian GNU/Linux 6 и 7 (x 86 и x 64)
- Oracle Linux 5, 6 и 7 (x 86 и x 64)
- Red Hat Enterprise Linux Server 5, 6 и 7 (x 86 и x 64)
- SUSE Linux Enterprise Server 10, 11 и 12 (x 86 и x 64)
- Ubuntu Server 12.04 Результатов и 14.04 Результатов (x 86 и x 64)

В следующей таблице описаны требуется путь для зависимостей для DSC для Linux.

| Требуемый пакет| Описание| Минимальная версия|
|---|---|---|
| glibc| Библиотеки GNU| 2…4 – 31.30|
| Python| Python| 2.4 – 3.4|
| omiserver| Откройте Управление инфраструктурой| 1.0.8.1|
| OpenSSL| Библиотеки OpenSSL| 0.9.8 или 1.0|
| ctypes| Библиотека Python CTypes| Должна совпадать с версией Python|
| libcurl| Клиентская библиотека перелистывания http| 7.15.1|

##Установка DSC для Linux

Необходимо установить [Откройте инфраструктуры управления (OMI)](https://collaboration.opengroup.org/omi/) перед установкой DSC для Linux.

###Установка OMI

Требуемой конфигурации состояния для Linux требуется сервер CIM откройте инфраструктуры управления (OMI), версия 1.0.8.1. OMI можно загрузить из Open Group: [Откройте инфраструктуры управления (OMI)](https://collaboration.opengroup.org/omi/).

Чтобы установить OMI, установите пакет, который подходит для вашей системы Linux (.rpm или .deb) и версию OpenSSL (ssl_098 или ssl_100) и архитектуры (x 64 и x 86). Пакетов RPM подходят для CentOS, Red Hat Enterprise Linux, SUSE Linux Enterprise Server и Oracle Linux. Пакеты DEB подходят для Debian GNU/Linux и Ubuntu Server. Ssl_098 пакеты, которые подходят для компьютеров с помощью OpenSSL установлена ssl 0.9.8_100 пакетов подходят для компьютеров с OpenSSL 1.0 установлена.

> **Примечание**: чтобы определить установленную версию OpenSSL, выполните команду `openssl версии`.

Выполните следующую команду для установки OMI системе CentOS 7 x 64.

`# sudo rpm - Uvh omiserver-1.0.8.ssl_100.rpm`

###Установка DSC

Чтобы установить DSC, установите пакет, который подходит для вашей системы Linux (.rpm или .deb) и версию OpenSSL (ssl_098 или ssl_100) и архитектуры (x 64 и x 86). Пакетов RPM подходят для CentOS, Red Hat Enterprise Linux, SUSE Linux Enterprise Server и Oracle Linux. Пакеты DEB подходят для Debian GNU/Linux и Ubuntu Server. Ssl_098 пакеты, которые подходят для компьютеров с помощью OpenSSL установлена ssl 0.9.8_100 пакетов подходят для компьютеров с OpenSSL 1.0 установлена.

> **Примечание**: чтобы определить установленную версию OpenSSL, запустите версию команда openssl.

Выполните следующую команду для установки DSC системе CentOS 7 x 64.

`rpm - Uvh # sudo dsc-1.0.0-254.ssl_100.x64.rpm`


##С помощью DSC для Linux

Ниже описаны процедуры для создания и запуска DSC конфигурации на компьютерах Linux.

###Создание документа MOF конфигурации

Ключевое слово Конфигурация Windows PowerShell используется для создания конфигурации для компьютеров Linux так же, как для компьютеров Windows. Следующие шаги описывают создание документа конфигурации для компьютера Linux с помощью Windows PowerShell.

1. Импортируйте модуль nx. Модуль Windows PowerShell nx содержит схему для встроенных ресурсов для DSC для Linux и должен быть установлен на локальном компьютере и импортирован в конфигурации.
   
   -Чтобы установить модуль nx, скопируйте каталог модуля nx либо `%UserProfile%\Documents\WindowsPowerShell\Modules\` или `C:\windows\system32\WindowsPowerShell\v1.0\Modules`. Модуль nx включается в DSC для Linux установочного пакета (MSI). Чтобы импортировать модуль nx в конфигурации, используйте __DSCResource импорта__ команды:

```powershell
Configuration ExampleConfiguration{

    Import-DSCResource -Module nx

}
```
2. Определить конфигурацию и создать документ конфигурации:

```powershell
Configuration ExampleConfiguration{

    Import-DSCResource -Module nx

    Node  "linuxhost.contoso.com"{
    nxFile ExampleFile {

        DestinationPath = "/tmp/example"
        Contents = "hello world `n"
        Ensure = "Present"
        Type = "File"
    }

    }
}
ExampleConfiguration -OutputPath:"C:\temp" 
```

###Принудительно отправить конфигурацию компьютера Linux

Конфигурация документы (файлы MOF) могут отправляться в компьютере Linux с помощью [начала DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) командлета. Для использования этого командлета вместе с [Get DscConfiguration](https://technet.microsoft.com/en-us/library/dn407379).aspx, или [Тест DscConfiguration](https://technet.microsoft.com/en-us/library/dn407382.aspx) командлетов, удаленно на компьютере Linux, необходимо использовать CIMSession.  [New-CimSession](https://technet.microsoft.com/en-us/library/jj590760.aspx) командлет используется для создания CIMSession компьютеру Linux.

Следующий код показывает, как создание CIMSession для DSC для Linux.

```powershell
$Node = "ostc-dsc-01"
$Credential = Get-Credential -UserName:"root" -Message:"Enter Password:"

#Ignore SSL certificate validation
#$opt = New-CimSessionOption -UseSsl:$true -SkipCACheck:$true -SkipCNCheck:$true -SkipRevocationCheck:$true

#Options for a trusted SSL certificate
$opt = New-CimSessionOption -UseSsl:$true 
$Sess=New-CimSession -Credential:$credential -ComputerName:$Node -Port:5986 -Authentication:basic -SessionOption:$opt -OperationTimeoutSec:90 
```

> **Note**.
* В режиме «Push» учетные данные пользователя должен быть привилегированного пользователя на компьютере Linux.
* Для Linux, New-CimSession должен использоваться с параметром – UseSSL значение $true для DSC поддерживаются только подключения SSL/TLS.
* Сертификат SSL, используемый OMI (для DSC) указан в файле: `/opt/omi/etc/omiserver.conf` со свойствами: pemfile и файл ключа.
   Если этот сертификат не является доверенным для компьютера Windows, на которых запущена [New-CimSession](https://technet.microsoft.com/en-us/library/jj590760.aspx) командлета на вы можете пропустить проверку сертификата с параметрами CIMSession: `- SkipCACheck: $true - SkipCNCheck: $true - SkipRevocationCheck: $true`

Выполните следующую команду для передачи конфигурации DSC на узел Linux.

`Начало DSCConfiguration-путь: «C:\temp» - cimsession: $sess-wait - verbose`

###Распространите конфигурацию с сервера по запросу

Конфигурации можно распространить на компьютер Linux с сервером по запросу, так же, как для компьютеров Windows. Рекомендации по использованию сервера по запросу в разделе [Windows PowerShell требуемой конфигурации по запросу серверам состояния](pullServer.md). Дополнительные сведения и ограничения, связанные с использованием компьютеров Linux с сервером по запросу см. в заметках о выпуске для требуемой конфигурации состояния для Linux.

###Работа с конфигурациями локально

DSC для Linux включает сценарии для работы с конфигурацией с локального компьютера Linux. Эти скрипты найдите в `/opt/microsoft/dsc/Scripts` и включают следующее:
* GetConfiguration.py
   
   Возвращает текущую конфигурацию, применяемого к компьютеру. Аналогично командлет Get-DscConfiguration командлета Windows PowerShell.

`# sudo./GetConfiguration.py`

* GetMetaConfiguration.py
   
   Возвращает текущую meta конфигурацию, применяемого к компьютеру. Аналогично командлет [Get DSCLocalConfigurationManager](https://technet.microsoft.com/en-us/library/dn407378.aspx) командлета.

`# sudo./GetMetaConfiguration.py`

* InstallModule.py
   
   Устанавливает пользовательский модуль DSC ресурсов. Требуется путь к ZIP-файл, содержащий библиотеку общий объект модуля и схемы MOF-файлы.

`# sudo./InstallModule.py /tmp/cnx_Resource.zip`

* RemoveModule.py
   
   Удаляет пользовательский модуль DSC ресурсов. Требуется имя модуля для удаления.

`# sudo./RemoveModule.py cnx_Resource`

* SendConfigurationApply.py
   
   Применяет MOF-файл конфигурации компьютера. Аналогично [начала DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) командлета. Требуется путь к MOF для применения конфигурации.

`# sudo./RemoveModule.py cnx_Resource`

* SendMetaConfiguration.py
   
   Применяет метаданные конфигурации MOF-файл на компьютере. Аналогично [набора DSCLocalConfigurationManager](https://technet.microsoft.com/en-us/library/dn521621.aspx) командлета. Требуется путь к MOF метаданные конфигурации вступили в силу.

`# sudo./SendMetaConfiguration.py — configurationmof /tmp/localhost.meta.mof`

##PowerShell Настройка требуемого состояния для файлов журнала Linux

Следующие файлы журнала создаются для DSC Linux сообщений.

| Файл журнала| Directory| Описание|
|---|---|---|
| omiserver.log| / opt/omi/var/журнала /| Сообщения, относящиеся к операции OMI CIM-сервер.|
| DSC.log| / opt/omi/var/журнала /| Сообщения, относящиеся к операции операций ресурсов локального диспетчера конфигурации (НОК) и DSC.|




