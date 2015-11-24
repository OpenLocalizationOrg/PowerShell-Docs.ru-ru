#Написание настраиваемых ресурсов DSC с классами PowerShell

> Область применения: Windows Windows PowerShell 5.0

С появлением классов PowerShell в Windows PowerShell 5.0 теперь можно определить путем создания класса DSC ресурса. Класс определяет схему и реализацию ресурсов, поэтому нет необходимости создавать отдельный файл MOF. Структура папок для ресурсов на основе класса также проще, поскольку **DSCResources** папку не требуется.

В DSC ресурс на основе класса схемы определяются как свойства класса, из которого можно изменить с помощью атрибутов для указания типа свойства... Ресурс реализуется **Get()**, **Set()**, и **Test()** методы (эквивалентно **Get TargetResource**, **TargetResource набор**, и **TargetResource тест** функции в ресурс скрипта.

В этом разделе мы создадим простой ресурс с именем **FileResource** управляющий файл по указанному пути.

Дополнительные сведения о ресурсах DSC см. в разделе [построения пользовательского Windows PowerShell требуемой состояние конфигурации ресурсов](authoringResource.md)

##Структура папок для ресурса класса

Чтобы реализовать настраиваемый ресурс DSC с классом PowerShell, создайте следующую структуру папок. Класс определяется в **MyDscResource.psm1** и манифеста модуля определяется в **MyDscResource.psd1**.

```
$env: psmodulepath (folder)
    |- MyDscResources (folder)
        |- MyDscResource.psm1 
           MyDscResource.psd1 
```

##Создание класса

Создание класса PowerShell, используйте ключевое слово class. Чтобы указать, что класс ресурсов DSC, используйте **DscResource()** атрибута. Имя класса является именем ресурса DSC.

```powershell
[DscResource()]
class FileResource{
}
```

###Объявление свойств

Схема ресурсов DSC определяется как свойства класса. Мы объявить три свойства следующим образом.

```powershell
[DscProperty(Key)]
[string]$Path


[DscProperty(Mandatory)]
[Ensure] $Ensure


[DscProperty(Mandatory)]
[string] $SourcePath

[DscProperty(NotConfigurable)]
[Nullable[datetime]] $CreationTime
```

Обратите внимание, что свойства изменяются атрибутами. Карта attribues на эквивалентные атрибуты, используемые в классах MOF следующим образом.

Свойство атрибута в атрибуте класса MOF описание
DscProperty(Key)
Ключ
Свойство является обязательным. Свойство является ключом. Значения всех свойств помечены как ключи необходимо объединить для уникальной идентификации экземпляра ресурса в конфигурации.
DscProperty(Mandatory)
Требуется
Свойство является обязательным.
DscProperty(NotConfigurable)
чтение
Свойство доступно только для чтения. Свойства, помеченные этим атрибутом не задаются в конфигурации, но заполняются методом Get(), если он присутствует.
DscProperty()
Записи
Свойство является настраиваемым, но он не является обязательным.

**$Path** и **$SourcePath** свойства являются обе строки.  **$CreationTime** — [DateTime](https://technet.microsoft.com/en-us/library/system.datetime.aspx) свойства.  **$Ensure** свойство является перечислением, определяются следующим образом.

```powershell
enum Ensure 
{ 
    Absent 
    Present 
}
```

###Реализация методов

**Get()**, **Set()**, и **Test()** методы аналогичны **Get TargetResource**, **набора TargetResource**, и **TargetResource тест** функции в ресурс скрипта.

Этот код также включает функцию CopyFile() вспомогательную функцию, которая копирует файл из **$SourcePath** для **$Path**.

```powershell
<#
        This method is equivalent of the Set-TargetResource script function.
        It sets the resource to the desired state.
    #>
    [void] Set()
    {        
        $fileExists = $this.TestFilePath($this.Path)
        if($this.ensure -eq [Ensure]::Present)
        {
            if(-not $fileExists)
            {
                $this.CopyFile()
            }
        }
        else
        {
            if($fileExists)
            {
                Write-Verbose -Message "Deleting the file $($this.Path)"
                Remove-Item -LiteralPath $this.Path -Force
            }
        }
    }

    <# 

        This method is equivalent of the Test-TargetResource script function. 
        It should return True or False, showing whether the resource
        is in a desired state.         
    #>
    [bool] Test()
    {
        $present = $this.TestFilePath($this.Path)

        if($this.Ensure -eq [Ensure]::Present)
        {
            return $present
        }
        else
        {
            return -not $present
        }
    }


    <#
        This method is equivalent of the Get-TargetResource script function.
        The implementation should use the keys to find appropriate resources.
        This method returns an instance of this class with the updated key
         properties.
    #>
    [FileResource] Get()
    {
        $present = $this.TestFilePath($this.Path)        

        if ($present)
        {
            $file = Get-ChildItem -LiteralPath $this.Path
            $this.CreationTime = $file.CreationTime
            $this.Ensure = [Ensure]::Present
        }
        else
        {
            $this.CreationTime = $null
            $this.Ensure = [Ensure]::Absent
        }        

        return $this
    }

    <#
        Helper method to check if the file exists and it is file
    #>
    [bool] TestFilePath([string] $location)
    {      
        $present = $true

        $item = Get-ChildItem -LiteralPath $location -ea Ignore
        if ($item -eq $null)
        {
            $present = $false            
        }
        elseif( $item.PSProvider.Name -ne "FileSystem")
        {
            throw "Path $($location) is not a file path."
        }
        elseif($item.PSIsContainer)
        {
            throw "Path $($location) is a directory path."
        }
        return $present
    }

    <#
        Helper method to copy file from source to path
    #>
    [void] CopyFile()
    { 
        if(-not $this.TestFilePath($this.SourcePath))
        {
            throw "SourcePath $($this.SourcePath) is not found."
        }

        [System.IO.FileInfo] $destFileInfo = new-object System.IO.FileInfo($this.Path)
        if (-not $destFileInfo.Directory.Exists)
        {
            Write-Verbose -Message "Creating directory $($destFileInfo.Directory.FullName)"

            #use CreateDirectory instead of New-Item to avoid code
            # to handle the non-terminating error
            [System.IO.Directory]::CreateDirectory($destFileInfo.Directory.FullName)
        }

        if(Test-Path -LiteralPath $this.Path -PathType Container)
        {
            throw "Path $($this.Path) is a directory path"
        }

        Write-Verbose -Message "Copying $($this.SourcePath) to $($this.Path)"

        #DSC engine catches and reports any error that occurs
        Copy-Item -LiteralPath $this.SourcePath -Destination $this.Path -Force
    }
}<# 
        This method replaces the Set-TargetResource DSC script function.
        It sets the resource to the desired state. 
    #>
    [void] Set() 
    {       
        $fileExists = Test-Path -path $this.Path -PathType Leaf 
        if($this.ensure -eq [Ensure]::Present)
        {
            if(-not $fileExists)
            {
                $this.CopyFile()            
            }
        }
        else
        {
            if($fileExists)         
            {
                Write-Verbose -Message "Deleting the file $this.Path"
                Remove-Item -LiteralPath $this.Path                     
            }
        } 
    } 


    <# 

        This method replaces the Test-TargetResource function. 
        It should return True or False, showing whether the resource
         is in a desired state.         
    #> 

    [bool] Test()
    {
        if(Test-Path -path $this.Path -PathType Container)
        {
            throw "Path '$this.Path' is a directory path."
        }

        $fileExists = Test-Path -path $this.Path -PathType Leaf

        if($this.ensure -eq [Ensure]::Present)
        {
            return $fileExists
        }

        return (-not $fileExists)
    }


    <# 
        This method replaces the Get-TargetResource function.         
        The implementation should use the keys to find appropriate resources. 
        This method returns an instance of this class with the updated key properties. 
    #> 

    [FileResource] Get() 
    {
        $file = Get-item $this.Path
        return $this
    }

    <#
        Helper method to copy file from source to path.
        Because this resource provider run under system,
        Only the Administrators and system have full
         access to the new created directory and file
    #>
    CopyFile()
    {
        if(Test-Path -path $this.SourcePath -PathType Container)
        {
            throw "SourcePath '$this.SourcePath' is a directory path"
        }

        if( -not (Test-Path -path $this.SourcePath -PathType Leaf))
        {
            throw "SourcePath '$this.SourcePath' is not found."
        }

        [System.IO.FileInfo] $destFileInfo = new-object System.IO.FileInfo($this.Path)
        if (-not $destFileInfo.Directory.Exists)
        {         
            Write-Verbose -Message "Creating directory $($destFileInfo.Directory.FullName)"

            #use CreateDirectory instead of New-Item to avoid lines
            # to handle the non-terminating error
            [System.IO.Directory]::CreateDirectory($destFileInfo.Directory.FullName)
        }

        if(Test-Path -path $this.Path -PathType Container)
        {
            throw "Path '$this.Path' is a directory path"
        }

        Write-Verbose -Message "Copying $this.SourcePath to $this.Path"

        #DSC engine catches and reports any error that occurs
        Copy-Item -Path $this.SourcePath -Destination $this.Path -Force
    } 
 <# 
        This method replaces the Set-TargetResource DSC script function.
        It sets the resource to the desired state. 
    #>
    [void] Set() 
    {       
        $fileExists = Test-Path -path $Path -PathType Leaf 
        if($ensure -eq [Ensure]::Present)
        {
            if(-not $fileExists)
            {
                $this.CopyFile()            
            }
        }
        else
        {
            if($fileExists)         
            {
                Write-Verbose -Message "Deleting the file $Path"
                Remove-Item -LiteralPath $Path                     
            }
        } 
    } 


    <# 

        This method replaces the Test-TargetResource function. 
        It should return True or False, showing whether the resource
         is in a desired state.         
    #> 

    [bool] Test()
    {
        if(Test-Path -path $Path -PathType Container)
        {
            throw "Path '$Path' is a directory path."
        }

        $fileExists = Test-Path -path $Path -PathType Leaf
if($ensure -eq [Ensure]::Present)
        {
            return $fileExists
        }

        return (-not $fileExists)
    }


    <# 
        This method replaces the Get-TargetResource function.         
        The implementation should use the keys to find appropriate resources. 
        This method returns an instance of this class with the updated key properties. 
    #> 

    [FileResource] Get() 
    {
        $file = Get-item $Path
        return $this
    }

    <#
        Helper method to copy file from source to path
        Because this resource provider run under system,
        Only the Administrators and system have full
         access to the new created directory and file
    #>
    CopyFile()
    {
        if(Test-Path -path $SourcePath -PathType Container)
        {
            throw "SourcePath '$SourcePath' is a directory path"
        }

        if( -not (Test-Path -path $SourcePath -PathType Leaf))
        {
            throw "SourcePath '$SourcePath' is not found."
        }

        [System.IO.FileInfo] $destFileInfo = new-object System.IO.FileInfo($Path)
        if (-not $destFileInfo.Directory.Exists)
        {         
            Write-Verbose -Message "Creating directory $($destFileInfo.Directory.FullName)"

            #use CreateDirectory instead of New-Item to avoid lines
            # to handle the non-terminating error
            [System.IO.Directory]::CreateDirectory($destFileInfo.Directory.FullName)
        }

        if(Test-Path -path $Path -PathType Container)
        {
            throw "Path '$Path' is a directory path"
        }

        Write-Verbose -Message "Copying $SourcePath to $Path"

        #DSC engine catches and reports any error that occurs
        Copy-Item -Path $SourcePath -Destination $Path -Force
    }
```

###Полный файл

Следует файла класса.

```powershell
enum Ensure
{
    Absent
    Present
}

<#
   This resource manages the file in a specific path.
   [DscResource()] indicates the class is a DSC resource
#>

[DscResource()]
class FileResource
{
    <# 
       This property is the fully qualified path to the file that is
       expected to be present or absent.

       The [DscProperty(Key)] attribute indicates the property is a
       key and its value uniquely identifies a resource instance.
       Defining this attribute also means the property is required
       and DSC will ensure a value is set before calling the resource.

       A DSC resource must define at least one key property.
    #>
    [DscProperty(Key)]
    [string]$Path

    <#
        This property indicates if the settings should be present or absent
        on the system. For present, the resource ensures the file pointed
        to by $Path exists. For absent, it ensures the file point to by
        $Path does not exist.

        The [DscProperty(Mandatory)] attribute indicates the property is
        required and DSC will guarantee it is set.

        If Mandatory is not specified or if it is defined as 
        Mandatory=$false, the value is not guaranteed to be set when DSC
        calls the resource.  This is appropriate for optional properties.
    #>
    [DscProperty(Mandatory)]
    [Ensure] $Ensure    

    <#
       This property defines the fully qualified path to a file that will
       be placed on the system if $Ensure = Present and $Path does not
        exist.

       NOTE: This property is required because [DscProperty(Mandatory)] is
        set.
    #>
    [DscProperty(Mandatory)]
    [string] $SourcePath

    <#
       This property reports the file's create timestamp.

       [DscProperty(NotConfigurable)] attribute indicates the property is
       not configurable in DSC configuration.  Properties marked this way
       are populated by the Get() method to report additional details
       about the resource when it is present.

    #>
    [DscProperty(NotConfigurable)]
    [Nullable[datetime]] $CreationTime

    <#
        This method is equivalent of the Set-TargetResource script function.
        It sets the resource to the desired state.
    #>
    [void] Set()
    {        
        $fileExists = $this.TestFilePath($this.Path)
        if($this.ensure -eq [Ensure]::Present)
        {
            if(-not $fileExists)
            {
                $this.CopyFile()
            }
        }
        else
        {
            if($fileExists)
            {
                Write-Verbose -Message "Deleting the file $($this.Path)"
                Remove-Item -LiteralPath $this.Path -Force
            }
        }
    }

    <# 

        This method is equivalent of the Test-TargetResource script function. 
        It should return True or False, showing whether the resource
        is in a desired state.         
    #>
    [bool] Test()
    {
        $present = $this.TestFilePath($this.Path)

        if($this.Ensure -eq [Ensure]::Present)
        {
            return $present
        }
        else
        {
            return -not $present
        }
    }


    <#
        This method is equivalent of the Get-TargetResource script function.
        The implementation should use the keys to find appropriate resources.
        This method returns an instance of this class with the updated key
         properties.
    #>
    [FileResource] Get()
    {
        $present = $this.TestFilePath($this.Path)        

        if ($present)
        {
            $file = Get-ChildItem -LiteralPath $this.Path
            $this.CreationTime = $file.CreationTime
            $this.Ensure = [Ensure]::Present
        }
        else
        {
            $this.CreationTime = $null
            $this.Ensure = [Ensure]::Absent
        }        

        return $this
    }

    <#
        Helper method to check if the file exists and it is file
    #>
    [bool] TestFilePath([string] $location)
    {      
        $present = $true

        $item = Get-ChildItem -LiteralPath $location -ea Ignore
        if ($item -eq $null)
        {
            $present = $false            
        }
        elseif( $item.PSProvider.Name -ne "FileSystem")
        {
            throw "Path $($location) is not a file path."
        }
        elseif($item.PSIsContainer)
        {
            throw "Path $($location) is a directory path."
        }
        return $present
    }

    <#
        Helper method to copy file from source to path
    #>
    [void] CopyFile()
    { 
        if(-not $this.TestFilePath($this.SourcePath))
        {
            throw "SourcePath $($this.SourcePath) is not found."
        }

        [System.IO.FileInfo] $destFileInfo = new-object System.IO.FileInfo($this.Path)
        if (-not $destFileInfo.Directory.Exists)
        {
            Write-Verbose -Message "Creating directory $($destFileInfo.Directory.FullName)"

            #use CreateDirectory instead of New-Item to avoid code
            # to handle the non-terminating error
            [System.IO.Directory]::CreateDirectory($destFileInfo.Directory.FullName)
        }

        if(Test-Path -LiteralPath $this.Path -PathType Container)
        {
            throw "Path $($this.Path) is a directory path"
        }

        Write-Verbose -Message "Copying $($this.SourcePath) to $($this.Path)"

        #DSC engine catches and reports any error that occurs
        Copy-Item -LiteralPath $this.SourcePath -Destination $this.Path -Force
    }
}# This module defines a class for a DSC "FileResource" provider. 


enum Ensure 
{ 
    Absent 
    Present 
} 


<# This resource manages the file in a specific path. 
[DscResource()] indicates the class is a DSC resource 
#> 

[DscResource()]
class FileResource{ 

    <# This is a key property 
       [DscResourceKey()] also means the property is required.
       It is guaranteed to be set, other properties may not
        be set if the configuration did not specify values.
    #> 
    [DscResourceKey()]
    [string]$Path

    <# 
       [DscResourceMandatory()] means the property is required.
       It is guaranteed to be set, other properties may not be set
        if the configuration did not specify values.
    #>
    [DscResourceMandatory()]
    [Ensure] $Ensure

    <# 
       [DscResourceMandatory()] means the property is required.
    #>
    [DscResourceMandatory()]
    [string] $SourcePath     

    [DscResource

    <# 
        This method replaces the Set-TargetResource DSC script function.
        It sets the resource to the desired state. 
    #>
    [void] Set() 
    {       
        $fileExists = Test-Path -path $this.Path -PathType Leaf 
        if($this.ensure -eq [Ensure]::Present)
        {
            if(-not $fileExists)
            {
                $this.CopyFile()            
            }
        }
        else
        {
            if($fileExists)         
            {
                Write-Verbose -Message "Deleting the file $this.Path"
                Remove-Item -LiteralPath $this.Path                     
            }
        } 
    } 


    <# 

        This method replaces the Test-TargetResource function. 
        It should return True or False, showing whether the resource
         is in a desired state.         
    #> 

    [bool] Test()
    {
        if(Test-Path -path $this.Path -PathType Container)
        {
            throw "Path '$this.Path' is a directory path."
        }

        $fileExists = Test-Path -path $this.Path -PathType Leaf

        if($this.ensure -eq [Ensure]::Present)
        {
            return $fileExists
        }

        return (-not $fileExists)
    }


    <# 
        This method replaces the Get-TargetResource function.         
        The implementation should use the keys to find appropriate resources. 
        This method returns an instance of this class with the updated key properties. 
    #> 

    [FileResource] Get() 
    {
        $file = Get-item $this.Path
        return $this
    }

    <#
        Helper method to copy file from source to path.
        Because this resource provider run under system,
        Only the Administrators and system have full
         access to the new created directory and file
    #>
    CopyFile()
    {
        if(Test-Path -path $this.SourcePath -PathType Container)
        {
            throw "SourcePath '$this.SourcePath' is a directory path"
        }

        if( -not (Test-Path -path $this.SourcePath -PathType Leaf))
        {
            throw "SourcePath '$this.SourcePath' is not found."
        }

        [System.IO.FileInfo] $destFileInfo = new-object System.IO.FileInfo($this.Path)
        if (-not $destFileInfo.Directory.Exists)
        {         
            Write-Verbose -Message "Creating directory $($destFileInfo.Directory.FullName)"

            #use CreateDirectory instead of New-Item to avoid lines
            # to handle the non-terminating error
            [System.IO.Directory]::CreateDirectory($destFileInfo.Directory.FullName)
        }

        if(Test-Path -path $this.Path -PathType Container)
        {
            throw "Path '$this.Path' is a directory path"
        }

        Write-Verbose -Message "Copying $this.SourcePath to $this.Path"

        #DSC engine catches and reports any error that occurs
        Copy-Item -Path $this.SourcePath -Destination $this.Path -Force
    }
} 
```

##Создание манифеста

Чтобы сделать доступными для обработчика DSC ресурсов на основе класса, необходимо включить **DscResourcesToExport** инструкции в файле манифеста, указывает, что модуль для экспорта ресурса. Наш манифест выглядит следующим образом:

```powershell
@{

# Script module or binary module file associated with this manifest.
RootModule = 'MyDscResource.psm1'

DscResourcesToExport = 'FileResource'

# Version number of this module.
ModuleVersion = '1.0'

# ID used to uniquely identify this module
GUID = '81624038-5e71-40f8-8905-b1a87afe22d7'

# Author of this module
Author = 'Microsoft Corporation'

# Company or vendor of this module
CompanyName = 'Microsoft Corporation'

# Copyright statement for this module
Copyright = '(c) 2014 Microsoft. All rights reserved.'

# Description of the functionality provided by this module
# Description = ''

# Minimum version of the Windows PowerShell engine required by this module
PowerShellVersion = '5.0'

# Name of the Windows PowerShell host required by this module
# PowerShellHostName = ''
} 
```

##Тестирование ресурса

После сохранения класс и файлы манифеста в структуре папок, как описано выше, можно создать конфигурацию, которая использует новый ресурс. Сведения о способах выполнения конфигурации DSC см. [Введение в действие конфигурации](enactingConfigurations.md). Следующая конфигурация будет проверять ли файл в `c:\test\test.txt` существует и, если это не так, копирует файл из `c:\test.txt` (следует создать `c:\test.txt` перед запуском конфигурации).

```powershell
Configuration Test
{
    Import-DSCResource -module MyDscResource
    FileResource file
    {
        Path = "C:\test\test.txt"
        SourcePath = "c:\test.txt"
        Ensure = "Present"
    } 
}
Test
Start-DscConfiguration -Wait -Force Test
```

##См. также

###Основные понятия

[Построение пользовательского Windows PowerShell требуемой состояния конфигурации ресурсов](authoringResource.md)



