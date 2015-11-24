#DSC журнала ресурсов

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

__Журнала__ ресурсов в Windows PowerShell требуемой состояние конфигурации (DSC) предоставляет механизм для записи сообщений в журнал событий Microsoft-Windows-желаемый состояние конфигурации и аналитика.

```
Syntax

Log [string] #ResourceName
{
    Message = [string]
    [ DependsOn = [string[]] ]
}
```

##Свойства

| Свойство| Описание|
|---|---|
| Сообщение| Указывает сообщение, которое требуется выполнить запись в журнал событий Microsoft-Windows-Desired состояние конфигурации и аналитика.|
| DependsOn| Указывает, что конфигурация другого ресурса необходимо выполнить это сообщение журнала записывается.Например, если идентификатор конфигурации ресурсов сценария блока, который вы хотите запустить сначала — __ResourceName__ и его тип является __типа ресурса__, для использования этого свойства используется синтаксис `DependsOn = "[типа ресурса] ResourceName"`.|

##Пример

Следующий пример показано, как включить сообщение в журнале событий Microsoft-Windows-Desired состояние конфигурации и аналитика.

> **Примечание**: при запуске [Тест DscConfiguration](https://technet.microsoft.com/en-us/library/dn407382.aspx) этот ресурс настроен, оно всегда будет возвращать **$false**.

```powershell 
Log LogExample
{
    Message = "This message will appear in the Microsoft-Windows-Desired State Configuration/Analytic event log."
} 
```





