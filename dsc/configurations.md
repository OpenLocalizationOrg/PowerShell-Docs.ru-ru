#DSC конфигураций

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

DSC конфигурации являются сценариев PowerShell, которые определяют специальный тип функции. 
Для определения конфигурации, используйте ключевое слово PowerShell __Конфигурация__.

```powershell
Configuration MyDscConfiguration {

    Node “TEST-PC1” {
        WindowsFeature MyFeatureInstance {
            Ensure = “Present”
            Name =  “RSAT”
        }
        WindowsFeature My2ndFeatureInstance {
            Ensure = “Present”
            Name = “Bitlocker”
        }
    }
}
```

Сохраните скрипт как ps1-файл.

##Синтаксис конфигурации

Скрипт конфигурации состоит из следующих частей:

- **Конфигурация** блока. Это блок скрипта внешней. Можно определить с помощью **Конфигурация** ключевое слово и предоставление имени. В этом случае имя конфигурации — «MyDscConfiguration».
- Один или несколько **узел** блоков. Эти определения узлов (компьютеров или виртуальных машин), которые вы настраиваете. В представленной выше конфигурации есть **узел** блока, ориентированном на компьютере с именем «Тест ПК1».
- Один или несколько блоков ресурсов. Это происходит, где конфигурации задает свойства для ресурсов, которые его настройки. В этом случае существует два ресурса блоков, каждый из которых вызов ресурса «WindowsFeature».

В **Конфигурация** блока, можно сделать все, что обычно может в функции PoweShell. Например в предыдущем примере, если вы не хотите вписывать имя конечного компьютера в конфигурации, можно добавить параметр для имени узла:

```powershell
Configuration MyDscConfiguration {

    param(
        [string[]]$computerName="localhost"
    )
    Node $computerName {
        WindowsFeature MyFeatureInstance {
            Ensure = “Present”
            Name =  “RSAT”
        }
        WindowsFeature My2ndFeatureInstance {
            Ensure = “Present”
            Name = “Bitlocker”
        }
    }
}
```

В этом примере укажите имя узла, передав в качестве параметра $computerName при вы [компиляции конфигурации] (# компиляции конфигурации). По умолчанию используется имя «localhost».

##Компиляция конфигурации

Перед можно внедрения конфигурацию, необходимо скомпилировать его в документ MOF. Это делается путем вызова конфигурации так же, как функция PowerShell.
> __Примечание:__ вызов конфигурацию, она должна быть в глобальной области видимости (как и любой другой функции PowerShell). Можно сделать это происходит либо по «точкой» скрипта, или выполнив скрипт конфигурации с помощью F5 или щелкнув __запустить скрипт__ кнопку в ISE. Точка источником скрипт, выполните команду ". .\myConfig.ps1` где `myConfig.ps1 "имя файла сценария, содержащего конфигурацию.

Он создает при вызове конфигурации:

- Папка в текущем каталоге с тем же именем, как конфигурации.
- Файл с именем _имя_узла_.mof в новый каталог, где _имя_узла_ имя целевого узла конфигурации. Если несколько узлов, MOF-файл создается для каждого узла.

> __Примечание__: MOF-файл содержит все сведения о конфигурации для целевой узел. По этой причине важно обеспечить безопасность. Дополнительные сведения см. в разделе [Защита MOF-файл](secureMOF.md).

Компиляция первой конфигурации выше результаты в следующую структуру папок:

```powershell
PS C:\users\default\Documents\DSC Configurations> . .\MyDscConfiguration.ps1
PS C:\users\default\Documents\DSC Configurations> MyDscConfiguration
    Directory: C:\users\default\Documents\DSC Configurations\MyDscConfiguration
Mode                LastWriteTime         Length Name                                                                                              
----                -------------         ------ ----                                                                                         
-a----       10/23/2015   4:32 PM           2842 TEST-PC1.mof
```

Если конфигурация принимает параметр, как показано во втором примере, должно быть предоставлено во время компиляции. Вот как выглядит.

```powershell
PS C:\users\default\Documents\DSC Configurations> . .\MyDscConfiguration.ps1
PS C:\users\default\Documents\DSC Configurations> MyDscConfiguration -computerName 'MyTestNode'
    Directory: C:\users\default\Documents\DSC Configurations\MyDscConfiguration
Mode                LastWriteTime         Length Name                                                                                              
----                -------------         ------ ----                                                                                         
-a----       10/23/2015   4:32 PM           2842 MyTestNode.mof
```

##С помощью DependsOn

Полезные ключевое слово DSC __DependsOn__. Обычно (но не обязательно), применяется DSC ресурсов в том порядке, в котором они появляются в конфигурации. Однако __DependsOn__ определяет, какие ресурсы зависят от других ресурсов и НОК гарантирует, что они применяются в правильном порядке, независимо от порядка, в какой ресурс определенных экземпляров. Например, конфигурация может указать, что экземпляр __пользователя__ ресурс зависит от существования __группы__ экземпляра:

```powershell
Configuration DependsOnExample {
    Node Test-PC1 {
        Group GroupExample {
            Ensure = “Present”
            GroupName = “TestGroup”
        }
User UserExample {
Ensure = “Present”
FullName = “TestUser”
DependsOn = “GroupExample”
}
    }
}
```

##С помощью новых ресурсов в конфигурации

При запуске предыдущих примерах, можно заметить, были предупреждение без явного импорта использовали ресурса.
В настоящее время DSC поставляется с 12 ресурсы в рамках модуля PSDesiredStateConfiguration. Другие ресурсы во внешних модулях, которые должны быть помещены в `$env: PSModulePath` чтобы распознаваться НОК. Новый командлет [Get DscResource](https://technet.microsoft.com/en-us/library/dn521625.aspx), можно использовать, чтобы определить, какие ресурсы установлены в системе и доступны для использования с НОК. 
Когда эти модули были помещены в `$env: PSModulePath` и правильно распознаваться [Get DscResource](https://technet.microsoft.com/en-us/library/dn521625.aspx), они по-прежнему должны быть загружены в конфигурацию. __DscResource импорта__ динамическое ключевое слово, которое можно распознать за __Конфигурация__ блока (т. е. не командлет). __DscResource импорта__ поддерживает два параметра:
* __ModuleName__ рекомендуемый способ использования __DscResource импорта__. Он принимает имя модуля, содержащего ресурсы для импорта (а также строковый массив пространств имен модулей).
* __Имя__ имя ресурса для импорта. Это понятное имя, возвращенное как «Имя» [Get DscResource](https://technet.microsoft.com/en-us/library/dn521625.aspx), имя класса, когда определение схемы ресурсов, но (возвращаются в виде __типа ресурса__ по [Get DscResource](https://technet.microsoft.com/en-us/library/dn521625.aspx)).

##См. также

* [Общие сведения о состоянии конфигурации требуемого Windows PowerShell](overview.md)
* [Ресурсы DSC](resources.md)
* [Настройка локальных Configuration Manager](metaconfig.md)


