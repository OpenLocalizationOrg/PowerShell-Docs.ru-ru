#Требуемая конфигурация состояние PowerShell частичного конфигураций

> Область применения: Windows PowerShell 5.0

В PowerShell 5.0 требуемой конфигурации состояния (DSC) позволяет конфигураций доставки во фрагментах и из нескольких источников. Локальный диспетчер конфигурации (НОК) на целевом узле составляет фрагментов перед их применением в одной конфигурации. Эта возможность позволяет совместное управление конфигурации между группам или отдельным пользователям. Например при двух или нескольких команд разработчиков совместная работа в службе, они каждого можете для создания конфигураций для управления их часть службы. Каждый из этих конфигураций может изымается из разных по запросу серверов, и они могут быть добавлены на различных этапах разработки. Частичное конфигурации также позволяют различных пользователям или группам контролировать различные аспекты настройки узлов без необходимости координировать редактирования документа одной конфигурации. Например одна группа может быть ответственность за развертывание ВМ и операционной системы, а другой команды развернуть другие приложения и службы на эту виртуальную Машину. С частичным конфигураций каждая команда может создать собственную конфигурацию без любого из них будет излишне сложным.

Можно использовать частичные конфигураций в режим push, режим по запросу или сочетание этих двух.

##Частичное настройки в режиме push

