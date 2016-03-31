#DSC WindowsFeature ресурсов

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

**WindowsFeature** ресурсов в Windows PowerShell требуемой состояние конфигурации (DSC) предоставляет механизм, позволяющий убедиться, что роли и компоненты добавляются или удаляются на целевой узел.

##Синтаксис

```
WindowsFeature [string] #ResourceName
{
    Name = [string]
    [ Credential = [PSCredential] ]
    [ Ensure = [string] { Absent | Present }  ]
    [ IncludeAllSubFeature = [bool] ]
    [ LogPath = [string] ]
    [ DependsOn = [string[]] ]
    [ Source = [string] ]
}
```

##Свойства

| Свойство| Описание|
|---|---|
| Название| Указывает имя роли или компонента, необходимо обеспечить добавляется или удаляется.Это тот же __имя__ свойство из [Get-WindowsFeature](https://technet.microsoft.com/en-us/library/jj205469.aspx) командлет, а не отображаемое имя роли или компонента.|
| Учетные данные| Указывает учетные данные, используемые для добавления или удаления ролей или компонентов.|
| Убедитесь| Указывает, если добавить роль или компонент.Для обеспечения роль или компонент добавлены, установите это свойство для «Отсутствует», чтобы убедиться, что удаляется роль или компонент, задать свойство «Отсутствует».|
| IncludeAllSubFeature| Присвойте этому свойству значение __$true__ для обеспечения состояния всех необходимых компонентов с состоянием функции укажите с __имя__ свойства.|
| LogPath| Указывает путь к файлу журнала место в журнал операции поставщика ресурсов.|
| DependsOn| Указывает, что конфигурация другого ресурса должна выполняться перед настройкой этого ресурса.Например, если идентификатор конфигурации ресурсов сценария блока, который вы хотите запустить сначала — __ResourceName__ и его тип является __типа ресурса__, для использования этого свойства используется синтаксис `DependsOn = "[типа ресурса] ResourceName"`.|
| Источник| Указывает расположение исходного файла для установки при необходимости.|

##Пример

```powershell
WindowsFeature RoleExample
{
    Ensure = "Present" 
    # Alternatively, to ensure the role is uninstalled, set Ensure to "Absent"
    Name = "Web-Server" # Use the Name property from Get-WindowsFeature  
}
```




