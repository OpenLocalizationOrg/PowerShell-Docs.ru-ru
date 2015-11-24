#Написание настраиваемых DSC ресурсов с помощью MOF

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

В этом разделе мы определения схемы для настраиваемого ресурса конфигурации состояния требуемой Windows PowerShell (DSC) в MOF-файле и реализации ресурс в файле сценария Windows PowerShell. Для создания и поддержки веб-сайта — это настраиваемый ресурс.

##Создание схемы MOF

Схема определяет свойства ресурса, можно настроить с помощью DSC сценария конфигурации.

###Структура папок для ресурса MOF

Чтобы реализовать настраиваемый ресурс DSC со схемой MOF, создайте следующую структуру папок. Схема MOF определяется в файле демонстрации_IISWebsite.schema.mof и скрипт ресурсов определяется в демонстрации_IISWebsite.ps1. При необходимости можно создать файл манифеста (psd1) модуля.

```
$env: psmodulepath (folder)
    |- MyDscResources (folder)
        |- DSCResources (folder)
            |- Demo_IISWebsite (folder)
                |- Demo_IISWebsite.psd1 (file, optional)
                |- Demo_IISWebsite.psm1 (file, required)
                |- Demo_IISWebsite.schema.mof (file, required)
```

Обратите внимание, что это необходимо для создания папки с именем DSCResources в папке верхнего уровня, в папке для каждого ресурса, должен иметь имя, совпадающее с именем ресурса.

###Содержимое файла MOF

Ниже приведен пример файла MOF, который может использоваться для ресурса настраиваемого веб-сайта. Чтобы выполнить этот пример, сохраните эту схему в файл и назвать файл *Demo_IISWebsite.schema.mof*.

```
[ClassVersion("1.0.0"), FriendlyName("Website")] 
class Demo_IISWebsite : OMI_BaseResource
{
  [Key] string Name;
  [Required] string PhysicalPath;
  [write,ValueMap{"Present", "Absent"},Values{"Present", "Absent"}] string Ensure;
  [write,ValueMap{"Started","Stopped"},Values{"Started", "Stopped"}] string State;
  [write] string Protocol[];
  [write] string BindingInfo[];
  [write] string ApplicationPool;
  [read] string ID;
};
```

Обратите внимание на следующие сведения о предыдущем коде:

