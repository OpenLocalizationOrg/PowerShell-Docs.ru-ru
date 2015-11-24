#Составной ресурсы: использование конфигурации DSC в качестве ресурса

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

В реальных ситуациях конфигураций может стать длинные и сложные, вызвав много различных ресурсов и задание много различных свойств. Для ликвидации сложность настройки конфигурации состояния требуемой Windows PowerShell (DSC) можно использовать как ресурс для других конфигураций. Мы называем это составное ресурсов. Составной ресурс — DSC конфигурации, который принимает параметры. Параметры конфигурации в качестве свойства ресурса. Конфигурация сохраняется как файл с **. расширение Schema.psm1** и принимает месте схемы MOF и ресурса скрипта в типичных ресурсов DSC (Дополнительные сведения о ресурсах DSC см. в разделе [ресурсов Windows PowerShell требуемой состояние конфигурации](resources.md).

##Создание составного ресурса

В нашем примере мы создать конфигурацию, которая вызывает ряд существующие ресурсы для настройки виртуальных машин. Вместо указания значения, задаваемые в блоках конфигурация, конфигурация принимает ряд параметров, которые затем используются в блоках конфигурации.

```powershell
Configuration xVirtualMachine
{
param
(
# Name of VMs
[Parameter(Mandatory)]
[ValidateNotNullOrEmpty()]
[String[]]$VMName,

# Name of Switch to create
[Parameter(Mandatory)]
[ValidateNotNullOrEmpty()]
[String]$SwitchName,

# Type of Switch to create
[Parameter(Mandatory)]
[ValidateNotNullOrEmpty()]
[String]$SwitchType,

# Source Path for VHD
[Parameter(Mandatory)]
[ValidateNotNullOrEmpty()]
[String]$VhdParentPath,

# Destination path for diff VHD
[Parameter(Mandatory)]
[ValidateNotNullOrEmpty()]
[String]$VHDPath,

# Startup Memory for VM
[Parameter(Mandatory)]
[ValidateNotNullOrEmpty()]
[String]$VMStartupMemory,

# State of the VM
[Parameter(Mandatory)]
[ValidateNotNullOrEmpty()]
[String]$VMState
)

# Import the module that defines custom resources
Import-DscResource -Module xComputerManagement,xHyper-V

# Install the HyperV role 
WindowsFeature HyperV
{
    Ensure = "Present"
    Name = "Hyper-V" 
}

# Create the virtual switch 
xVMSwitch $switchName
{
    Ensure = "Present"
    Name = $switchName
    Type = $SwitchType
    DependsOn = "[WindowsFeature]HyperV"
}

# Check for Parent VHD file
File ParentVHDFile
{
    Ensure = "Present"
    DestinationPath = $VhdParentPath
    Type = "File"
    DependsOn = "[WindowsFeature]HyperV"
}

# Check the destination VHD folder
File VHDFolder
{
    Ensure = "Present"
    DestinationPath = $VHDPath
    Type = "Directory"
    DependsOn = "[File]ParentVHDFile"
}

 # Creae VM specific diff VHD
foreach($Name in $VMName)
{
    xVHD "VhD$Name"
    {
        Ensure = "Present"
        Name = $Name
        Path = $VhDPath
        ParentPath = $VhdParentPath
        DependsOn = @("[WindowsFeature]HyperV",
                      "[File]VHDFolder")
    }
}

# Create VM using the above VHD
foreach($Name in $VMName)
{
    xVMHyperV "VMachine$Name"
    {
        Ensure = "Present"
        Name = $Name
        VhDPath = (Join-Path -Path $VhDPath -ChildPath $Name)
        SwitchName = $SwitchName
        StartupMemory = $VMStartupMemory
        State = $VMState
        MACAddress = $MACAddress 
        WaitForIP = $true
        DependsOn = @("[WindowsFeature]HyperV",
                      "[xVHD]Vhd$Name")
    }
} 
}
```

###Сохранение конфигурации в качестве составной ресурса

Для использования параметризованных конфигурации как ресурс DSC, сохраните его в структуру каталогов, как и других ресурсов на базе MOF и назовите его с **. schema.psm1** расширения. В этом примере мы имя файла **xVirtualMachine.schema.psm1**. Также необходимо создать манифест с именем **xVirtualMachine.psd1** содержащий следующую строку. Обратите внимание, что это в дополнение к **MyDscResources.psd1**, манифеста модуля для всех ресурсов в группе **MyDscResources** папки.

```powershell
RootModule = 'xVirtualMachine.schema.psm1'
```

Когда закончите, структуру папки должен иметь следующий вид.

```
$env: psmodulepath
    |- MyDscResources 
           MyDscResources.psd1
        |- DSC Resources 
            |- xVirtualMachine
                |- xVirtualMachine.psd1 
                |- xVirtualMachine.schema.psm1
```

Ресурс теперь можно обнаружить с помощью командлета Get-DscResource и его свойства доступны для обнаружения, либо командлет, или используя **Ctrl + пробел** автозавершения в Windows PowerShell ISE.

##С помощью составного ресурсов

Далее мы создать конфигурацию, которая вызывает составного ресурсов. Эта конфигурация вызывает ресурсов xVirtualMachine для создания виртуальной машины, а затем вызывает **xComputer** ресурсов, чтобы переименовать его.

```powershell
configuration RenameVM
{
Import-DscResource -Module TestCompositeResource

Node localhost
{
    xVirtualMachine VM
    {
        VMName = "Test"
        SwitchName = "Internal"
        SwitchType = "Internal"
        VhdParentPath = "C:\Demo\Vhd\RTM.vhd"
        VHDPath = "C:\Demo\Vhd"
        VMStartupMemory = 1024MB
        VMState = "Running"
    }
    }
   Node "192.168.10.1"
   {   
    xComputer Name
    {
        Name = "SQL01"
        DomainName = "fourthcoffee.com" 
    }                                                                                                                                                                                                                                                               
}
} 
```

##См. также

###Основные понятия

* [Написание настраиваемых DSC ресурсов с помощью MOF](authoringResourceMOF.md)
* [Приступая к работе с Windows PowerShell нужное состояние конфигурации](overview.md)



