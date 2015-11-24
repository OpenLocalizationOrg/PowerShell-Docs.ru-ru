#DSC для Linux nxGroup ресурсов

**NxGroup** ресурсов в PowerShell требуемой состояние конфигурации (DSC) предоставляет механизм для управления локальными группами на узле Linux.

##Синтаксис

```powershell
nxGroup <string> #ResourceName
{
    GroupName = <string>
    [ Ensure = <string> { Absent | Present }  ]
    [ Members = <string[]> ]
    [ MebersToInclude = <string[]>]
    [ MembersToExclude = <string[]> ]
    [ DependsOn = <string[]> ]
}
```

##Свойства

| Свойство| Описание|
|---|---|
| Имя группы| Указывает имя группы, для которого требуется обеспечить определенное состояние.|
| Убедитесь| Определяет, нужно ли проверять наличие группы.Установите это свойство для «Отсутствует», чтобы убедиться, что группа существует.Задайте для него значение «Отсутствует», чтобы убедиться, что группа не существует.Значение по умолчанию — «Отсутствует».|
| Члены| Задает элементы, которые образуют группу.|
| MembersToInclude| Указывает, что для пользователей, которым требуется обеспечить являются членами группы.|
| MembersToExclude| Указывает, что для пользователей, которым требуется обеспечить не являются членами группы.|
| PreferredGroupID| Задает идентификатор группы указанное значение, если это возможно.Если идентификатор группы используется в настоящий момент, используется следующий идентификатор группы доступны.|
| DependsOn| Указывает, что конфигурация другого ресурса должна выполняться перед настройкой этого ресурса.Например если **идентификатор** ресурса блок скрипта конфигурации, необходимо запустить сначала — **ResourceName** и его тип является **типа ресурса**, синтаксис для использования этого свойства `DependsOn = "[типа ресурса] ResourceName"`.|

##Пример

В следующем примере проверяется, что пользователь «monuser» существует и является членом группы «DBusers».

```
Import-DSCResource -Module nx 

Node $node {

nxUser UserExample{
   UserName = "monuser"
   Description = "Monitoring user"
   Password  =    '$6$fZAne/Qc$MZejMrOxDK0ogv9SLiBP5J5qZFBvXLnDu8HY1Oy7ycX.Y3C7mGPUfeQy3A82ev3zIabhDQnj2ayeuGn02CqE/0'
   Ensure = "Present"
   HomeDirectory = "/home/monuser"
}

nxGroup GroupExample{
   GroupName = "DBusers"
   Ensure = "Present"
   MembersToInclude = "monuser"
   DependsOn = "[nxUser]UserExample"            
}
}
```