Использовать частичную конфигураций в режим push, настройте НОК на целевом узле для получения частичной конфигурации. Каждой частичной конфигурации должна помещаться в стек в целевой объект с помощью командлета DSCConfiguration публикации. Целевой узел затем объединяет частичной конфигурации в одной конфигурации и конфигурации можно применять путем вызова [начала DscConfigurationxt](https://technet.microsoft.com/en-us/library/dn521623.aspx) командлета.

###Настройка НОК для конфигураций частичного принудительном режиме

Чтобы настроить режим push НОК для частичного конфигураций, создать **DSCLocalConfigurationManager** конфигурации с одним **PartialConfiguration** блок для каждого частичного конфигурации. Дополнительные сведения о настройке НОК см. в разделе [Настройка локальный диспетчер конфигурации Windows](https://technet.microsoft.com/en-us/library/mt421188.aspx). В следующем примере показано НОК конфигурации, ожидающую две частичные конфигурации — один для развертывания операционной системы и развертывает и настраивает SharePoint.

```powershell
[DSCLocalConfigurationManager()]
configuration PartialConfigDemo
{
    Node localhost
    {

           PartialConfiguration OSInstall
        {
            Description = 'Configuration for the Base OS'
            RefreshMode = 'Push'
        }
           PartialConfiguration SharePointConfig
        {
            Description = 'Configuration for the SharePoint server'
            RefreshMode = 'Push'
        }
    }
}
PartialConfigDemo 
```

**RefreshMode** для каждого частичного конфигурации задано значение «Push». Имена **PartialConfiguration** блоки (в данном случае «OSInstall» и «SharePointConfig») должны совпадать с именами точно конфигурации, которые передаются на целевой узел.

###Публикация и запуск конфигурации частичного принудительном режиме

![Структура папок PartialConfig](./images/PartialConfig1.jpg)

Затем вызовите **публикации DSCConfiguration** для каждой конфигурации, передав папки содержат документы конфигурации как параметров пути. После публикации обеих конфигураций, можно вызвать `DSCConfiguration начала — UseExisting` на целевом узле.

##Частичное настройки в режиме по запросу

Частичное конфигурации может быть извлечено из одного или нескольких серверов по запросу (Дополнительные сведения о серверах по запросу см. в разделе [Windows PowerShell требуемой конфигурации по запросу серверам состояния](pullServer.md). Чтобы сделать это, необходимо настроить НОК на целевом узле частичной конфигурации и имя и правильно найдите документы конфигурации на серверах по запросу.

###Настройка НОК для конфигурации узлов по запросу

Чтобы настроить НОК для извлечения частичной конфигурации с сервера по запросу, определить сервера по запросу на **ConfigurationRepositoryWeb** (для HTTP-сервером по запросу) или **ConfigurationRepositoryShare** (для сервера по запросу SMB) блока. Затем создайте **PartialConfiguration** блоки, ссылки на сервер по запросу с помощью **ConfigurationSource** свойства. Необходимо также создать блок параметры для указания, что НОК использует режим по запросу, а также указать ConfigurationID, позволяет определить конфигурации сервера по запросу и целевого узла. В следующей конфигурации meta определяет сервер по запросу HTTP с именем CONTOSO PullSrv и две частичные конфигурации, использующих, по запросу сервера.

```powershell
[DSCLocalConfigurationManager()]
configuration PartialConfigDemo
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

           PartialConfiguration OSInstall
        {
            Description = 'Configuration for the Base OS'
            ConfigurationSource = '[ConfigurationRepositoryWeb]CONTOSO-PullSrv'
            RefreshMode = 'Pull'
        }
           PartialConfiguration SharePointConfig
        {
            Description = 'Configuration for the Sharepoint Server'
            ConfigurationSource = '[ConfigurationRepositoryWeb]CONTOSO-PullSrv'
            DependsOn = [PartialConfiguration]OSInstall
            RefreshMode = 'Pull'
        }
    }
}
PartialConfigDemo 
```

Можно извлечь частичной конфигурации из более чем одного сервера по запросу — просто необходимо определить каждый сервер по запросу, а затем ссылаться на сервере соответствующие запросу в каждом блоке PartialConfiguration.

После создания метаданные конфигурации, необходимо запустить его, чтобы создать документ конфигурации (MOF-файл), а затем вызвать набор [DscLocalConfigurationManager] (https://technet.microsoft.com/en-us/library/dn521621 (v=wps.630).aspx), чтобы настроить НОК.

###Присвоение имени и размещения документов настройки на сервере по запросу

Документы частичной конфигурации должны быть расположены в папке, указанной как **ConfigurationPath** в `web.config` файл для сервера по запросу (обычно `C:\Program Files\WindowsPowerShell\DscService\Configuration`). Документы конфигурации должен называться следующим образом: _ConfigurationName_. _ConfigurationID_`.mof`, где _ConfigurationName_ имя частичного конфигурации и _ConfigurationID_ идентификатор конфигурации определяется в НОК на целевом узле. В нашем примере конфигурации документы должны быть именами следующим образом.
![Имена PartialConfig на сервере по запросу](images/PartialConfigPullServer.jpg)

###Программа частичного конфигураций на сервере по запросу

После НОК на целевом узле был настроен и конфигурации документы были созданы и правильное имя на сервере по запросу, целевой узел по запросу частичного конфигураций, объедините их и применять Результирующая конфигурация через регулярные интервалы в соответствии с **RefreshFrequencyMins** свойство НОК. Для принудительного обновления можно вызвать командлет Update-DscConfiguration для извлечения конфигурации, а затем `DSCConfiguration начала — UseExisting` их применения.

##Частичное конфигураций в смешанном режиме по запросу и принудительная

Можно также смешивать push и pull режимы для частичного конфигураций. То есть может иметь один частичной конфигурации, получаемым с сервера по запросу, пока другой конфигурации частичного помещается. Каждой частичной конфигурации следует Рассматривайте как, в зависимости от его режим обновления, как описано в предыдущих разделах. Например следующая конфигурация метаданных описываются тот же пример с частичной конфигурации операционной системы в режиме по запросу и частичные конфигурации SharePoint в режиме push.

```powershell
[DSCLocalConfigurationManager()]
configuration PartialConfigDemo
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

           PartialConfiguration OSInstall
        {
            Description = 'Configuration for the Base OS'
            ConfigurationSource = '[ConfigurationRepositoryWeb]CONTOSO-PullSrv'
            RefreshMode = 'Pull'
        }
           PartialConfiguration SharePointConfig
        {
            Description = 'Configuration for the Sharepoint Server'
            DependsOn = [PartialConfiguration]OSInstall
            RefreshMode = 'Push'
        }
    }
}
PartialConfigDemo 
```

Обратите внимание, что **RefreshMode** в блоке параметров является «По запросу», но **RefreshMode** для OSInstall частичной конфигурации — «Push».

Необходимо имя и найдите документы, конфигурации, как описано выше, для их обновления соответствующих режимов. Можно вызвать **публикации DSCConfiguration** для публикации SharePointInstall частичной конфигурации и либо дождитесь конфигурации OSInstall, полученных с сервера по запросу или принудительное обновление, вызвав Update [DscConfiguration] (https://technet.microsoft.com/en-us/library/mt143541 (v=wps.630).aspx).

##См. также

**Основные понятия**
[Windows PowerShell требуемой конфигурации состояния серверов по запросу](pullServer.md) 
[Настройка конфигурации диспетчера локальных Windows](https://technet.microsoft.com/en-us/library/mt421188.aspx)




