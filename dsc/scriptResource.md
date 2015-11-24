#DSC сценария ресурса

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

**Сценарий** ресурсов в Windows PowerShell требуемой состояние конфигурации (DSC) предоставляет механизм для выполнения блоков сценария Windows PowerShell на целевые узлы.

##Синтаксис

```
Script [string] #ResourceName
{
    GetScript = [string]
    SetScript = [string]
    TestScript = [string]
    [ Credential = [PSCredential] ]
    [ DependsOn = [string[]] ]
}
```

##Свойства

| Свойство| Описание|
|---|---|
| GetScript| Предоставляет блок сценария Windows PowerShell, который выполняется при вызове [Get DscConfiguration](https://technet.microsoft.com/en-us/library/dn407379.aspx) командлета.Этот блок должен возвращать хэш-таблицы.|
| SetScript| Предоставляет блок сценария Windows PowerShell.При вызове [начала DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) командлета, **TestScript** блок выполняется первой.Если **TestScript** блок возвращает **$false**,  **SetScript** блок будет запускаться.Если **TestScript** блок возвращает **$true**,  **SetScript** блок не будет запускаться.|
| TestScript| Предоставляет блок сценария Windows PowerShell.При вызове [начала DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) командлета, этот блок выполняется.Если он возвращает **$false**, SetScript блок будет выполняться.Если он возвращает **$true**, SetScript, блок будет выполняться. **TestScript** блок выполняется при вызове [Тест DscConfiguration](https://technet.microsoft.com/en-us/library/dn407382.aspx) командлета.Однако в данном случае **SetScript** не выполняется, независимо от того, какое значение TestScript блок возвращает блока. **TestScript** блок должен возвращает True, если реальной конфигурации соответствует текущей конфигурации желаемое состояние и False, если не совпадает.(Текущая конфигурация желаемое состояние является последней конфигурации, в силу на узле, который использует DSC.)|
| Учетные данные| Указывает учетные данные, используемые для запуска этого скрипта, если необходимы учетные данные.|
| DependsOn| Указывает, что конфигурация другого ресурса должна выполняться перед настройкой этого ресурса.Например, если идентификатор конфигурации ресурсов сценария блока, который вы хотите запустить сначала — **ResourceName** и его тип является **типа ресурса**, для использования этого свойства используется синтаксис `DependsOn = "[типа ресурса] ResourceName"`.

##Пример

```powershell
Script ScriptExample
{
    SetScript = { 
        $sw = New-Object System.IO.StreamWriter("C:\TempFolder\TestFile.txt")
        $sw.WriteLine("Some sample string")
        $sw.Close()
    }
    TestScript = { Test-Path "C:\TempFolder\TestFile.txt" }
    GetScript = { <# This must return a hash table #> }          
}
```