* `FriendlyName` определяет имя, можно использовать для ссылки на этот настраиваемый ресурс в сценариях DSC конфигурации. В этом примере `веб-сайт` эквивалентен понятное имя `архив` для встроенных ресурсов архива.
* В классе можно определить настраиваемый ресурс должен быть производным от `OMI_BaseResource`.
* Квалификатор типа `[ключ]`, на свойство указывает, что это свойство будет уникально определять экземпляр ресурса. Объект `[ключ]` свойство также является обязательным.
* `[Обязательно]` квалификатор указывает, что свойство является обязательным (значение необходимо указать в любой скрипт конфигурации, использующий этот ресурс).
* `[Записи]` квалификатор указывает, что это свойство является необязательным при использовании настраиваемого ресурса в сценарий настройки.  `[Чтение]` квалификатор указывает, что свойство не может установить конфигурацию и является только в целях отчетности.
* `Значения` ограничивает значения, которые могут быть присвоены свойству в список значений, определенных в `ValueMap`. Дополнительные сведения см. в разделе [ValueMap и квалификаторы значение](https://msdn.microsoft.com/library/windows/desktop/aa393965.aspx).
* Включая свойство называется `Убедитесь, что` в ресурс рекомендуется в качестве способа обеспечить согласованный стиль с помощью встроенных DSC ресурсов.
* Имя файла схемы для настраиваемого ресурса следующим образом: `classname.schema.mof`, где `classname` — идентификатор, который соответствует `класс` ключевое слово в определении схемы.

###Написание скрипта ресурсов

Сценарий ресурса реализует логику ресурса. В этом модуле должен включать три функции, которые вызываются **Get TargetResource**, **набора TargetResource**, и **Тест TargetResource**. Все три функции необходимо выполнить набор параметров, идентичный набор свойств, определенных в схеме MOF, созданный для этого ресурса. В этом документе данный набор свойств называется «свойства ресурса». Вызывается эти три функции в файле хранилища <ResourceName>.psm1. В следующем примере функции хранятся в файле с именем Demo_IISWebsite.psm1.

> **Примечание**: при выполнении один и тот же сценарий конфигурации на ресурс более одного раза, должно появиться без ошибок и ресурс должен оставаться в том же состоянии, как после выполнения скрипта. Для этого убедитесь, что вашей **Get TargetResource** и **TargetResource тест** функции оставьте без изменений ресурса и этот вызов **TargetResource набор** работать более чем один раз в последовательности с тем же параметром значения всегда эквивалентно вызову его один раз.

В **Get TargetResource** функции реализации, используйте значения свойств ключа ресурса, которые предоставляются как параметры для проверки состояния экземпляра указанного ресурса. Это функция должна возвращать хэш-таблицы, в котором перечислены все свойства ресурса как ключи и фактические значения этих свойств с соответствующими значениями. Ниже приведен пример.

```powershell
# DSC uses the Get-TargetResource function to fetch the status of the resource instance specified in the parameters for the target machine
function Get-TargetResource 
{
    param 
    (       
        [ValidateSet("Present", "Absent")]
        [string]$Ensure = "Present",

        [Parameter(Mandatory)]
        [ValidateNotNullOrEmpty()]
        [string]$Name,

        [Parameter(Mandatory)]
        [ValidateNotNullOrEmpty()]
        [string]$PhysicalPath,

        [ValidateSet("Started", "Stopped")]
        [string]$State = "Started",

        [string]$ApplicationPool,

        [string[]]$BindingInfo,

        [string[]]$Protocol
    )

        $getTargetResourceResult = $null;

        <# Insert logic that uses the mandatory parameter values to get the website and assign it to a variable called $Website #>
        <# Set $ensureResult to "Present" if the requested website exists and to "Absent" otherwise #>

        # Add all Website properties to the hash table
        # This simple example assumes that $Website is not null
        $getTargetResourceResult = @{
                                      Name = $Website.Name; 
                                        Ensure = $ensureResult;
                                        PhysicalPath = $Website.physicalPath;
                                        State = $Website.state;
                                        ID = $Website.id;
                                        ApplicationPool = $Website.applicationPool;
                                        Protocol = $Website.bindings.Collection.protocol;
                                        Binding = $Website.bindings.Collection.bindingInformation;
                                    }

        $getTargetResourceResult;
}
```

В зависимости от значения, заданные для свойства ресурса в скрипт конфигурации **набора TargetResource** необходимо выполнить одно из следующих действий:

* Создание нового веб-сайта
* Обновление существующего веб-сайта
* Удаление существующего веб-сайта

Это показано в следующем примере.

```powershell
# The Set-TargetResource function is used to create, delete or configure a website on the target machine. 
function Set-TargetResource 
{
    [CmdletBinding(SupportsShouldProcess=$true)]
    param 
    (       
        [ValidateSet("Present", "Absent")]
        [string]$Ensure = "Present",

        [Parameter(Mandatory)]
        [ValidateNotNullOrEmpty()]
        [string]$Name,

        [Parameter(Mandatory)]
        [ValidateNotNullOrEmpty()]
        [string]$PhysicalPath,

        [ValidateSet("Started", "Stopped")]
        [string]$State = "Started",

        [string]$ApplicationPool,

        [string[]]$BindingInfo,

        [string[]]$Protocol
    )

    <# If Ensure is set to "Present" and the website specified in the mandatory input parameters does not exist, then create it using the specified parameter values #>
    <# Else, if Ensure is set to "Present" and the website does exist, then update its properties to match the values provided in the non-mandatory parameter values #>
    <# Else, if Ensure is set to "Absent" and the website does not exist, then do nothing #>
    <# Else, if Ensure is set to "Absent" and the website does exist, then delete the website #>
}
```

Наконец **TargetResource тест** функция должна принимать же задать в качестве параметра **Get TargetResource** и **набор TargetResource**. В реализации **Тест TargetResource**, проверьте состояние экземпляра ресурса, указанного в ключевые параметры. Если фактическое состояние экземпляра ресурса не совпадают значения, указанные в набор параметров, возвращает **$false**. В противном случае — возвращает **$true**.

В следующем коде реализуется **Тест TargetResource** функции.

```powershell
function Test-TargetResource
{
[CmdletBinding()]
[OutputType([System.Boolean])]
param
(
[ValidateSet("Present","Absent")]
[System.String]
$Ensure,

[parameter(Mandatory = $true)]
[System.String]
$Name,

[parameter(Mandatory = $true)]
[System.String]
$PhysicalPath,

[ValidateSet("Started","Stopped")]
[System.String]
$State,

[System.String[]]
$Protocol,

[System.String[]]
$BindingData,

[System.String]
$ApplicationPool
)

#Write-Verbose "Use this cmdlet to deliver information about command processing."

#Write-Debug "Use this cmdlet to write debug information while troubleshooting."


#Include logic to 
$result = [System.Boolean]
#Add logic to test whether the website is present and its status mathes the supplied parameter values. If it does, return true. If it does not, return false.
$result 
}
```

**Примечание**: для упрощения отладки используйте **Write-Verbose** командлет в реализации предыдущих трех функций. Этот командлет записывает текст в поток подробное сообщение. По умолчанию, поток подробное сообщение не отображается, но его можно отобразить, изменив значение **$VerbosePreference** переменных или с помощью **Подробно** параметр в командлетах DSC = новый.

###Создание манифеста модуля

Наконец, используйте **New-ModuleManifest** командлет, чтобы определить <ResourceName>psd1-файле для модуля настраиваемого ресурса. При вызове этого командлета, ссылки на файл модуля (.psm1) сценария, описанных в предыдущем разделе. Включить **Get TargetResource**, **TargetResource набор**, и **Тест TargetResource** в списке функций для экспорта. Ниже приведен пример файла манифеста.

```powershell
# Module manifest for module 'Demo.IIS.Website'
#
# Generated on: 1/10/2013
#

@{

# Script module or binary module file associated with this manifest.
# RootModule = ''

# Version number of this module.
ModuleVersion = '1.0'

# ID used to uniquely identify this module
GUID = '6AB5ED33-E923-41d8-A3A4-5ADDA2B301DE'

# Author of this module
Author = 'Contoso'

# Company or vendor of this module
CompanyName = 'Contoso'

# Copyright statement for this module
Copyright = 'Contoso. All rights reserved.'

# Description of the functionality provided by this module
Description = 'This Module is used to support the creation and configuration of IIS Websites through Get, Set and Test API on the DSC managed nodes.'

# Minimum version of the Windows PowerShell engine required by this module
PowerShellVersion = '4.0'

# Minimum version of the common language runtime (CLR) required by this module
CLRVersion = '4.0'

# Modules that must be imported into the global environment prior to importing this module
RequiredModules = @("WebAdministration")

# Modules to import as nested modules of the module specified in RootModule/ModuleToProcess
NestedModules = @("Demo_IISWebsite.psm1")

# Functions to export from this module
FunctionsToExport = @("Get-TargetResource", "Set-TargetResource", "Test-TargetResource")

# Cmdlets to export from this module
#CmdletsToExport = '*'

# HelpInfo URI of this module
# HelpInfoURI = ''
}
```



