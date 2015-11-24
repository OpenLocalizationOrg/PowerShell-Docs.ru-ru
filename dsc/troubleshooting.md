#Устранение неполадок DSC

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

В этом разделе описываются методы для получения требуемой конфигурации состояния (DSC) скрипты так, чтобы работать без ошибок. С помощью журналы эффективно для отслеживания ошибок и понимать, как очищает кэш для просмотра результатов изменения ресурсов, можно будет более эффективное устранение неполадок DSC. В двух разделах рассматриваются эти технологии.

* Сценарий не будет работать: **с помощью DSC журналы диагностики ошибок сценария**
* Мои ресурсы не обновляют: **Сброс кэша**

##Сценарий не будет работать: с помощью DSC журналы диагностики ошибок скрипта

Как и все программное обеспечение Windows DSC записывает ошибки и события в [Журналы](https://msdn.microsoft.com/library/windows/desktop/aa363632.aspx) можно просмотреть в [средство просмотра событий](http://windows.microsoft.com/windows/what-information-event-logs-event-viewer). Изучение этих журналов может помочь понять причину сбоя конкретной операции и предотвращения сбоя в будущем. Написание сценариев настройки может быть сложным упростить отслеживание ошибок как автор вы, использование ресурса DSC журнала для отслеживания хода выполнения в конфигурацию в DSC Аналитический журнал событий.

##Где хранятся журналы событий DSC?

В окне просмотра событий DSC события, в: **приложения и состояние конфигурации службы журналы/Microsoft/Windows/желаемый**

Командлет PowerShell соответствующий [Get-WinEvent](https://technet.microsoft.com/library/hh849682.aspx), также можно выполнить просмотр журналов событий:

```
PS C:\Users> Get-WinEvent -LogName "Microsoft-Windows-Dsc/Operational"
   ProviderName: Microsoft-Windows-DSC
TimeCreated                     Id LevelDisplayName Message                                                                                                  
-----------                     -- ---------------- -------                                                                                                  
11/17/2014 10:27:23 PM        4102 Information      Job {02C38626-D95A-47F1-9DA2-C1D44A7128E7} : 
```

Как показано выше, имя элемента DSC основной журнала — **Microsoft -> окна -> DSC** (другие имена журнала в группе Windows не показанных здесь для краткости). Имя первичного добавляется к имени канала для создания полного имени. Записывает ядро DSC главным образом в три типа журналов: [Журнал операций, аналитический и отладочный журналы](https://technet.microsoft.com/library/cc722404.aspx). Поскольку аналитический и журналы отладки по умолчанию отключены, их следует включить в средстве просмотра событий. Чтобы сделать это, откройте средство просмотра событий, введя команду eventvwr в Windows PowerShell; или щелкните **Запуск** щелкните **панели управления**, щелкните **Администрирование**, и нажмите кнопку **средство просмотра событий**. На **представление** меню в окне просмотра событий, нажмите кнопку **Отобразить аналитический и отладочный журналы**. Имя журнала для аналитической канала — **Microsoft-Windows-Dsc-аналитический**, и канал отладки **Microsoft-Windows-Dsc и отладки**. Можно также использовать [wevtutil](https://technet.microsoft.com/library/cc732848.aspx) программы, чтобы включить журналы, как показано в следующем примере.

```powershell
wevtutil.exe set-log “Microsoft-Windows-Dsc/Analytic” /q:true /e:true
```

##Что содержит журналы DSC?

Журналы DSC разбиты на три журнала каналов, в зависимости от степени важности сообщения. Операционный журнал в DSC содержит все сообщения об ошибках и может использоваться для выявления проблем. Аналитический журнал имеет высокий объем событий, а также можно определить, где возникли ошибки. Этот канал также содержит подробные сообщения (если таковые имеются). Журнал отладки содержит журналы, которые помогут понять, как ошибки. Сообщения о событиях DSC структурированы таким образом, чтобы каждое сообщение события начинается с идентификатор задания, который уникальным образом представляет операцию DSC. В приведенном ниже примере предпринимается попытка получить сообщение из первое событие в журнал в журнале операций DSC.

```powershell
PS C:\Users> $AllDscOpEvents=get-winevent -LogName "Microsoft-Windows-Dsc/Operational"
PS C:\Users> $FirstOperationalEvent=$AllDscOpEvents[0]
PS C:\Users> $FirstOperationalEvent.Message
Job {02C38626-D95A-47F1-9DA2-C1D44A7128E7} : 
Consistency engine was run successfully. 
```

DSC события регистрируются в определенной структуры, что позволяет пользователю для статистической обработки событий из одного задания DSC. Структура выглядит следующим образом:

**Идентификатор задания: \ < Guid\ >**
**\ < событие Message\ >**

##Сбор событий от одной операции DSC

Журналы событий DSC содержать события, создаваемые при выполнении различных операций DSC. Однако вы обычно будете со сведениями о беспокоит только одной конкретной операции. Все журналы DSC можно группировать по свойству идентификатора задания, уникальный для каждой операции DSC. Идентификатор задания отображаются в виде все события DSC первого значения свойства. Далее описывается процесс для сбора всех событий в структуре сгруппированные массива.

```powershell
<##########################################################################
 Step 1 : Enable analytic and debug DSC channels (Operational channel is enabled by default)
###########################################################################>

wevtutil.exe set-log “Microsoft-Windows-Dsc/Analytic” /q:true /e:true
wevtutil.exe set-log “Microsoft-Windows-Dsc/Debug” /q:True /e:true

<##########################################################################
 Step 2 : Perform the required DSC operation (Below is an example, you could run any DSC operation instead)
###########################################################################>

Get-DscLocalConfigurationManager

<##########################################################################
Step 3 : Collect all DSC Logs, from the Analytic, Debug and Operational channels
###########################################################################>

$DscEvents=[System.Array](Get-WinEvent "Microsoft-windows-dsc/operational") `
         + [System.Array](Get-WinEvent "Microsoft-Windows-Dsc/Analytic" -Oldest) `
         + [System.Array](Get-Winevent "Microsoft-Windows-Dsc/Debug" -Oldest)


<##########################################################################
 Step 4 : Group all logs based on the job ID
###########################################################################>
$SeparateDscOperations=$DscEvents | Group {$_.Properties[0].value}  
```

В данном случае — переменная `$SeparateDscOperations` содержит журналы, сгруппированных по идентификаторы заданий. Каждый элемент массива этой переменной представляет группу событий, регистрируемых другую операцию DSC, предоставляя доступ к Дополнительные сведения о журналах.

```
PS C:\> $SeparateDscOperations

Count Name                      Group                                                                     
----- ----                      -----                                                                     
   48 {1A776B6A-5BAC-11E3-BF... {System.Diagnostics.Eventing.Reader.EventLogRecord, System.Diagnostics....
   40 {E557E999-5BA8-11E3-BF... {System.Diagnostics.Eventing.Reader.EventLogRecord, System.Diagnostics....
PS C:\> $SeparateDscOperations[0].Group
   ProviderName: Microsoft-Windows-DSC
TimeCreated                     Id LevelDisplayName Message                                               
-----------                     -- ---------------- -------                                               
12/2/2013 3:47:29 PM          4115 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4198 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4114 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4102 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4098 Warning          Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4098 Warning          Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4176 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4182 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4182 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4182 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4182 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4182 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4182 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4182 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...      
12/2/2013 3:47:29 PM          4182 Information      Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : ...       
```

Можно извлекать данные в переменной `$SeparateDscOperations` с помощью [Where-Object](https://technet.microsoft.com/library/ee177028.aspx). Ниже приведены пять ситуаций, в которых может потребоваться извлечь данные для устранения неполадок DSC.

###1: сбои операций

Все события имеют [уровни серьезности] (https://msdn.microsoft.com/library/dd996917 (v=vs.85)). Эти сведения могут использоваться для идентификации события ошибок:

```
PS C:\> $SeparateDscOperations  | Where-Object {$_.Group.LevelDisplayName -contains "Error"}
Count Name                      Group                                                                     
----- ----                      -----                                                                     
   38 {5BCA8BE7-5BB6-11E3-BF... {System.Diagnostics.Eventing.Reader.EventLogRecord, System.Diagnostics....
```

###2: запустите подробные сведения об операциях за последний час половина

`Время создания`, свойство для каждого события Windows, указывает время создания события. Сравнение с объектом определенной даты времени это свойство может использоваться для фильтрации всех событий:

```powershell
PS C:\> $DateLatest=(Get-date).AddMinutes(-30)
PS C:\> $SeparateDscOperations  | Where-Object {$_.Group.TimeCreated -gt $DateLatest}
Count Name                      Group                                                                     
----- ----                      -----                                                                     
    1 {6CEC5B09-5BB0-11E3-BF... {System.Diagnostics.Eventing.Reader.EventLogRecord}   
```

###3: сообщения от последней операции

Последняя операция хранится в первый индекс массива группы `$SeparateDscOperations`. Запрос группы сообщений для индекса 0 возвращает все сообщения для последней операции:

```powershelll
PS C:\> $SeparateDscOperations[0].Group.Message
Job {5BCA8BE7-5BB6-11E3-BF41-00155D553612} : 
Running consistency engine.
Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : 
Configuration is sent from computer NULL by user sid S-1-5-18.
Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : 
Displaying messages from built-in DSC resources:
 WMI channel 1 
 ResourceID:  
 Message : [INCH-VM]:                            [] Starting consistency engine.
Job {1A776B6A-5BAC-11E3-BF41-00155D553612} : 
Displaying messages from built-in DSC resources:
 WMI channel 1 
 ResourceID:  
 Message : [INCH-VM]:                            [] Consistency check completed. 
```

###4: сообщения об ошибках, зарегистрированные для последней операции с ошибками

`$SeparateDscOperations [0]. Группа` содержит набор событий для последней операции. Запустите `Where-Object` командлета для фильтрации событий в зависимости от уровня отображаемому имени. Результаты сохраняются в `$myFailedEvent` переменной, которой можно дальнейшие подлежат Чтобы получить сообщение об ошибке:

```powershell
PS C:\> $myFailedEvent=($SeparateDscOperations[0].Group | Where-Object {$_.LevelDisplayName -eq "Error"})

PS C:\> $myFailedEvent.Message
Job {5BCA8BE7-5BB6-11E3-BF41-00155D553612} : 
DSC Engine Error : 
 Error Message Current configuration does not exist. Execute Start-DscConfiguration command with -Path pa
rameter to specify a configuration file and create a current configuration first. 
Error Code : 1 
```

###5: все события, созданные для конкретного задания идентификатора.

`$SeparateDscOperations` представляет собой массив групп, каждая из которых имеет имя, как уникальный идентификатор задания. Запустив `Where-Object` командлета, можно извлечь эти группы событий, которые имеют идентификатор определенного задания:

```powershell
PS C:\> ($SeparateDscOperations | Where-Object {$_.Name -eq $jobX} ).Group

   ProviderName: Microsoft-Windows-DSC

TimeCreated                     Id LevelDisplayName Message                                               
-----------                     -- ---------------- -------                                               
12/2/2013 4:33:24 PM          4102 Information      Job {847A5619-5BB2-11E3-BF41-00155D553612} : ...      
12/2/2013 4:33:24 PM          4168 Information      Job {847A5619-5BB2-11E3-BF41-00155D553612} : ...      
12/2/2013 4:33:24 PM          4146 Information      Job {847A5619-5BB2-11E3-BF41-00155D553612} : ...      
12/2/2013 4:33:24 PM          4120 Information      Job {847A5619-5BB2-11E3-BF41-00155D553612} : ...  
```

##С помощью xDscDiagnostics для анализа DSC журналы

**xDscDiagnostics** состоит из двух простых операций, которые могут помочь проанализировать DSC сбоев на компьютере модуль PowerShell: `Get xDscOperation` и `трассировки xDscOperation`. Эти функции может помочь выявить все локальные события из предыдущих операций DSC, или DSC на удаленных компьютерах (с действительные учетные данные). В данном случае — термин DSC операция используется для определения уникальных DSC выполнении отдельной от его начала до его конца. Например `DscConfiguration тест` бы отдельной операцией DSC. Аналогичным образом, каждый другой командлет в DSC (такие как `Get DscConfiguration`, `начала DscConfiguration`, т. д.) каждый может быть идентифицировано как отдельные операции DSC. Объясняются в двух командлетов [xDscDiagnostics](https://powershellgallery.com/packages/xDscDiagnostics) модуль PowerShell (DSC Resource Kit) и более подробно ниже. Выполнив доступна справка `Get-Help < имя командлета >`.

##Get-xDscOperation

Этот командлет позволяет найти результаты операций DSC, работающих на один или несколько компьютеров и возвращает объект, содержащий коллекцию событий, полученных при каждой операции DSC. Например в следующем выводе выполнялись три команды. Первый, и другой двух неудачного. Эти результаты обрабатываются в выходных данных `Get xDscOperation`.

TODO: Замените этот рисунок, демонстрирующий Get xDscOperation вывода

###Параметры

* **Новых**: принимает целое число для указания числа операций для отображения. По умолчанию она возвращает 10 новых операций. Например,
   TODO: Показать Get xDscOperation-последние 5
* **Имя_компьютера**: параметр, который принимает массив строк, каждая из которых содержит имя компьютера, из которой вы хотите сбора данных журнала событий DSC. По умолчанию он собирает данные от хост-компьютера. Чтобы включить эту функцию, необходимо запустить следующую команду на удаленных компьютерах, в режиме с повышенными правами, чтобы разрешает сбор событий
```powershell
  New-NetFirewallRule -Name "Service RemoteAdmin" -Action Allow
```
* **Учетные данные**: параметр, который имеет тип PSCredential, благодаря чему доступ к компьютерам, указанный в параметре ComputerName.

###Возвращаемый объект

Командлет возвращает массив объектов типа **Microsoft.PowerShell.xDscDiagnostics.GroupedEvents**. Каждый объект в этом массиве относится к другой операции DSC. Отображение по умолчанию для этого объекта имеет следующие свойства
* **SequenceID**: указывает порядковый номер, назначенный DSC операции на основе времени. Например последнее выполняемой операции будет иметь SequenceID как 1, второй — последняя операция DSC пришлось бы SequenceID 2 и т. д. Этот номер является другой идентификатор для каждого объекта в возвращенном массиве.
* **Время создания**: значение даты и времени, которое указывает начала операции DSC.
* **Имя_компьютера**: имя компьютера, из которых объединяются результаты.
* **Результат**: строка со значением **Сбой** или **успешно** указывающее, если эта операция DSC содержит ошибку или нет, соответственно.
* **AllEvents**: объект, представляющий коллекцию событий, созданные операцией DSC.

Например следующий результат показывает результат последней операции с нескольких компьютеров:
  TODO: Заменить рисунок для Get-xDscOperation отобразить журналы на удаленном компьютере

##XDscOperation трассировки

Этот командлет возвращает объект, содержащий коллекцию событий, их типы событий и выходные сообщения из определенной операции DSC. Как правило, при нахождении сбоя в любых операций с помощью `Get xDscOperation`, будет трассировки, чтобы узнать, какие события вызвали сбой операция.

###Параметры

* **SequenceID**: это целочисленное значение, присваиваемое любой операции, относящиеся к конкретному компьютеру. Указав идентификатор последовательности, скажем, 4, трассировки, четвертый из последней операции DSC будет выдано

Trace-xDscOperation с указанным идентификатором последовательности
* **JobID**: это значение GUID, назначенный xDscOperation НОК для уникальной идентификации операции. Если указано JobID трассировки соответствующая операция DSC выводится.
   TODO: Заменить рисунок для трассировки xDscOperation занимает JobID как параметр
* **Имя_компьютера** и **учетные данные**: эти параметры позволяют трассировки собираются с удаленных компьютеров:
```powershell
New-NetFirewallRule -Name "Service RemoteAdmin" -Action Allow
```
  TODO: Заменить рисунок для запуска на различных вычислительных трассировки xDscOperation

Обратите внимание, что, поскольку `трассировки xDscOperation` объединяет события из аналитика, отладки и рабочие журналы, он предложит включить эти журналы, как описано выше.

###Возвращаемый объект

Командлет возвращает массив объектов типа `Microsoft.PowerShell.xDscDiagnostics.TraceOutput`. Каждый объект в этот массив содержит следующие поля:
* **Имя_компьютера**: имя компьютера, из которых собираются журналы.
* **EventType**: это поле типа перечислителя, содержащий сведения о типе события. Это может быть любой из следующих:
   - *Рабочее*: событие является в журнале операций.
   - *Аналитический*: событие — из Аналитический журнал.
   - *Отладки*: — события из журнала отладки.
   - *Подробно*: результат события в виде подробных сообщений во время выполнения. Подробные сообщения позволяют легко определить последовательность событий, которые публикуются.
   - *Ошибка*: события ошибок. Ищет события ошибок, обычно быстро можно найти причину сбоя.
* **Время создания**: значение DateTime, указывающее, когда событие было зарегистрировано с DSC.
* **Сообщения**: сообщение, которое было зарегистрировано DSC в журналах событий.

Ниже перечислены поля в этом объекте, можно использовать дополнительные сведения о событии, но не отображаются по умолчанию:

* **JobID**: идентификатор задания (формат GUID) для этой операции DSC.
* **SequenceID**: SequenceID, уникальные для этой операции DSC на этом компьютере.
* **Событие**: это фактические событие в журнал DSC типа `System.Diagnostics.Eventing.Reader.EventLogRecord`. Это может также предоставляются с помощью командлета `Get-WinEvent`. Он содержит дополнительные сведения, например задачи, eventID и уровень события.

Кроме того, можно собирать информацию о события путем сохранения выходных данных `трассировки xDscOperation` в переменную. Для отображения всех событий для конкретной операции DSC можно использовать следующую команду:

```powershell
(Trace-xDscOperation -SequenceID 3).Event
```

При этом отобразится те же результаты, что `Get-WinEvent` командлета, например в приведенном ниже вывода:
  TODO: Какие выходные данные?

В идеальном случае сначала применяется `Get xDscOperations` в списке ожидания последнего несколько DSC конфигурации выполняется на компьютерах. После этого можно изучить любые одну операцию (с помощью его sequenceID или JobID) с `трассировки xDscOperation` для обнаружения, как в фоновом.

##Мои ресурсы не обновляют: сброс кэша

Обработчик DSC кэширует ресурсы, которые реализованы в виде модуля PowerShell для повышения эффективности. Однако это может вызвать проблемы при создании ресурса и его тестирования одновременно, поскольку DSC загрузит кэшированная версия до перезапуска процесса. Явно завершить процесс размещение механизма DSC является единственным способом сделать DSC загружена новая версия.

Аналогично, при запуске `начала DSCConfiguration`, после добавления и изменения пользовательских ресурсов, изменение не может выполняться, если или до, после перезагрузки компьютера. Это DSC выполняется в процессе размещения поставщика WMI (WmiPrvSE), так как правило, существует много экземпляров WmiPrvSE сразу. После перезагрузки, перезапуска хост-процесса и кэш очищается.

Для успешного перезапуска конфигурации и очистить кэш без перезагрузки, необходимо остановить и перезапустить процесс узла. Это можно сделать на основе каждого экземпляра, при котором идентифицировать процесс, остановить и перезапустить его. Или можно использовать `DebugMode`, как показано ниже, для перезагрузки PowerShell DSC ресурсов.

Для определения процесса, в котором размещается ядро DSC и остановите его для каждого экземпляра, может содержать идентификатор процесса WmiPrvSE, в которой размещается DSC ядра. Затем, чтобы обновить поставщик, остановить WmiPrvSE процесс, с помощью следующей команды, а затем выполните **начала DscConfiguration** еще раз.

```powershell
###
### find the process that is hosting the DSC engine
###
$dscProcessID = Get-WmiObject msft_providers | 
Where-Object {$_.provider -like 'dsccore'} | 
Select-Object -ExpandProperty HostProcessIdentifier 

###
### Stop the process
###
Get-Process -Id $dscProcessID | Stop-Process
```

##Использование DebugMode

Можно настроить DSC локального Configuration Manager (НОК) для использования `DebugMode` всегда очистки кэша при перезапуске хост-процесса. Если задано значение **TRUE**, он заставляет механизм всегда перезагружать PowerShell DSC ресурсов. После завершения записи ресурса, можно задать для него обратно в **FALSE** и ядро вернется к его поведение кэширования модулей.

Далее приводится Демонстрация Показать как `DebugMode` можно автоматически обновить кэш. Во-первых рассмотрим конфигурацию по умолчанию:

```
PS C:\Users\WinVMAdmin\Desktop> Get-DscLocalConfigurationManager


AllowModuleOverwrite           : False
CertificateID                  : 
ConfigurationID                : 
ConfigurationMode              : ApplyAndMonitor
ConfigurationModeFrequencyMins : 30
Credential                     : 
DebugMode                      : False
DownloadManagerCustomData      : 
DownloadManagerName            : 
LocalConfigurationManagerState : Ready
RebootNodeIfNeeded             : False
RefreshFrequencyMins           : 15
RefreshMode                    : PUSH
PSComputerName                 :  
```

Можно видеть, что `DebugMode` равен **FALSE**.

Чтобы настроить `DebugMode` демонстрации, используйте следующий ресурс PowerShell:

```powershell
function Get-TargetResource
{
    param
    (
        [Parameter(Mandatory)]
        $onlyProperty
    )
    return @{onlyProperty = Get-Content -path "$env:SystemDrive\OutputFromTestProviderDebugMode.txt"}
}
function Set-TargetResource
{
    param
    (
        [Parameter(Mandatory)]
        $onlyProperty
    )
    "1"|Out-File -PSPath "$env:SystemDrive\OutputFromTestProviderDebugMode.txt"
}
function Test-TargetResource
{
    param
    (
        [Parameter(Mandatory)]
        $onlyProperty
    )
    return $false
} 
```

Теперь, создать конфигурацию с помощью выше ресурсу с именем `TestProviderDebugMode`:

```powershell
Configuration ConfigTestDebugMode
{
    Import-DscResource -Name TestProviderDebugMode
    Node localhost
    {
        TestProviderDebugMode test
        {
            onlyProperty = "blah"
        }
    }
}
ConfigTestDebugMode
```

Вы увидите, что содержимое файла: "**$env:SystemDrive\OutputFromTestProviderDebugMode.txt**» является **1**.

Теперь обновите код поставщика, с помощью следующего сценария:

```powershell
$newResourceOutput = Get-Random -Minimum 5 -Maximum 30
$content = @"
function Get-TargetResource
{
    param
    (
        [Parameter(Mandatory)]
        `$onlyProperty
    )
    return @{onlyProperty = Get-Content -path "$env:SystemDrive\OutputFromTestProviderDebugMode.txt"}
}
function Set-TargetResource
{
    param
    (
        [Parameter(Mandatory)]
        `$onlyProperty
    )
    "$newResourceOutput"|Out-File -PSPath C:\OutputFromTestProviderDebugMode.txt
}
function Test-TargetResource
{
    param
    (
        [Parameter(Mandatory)]
        `$onlyProperty
    )
    return `$false
}
"@ | Out-File -FilePath "C:\Program Files\WindowsPowerShell\Modules\MyPowerShellModules\DSCResources\TestProviderDebugMode\TestProviderDebugMode.psm1
```

Этот скрипт создает случайное число и соответствующим образом обновляет поставщика cade. С `DebugMode` значение false, содержимое файла «**$env:SystemDrive\OutputFromTestProviderDebugMode.txt**"никогда не изменяются.

Теперь присвоим параметру `DebugMode` для **TRUE** в скрипте конфигурации:

```powershell
LocalConfigurationManager
{
    DebugMode = $true
} 
```

При запуске приведенный выше скрипт, вы увидите, что содержимое файла отличается каждый раз. (Можно запустить `Get DscConfiguration` его). Ниже приведен результат два дополнительных запуска (на результаты могут отличаться при запуске сценария).

```powershell
PS C:\Users\WinVMAdmin\Desktop> Get-DscConfiguration -CimSession (New-CimSession localhost)

onlyProperty                            PSComputerName                         
------------                            --------------                         
20                                      localhost                              

PS C:\Users\WinVMAdmin\Desktop> Get-DscConfiguration -CimSession (New-CimSession localhost)

onlyProperty                            PSComputerName                         
------------                            --------------                         
14 
```

##См. также

###Ссылка

* [DSC журнала ресурсов](logResource.md)

###Основные понятия

* [Построение пользовательского Windows PowerShell требуемой состояния конфигурации ресурсов](authoringResource.md)

###Другие ресурсы

* [Требуемого состояния конфигурации командлеты Windows PowerShell] (https://technet.microsoft.com/en-us/library/dn521624 (v=wps.630).aspx)




