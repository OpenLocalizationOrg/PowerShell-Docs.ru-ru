#DSC для Linux nxScript ресурсов

**NxScript** ресурсов в PowerShell требуемой состояние конфигурации (DSC) предоставляет механизм для выполнения сценариев Linux на узле Linux.

##Синтаксис

```
nxScript <string> #ResourceName
{
    GetScript = <string>
    SetScript = <string>
    TestScript = <string>
    [ User = <string> ]
    { Group = <string> ]
    [ DependsOn = <string[]> ]

}
```

##Свойства

| Свойство| Описание|
|---|---|
| GetScript| Содержит сценарий, который выполняется при вызове [Get DscConfiguration](https://technet.microsoft.com/en-us/library/dn521625.aspx) командлета.Скрипт должен начинаться с shebang, например, #! / bin/bash.|
| SetScript| Предоставляет скрипт.При вызове [начала DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) командлета, **TestScript** сначала запускается.Если **TestScript** блок возвращает код выхода, отличное от 0, **SetScript** блок будет запускаться.Если **TestScript** возвращает код выхода 0, **SetScript** не запустится.Скрипт должен начинаться с shebang, такие как `#! / bin/bash`.|
| TestScript| Предоставляет скрипт.При вызове [начала DscConfiguration](https://technet.microsoft.com/en-us/library/dn521623.aspx) этот сценарий выполняет.Если он возвращает код выхода, отличное от 0, SetScript будет работать.Если он возвращает код выхода 0, **SetScript** не запустится. **TestScript** также выполняется при вызове [Тест DscConfiguration](https://technet.microsoft.com/en-us/library/dn407382.aspx) командлета.Однако в данном случае **SetScript** не запустится, независимо от того, какой код выхода возвращается из **TestScript**. **TestScript** должен возвращать код выхода 0, если фактический конфигурации соответствует текущей конфигурации требуемое состояние и кода завершения, отличный от 0, если он не соответствует.(Текущей конфигурации желаемое состояние является последней конфигурации, в силу на узле, который использует DSC).Скрипт должен начинаться с shebang, например 1#!/bin/bash.1|
| Пользователь| Для выполнения скрипта как пользователь.|
| Группы| Для выполнения скрипта как группа.|
| DependsOn| Указывает, что конфигурация другого ресурса должна выполняться перед настройкой этого ресурса.Например если **идентификатор** ресурса блок скрипта конфигурации, необходимо запустить сначала — **ResourceName** и его тип является **типа ресурса**, синтаксис для использования этого свойства `DependsOn = "[типа ресурса] ResourceName"`.|

##Пример

В следующем примере показано использование **nxScript** ресурсов для выполнения дополнительной настройки управления.

```
Import-DSCResource -Module nx 

Node $node {
nxScript KeepDirEmpty{

    GetScript = @"
#!/bin/bash
ls /tmp/mydir/ | wc -l
"@

    SetScript = @"
#!/bin/bash
rm -rf /tmp/mydir/*
"@

    TestScript = @'
#!/bin/bash
filecount=`ls /tmp/mydir | wc -l`
if [ $filecount -gt 0 ]
then
    exit 1
else
    exit 0
fi
'@
} 
}
```




