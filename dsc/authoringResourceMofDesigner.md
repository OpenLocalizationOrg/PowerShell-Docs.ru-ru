#С помощью средства конструктора ресурсов

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

Средство конструктора ресурсов — это набор командлетов, предоставляемых **xDscResourceDesigner** модуль, облегчить создание ресурсов конфигурации состояния требуемой Windows PowerShell (DSC). Командлеты в этот ресурс помогают создать схему MOF, модуль сценариев и структуру каталогов для нового ресурса. Дополнительные сведения о ресурсах DSC см. в разделе [построения пользовательского Windows PowerShell требуемой состояние конфигурации ресурсов](authoringResource.md).
В этом разделе мы создадим DSC ресурс, который управляет пользователей Active Directory.
Используйте [установки модуля](https://technet.microsoft.com/en-us/library/dn807162.aspx) командлет, чтобы установить **xDscResourceDesigner** модуля.

> **Примечание**: **установки модуля** включается в **PowerShellGet** модуль, который включается в PowerShell 5.0. Можно загрузить **PowerShellGet** модуля PowerShell 3.0 и 4.0 на [предварительного просмотра модули PowerShell PackageManagement](https://www.microsoft.com/en-us/download/details.aspx?id=49186).

##Создание свойств ресурсов

Первое, что необходимо сделать — определить свойства, предоставляющие ресурс. Например определим пользователя Active Directory со следующими свойствами.

Имя параметра описание
* **Пользователя**: ключевое свойство, которое однозначно определяет пользователя.
* **Убедитесь, что**: задает учетную запись пользователя должна присутствовать, или отсутствует. Этот параметр будет иметь только два значения.
* **DomainCredential**: пароль для пользователя домена.
* **Пароль**: нужный пароль пользователя можно определить конфигурацию для изменения пароля пользователя при необходимости.

Для создания свойства, мы используем **Создать xDscResourceProperty** командлета. Следующие команды PowerShell создают свойства, описанные выше.

```powershell
PS C:\> $UserName = New-xDscResourceProperty –Name UserName -Type String -Attribute Key
PS C:\> $Ensure = New-xDscResourceProperty –Name Ensure -Type String -Attribute Write –ValidateSet “Present”, “Absent”
PS C:\> $DomainCredential = New-xDscResourceProperty –Name DomainCredential-Type PSCredential -Attribute Write
PS C:\> $Password = New-xDscResourceProperty –Name Password -Type PSCredential -Attribute Write
```

##Создание ресурса

Теперь, когда были созданы свойства ресурса, можно вызвать **Создать xDscResource** командлет, чтобы создать новый ресурс.  **XDscResource создать** выводит список свойств в качестве параметров. Она также использует путь, где необходимо создать модуль, имя нового ресурса и имя модуля, в котором он содержится. Следующая команда PowerShell создает ресурс.

```powershell
PS C:\> New-xDscResource –Name Demo_ADUser –Property $UserName, $Ensure, $DomainCredential, $Password –Path ‘C:\Program Files\WindowsPowerShell\Modules’ –ModuleName Demo_DSCModule
```

**XDscResource создать** создает схему MOF, сценарий сетевых ресурсов, структура каталогов, необходимые для нового ресурса и манифест для модуля, содержащегося в новый ресурс.

MOF-файл схемы находится в **C:\Program Files\WindowsPowerShell\Modules\Demo_DSCModule\DSCResources\Demo_ADUser\Demo_ADUser.schema.mof**, и его содержимое следующим образом.

```
[ClassVersion("1.0.0.0"), FriendlyName("Demo_ADUser")]
class Demo_ADUser : OMI_BaseResource
{
[Key] string UserName;
[Write, ValueMap{"Present","Absent"}, Values{"Present","Absent"}] string Ensure;
[Write, EmbeddedInstance("MSFT_Credential")] String DomainCredential;
[Write, EmbeddedInstance("MSFT_Credential")] String Password;
};
```

Сценарий ресурса находится в **C:\Program Files\WindowsPowerShell\Modules\Demo_DSCModule\DSCResources\Demo_ADUser\Demo_ADUser.psm1**. Он не содержит фактическую логику для реализации ресурс, который необходимо добавить текущего пользователя. Содержимое сценария сетевых выглядят следующим образом.

```powershell
function Get-TargetResource
{
[CmdletBinding()]
[OutputType([System.Collections.Hashtable])]
param
(
[parameter(Mandatory = $true)]
[System.String]
$UserName
)

#Write-Verbose "Use this cmdlet to deliver information about command processing."

#Write-Debug "Use this cmdlet to write debug information while troubleshooting."


<#
$returnValue = @{
UserName = [System.String]
Ensure = [System.String]
DomainAdminCredential = [System.Management.Automation.PSCredential]
Password = [System.Management.Automation.PSCredential]
}

$returnValue
#>
}


function Set-TargetResource
{
[CmdletBinding()]
param
(
[parameter(Mandatory = $true)]
[System.String]
$UserName,

[ValidateSet("Present","Absent")]
[System.String]
$Ensure,

[System.Management.Automation.PSCredential]
$DomainAdminCredential,

[System.Management.Automation.PSCredential]
$Password
)

#Write-Verbose "Use this cmdlet to deliver information about command processing."

#Write-Debug "Use this cmdlet to write debug information while troubleshooting."

#Include this line if the resource requires a system reboot.
#$global:DSCMachineStatus = 1


}


function Test-TargetResource
{
[CmdletBinding()]
[OutputType([System.Boolean])]
param
(
[parameter(Mandatory = $true)]
[System.String]
$UserName,

[ValidateSet("Present","Absent")]
[System.String]
$Ensure,

[System.Management.Automation.PSCredential]
$DomainAdminCredential,

[System.Management.Automation.PSCredential]
$Password
)

#Write-Verbose "Use this cmdlet to deliver information about command processing."

#Write-Debug "Use this cmdlet to write debug information while troubleshooting."


<#
$result = [System.Boolean]

$result
#>
}


Export-ModuleMember -Function *-TargetResource
```

##Обновление ресурса

Если необходимо добавить или изменить список параметров ресурса, можно вызвать **xDscResource обновление** командлета. Командлет обновляет ресурс новый список параметров. Если в скрипте ресурсов уже добавлена логика, остается без изменений.

Например предположим, что требуется включить последний журнал времени для пользователя в нашем ресурсов. Вместо того чтобы повторно полностью записи ресурса, можно вызвать **Создать xDscResourceProperty** для создания нового свойства, а затем вызвать **xDscResource обновление** и добавить новое свойство в список свойств.

```powershell
PS C:\> $lastLogon = New-xDscResourceProperty –Name LastLogon –Type Hashtable –Attribute Write –Description “For mapping users to their last log on time”
PS C:\> Update-xDscResource –Name ‘Demo_ADUser’ –Property $UserName, $Ensure, $DomainCredential, $Password, $lastLogon -Force
```

##Проверка схемы ресурсов

Конструктор ресурсов средство предоставляет один дополнительные командлет, который может использоваться для проверки правильности схемы MOF, написанный вручную. Вызовите **Тест xDscSchema** командлета, передав в качестве параметра путь схемы ресурсов MOF. Командлет будет выводить все ошибки схемы.

###См. также

####Основные понятия

[Построение пользовательского Windows PowerShell требуемой состояния конфигурации ресурсов](authoringResource.md)

####Другие ресурсы

[Модуль xDscResourceDesigner](https://powershellgallery.com/packages/xDscResourceDesigner)




