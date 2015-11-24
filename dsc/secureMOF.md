#Защита MOF-файл

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

DSC указывает целевые узлы какие конфигурации, они должны иметь путем отправки MOF-файл с этой информацией для каждого узла, когда локальный диспетчер конфигурации (НОК) реализует требуемой конфигурации. Поскольку этот файл содержит сведения о конфигурации, очень важно обеспечить безопасность. Чтобы сделать это, можно задать локальный Configuration Manager для проверки учетных данных пользователя. В этом разделе описывается передача эти учетные данные безопасного подключения к целевой узел путем их шифрования с помощью сертификатов.

##Предварительные требования

Для успешного шифрования учетные данные, используемые для защиты конфигурации DSC, убедитесь, что у вас есть следующее:

* **Средства выдачи и распространение сертификатов**. В этом разделе и примерах предполагается, что вы используете центр сертификации Active Directory. Дополнительные сведения о службах сертификатов Active Directory см. в разделе [Обзор служб сертификатов Active Directory](https://technet.microsoft.com/library/hh831740.aspx) и [службы сертификатов Active Directory в Windows Server 2008](https://technet.microsoft.com/windowsserver/dd448615.aspx).
* **Администрирования доступ к целевой узел или узлы**.
* **Каждый целевой узел с поддержкой шифрования сертификата, сохранить его в личное хранилище**. В Windows PowerShell путь к хранилищу — Cert: \LocalMachine\My. В примерах в этом разделе используется шаблона «проверка подлинности рабочей станции» (а также другие шаблоны сертификатов) можно найти в [по умолчанию шаблоны сертификатов] (https://technet.microsoft.com/library/cc740061 (v=WS.10).aspx).
* Если будет выполняться этой конфигурации на компьютере, отличном от целевого узла **экспортировать открытый ключ сертификата**, а затем импортировать его на компьютер, который будет выполняться конфигурацию из. Убедитесь в том, что можно экспортировать только **открытый** ключа; защиту закрытого ключа.

##Общий процесс

1. Настройка сертификатов, ключей и отпечатки, гарантируя, что каждый целевой узел копии сертификата и конфигурации компьютера содержит открытый ключ и отпечаток.
1. Создайте блок данных конфигурации, содержащий отпечаток открытого ключа и путь к файлу.
1. Создайте скрипт конфигурации, определяющий требуемой конфигурации для целевой узел и устанавливает расшифровки на целевые узлы с помощью команды локальной конфигурации диспетчер расшифровать данные конфигурации, с помощью сертификата и его отпечаток.
1. Выполните конфигурацию, которая будет задать параметры конфигурации диспетчера локальных и запуск DSC конфигурации.

##Данные конфигурации

Блок данных конфигурации определяет, какие целевые узлы для работы, следует ли или не шифровать учетные данные, средства шифрования и другие сведения. Дополнительные сведения о блоке данных конфигурации в разделе [отделения конфигурации и данных среды](configData.md).

В этом примере показан блок данных конфигурации, указывающее целевой узел для работы на именованный targetNode, путь файла сертификата открытого ключа (с именем targetNode.cer) и отпечаток для открытого ключа.

```powershell
$ConfigData= @{ 
    AllNodes = @(     
            @{  
                # The name of the node we are describing 
                NodeName = "targetNode" 

                # The path to the .cer file containing the 
                # public key of the Encryption Certificate 
                # used to encrypt credentials for this node 
                CertificateFile = "C:\publicKeys\targetNode.cer" 


                # The thumbprint of the Encryption Certificate 
                # used to decrypt the credentials on target node 
                Thumbprint = "AC23EA3A9E291A75757A556D0B71CBBF8C4F6FD8" 
            }; 
        );    
    }
```

##Скрипт конфигурации

В самом скрипте конфигурации используйте `PsCredential` параметр, чтобы убедиться, что учетные данные хранятся для кратчайшие сроки. При запуске примера предоставленного DSC запрашивать учетные данные и затем зашифровать MOF-файл с помощью CertificateFile, связанный с целевой узел в блоке данных конфигурации. Данный пример кода копирует файл из общей папки, защищенного с пользователем.

```
configuration CredentialEncryptionExample 
{ 
    param( 
        [Parameter(Mandatory=$true)] 
        [ValidateNotNullorEmpty()] 
        [PsCredential] $credential 
        ) 


    Node $AllNodes.NodeName 
    { 
        File exampleFile 
        { 
            SourcePath = "\\Server\share\path\file.ext" 
            DestinationPath = "C:\destinationPath" 
            Credential = $credential 
        } 
    } 
}
```

##Настройка расшифровки

Прежде чем `начала DscConfiguration` можно работать, необходимо указать локальный диспетчер конфигурации на каждом узле целевой сертификат для расшифровки учетных данных, использования ресурсов Ид_сертификата проверки отпечаток сертификата. Этот пример функция найдет соответствующий локальный сертификат (может потребоваться настроить его, чтобы найти точное сертификат, который вы хотите использовать):

```powershell
# Get the certificate that works for encryption 
function Get-LocalEncryptionCertificateThumbprint 
{ 
    (dir Cert:\LocalMachine\my) | %{
        # Verify the certificate is for Encryption and valid 
        if ($_.PrivateKey.KeyExchangeAlgorithm -and $_.Verify()) 
        { 
            return $_.Thumbprint 
        } 
    } 
}
```

С определяется его отпечаток сертификата можно обновить скрипт конфигурации для использования значения:

```powershell
configuration CredentialEncryptionExample 
{ 
    param( 
        [Parameter(Mandatory=$true)] 
        [ValidateNotNullorEmpty()] 
        [PsCredential] $credential 
        ) 


    Node $AllNodes.NodeName 
    { 
        File exampleFile 
        { 
            SourcePath = "\\Server\share\path\file.ext" 
            DestinationPath = "C:\destinationPath" 
            Credential = $credential 
        } 

        LocalConfigurationManager 
        { 
             CertificateId = $node.Thumbprint 
        } 
    } 
}
```

##Запуск конфигурации

На этом этапе можно запустить конфигурацию, которая будет выводить два файла:

* A *. meta.mof файл, предназначенный для настройки локального Configuration Manager для расшифровки учетные данные с помощью сертификата, которые хранятся в хранилище локального компьютера и определяется его отпечаток. Применяет набор DscLocalConfigurationManager *. meta.mof файл.
* MOF-файл, который фактически применяется конфигурация. Начало DscConfiguration применяется конфигурация.

Эти команды будут выполнены эти шаги:

```powershell
Write-Host "Generate DSC Configuration..."
CredentialEncryptionExample -ConfigurationData $ConfigData -OutputPath .\CredentialEncryptionExample

Write-Host "Setting up LCM to decrypt credentials..."
Set-DscLocalConfigurationManager .\CredentialEncryptionExample -Verbose 

Write-Host "Starting Configuration..."
Start-DscConfiguration .\CredentialEncryptionExample -wait -Verbose
```

Ниже приведен полный пример, включающий все эти действия, а также командлет вспомогательный метод, который экспортирует и копирует открытых ключей:

```powershell
# A simple example of using credentials
configuration CredentialEncryptionExample
{
    param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [PsCredential] $credential
        )


    Node $AllNodes.NodeName
    {
        File exampleFile
        {
            SourcePath = "\\server\share\file.txt"
            DestinationPath = "C:\Users\user"
            Credential = $credential
        }

        LocalConfigurationManager
        {
            CertificateId = $node.Thumbprint
        }
    }
}

# A Helper to invoke the configuration, with the correct public key 
# To encrypt the configuration credentials
function Start-CredentialEncryptionExample
{
    [CmdletBinding()]
    param ($computerName)


    [string] $thumbprint = Get-EncryptionCertificate -computerName $computerName -Verbose
    Write-Verbose "using cert: $thumbprint"

    $certificatePath = join-path -Path "$env:SystemDrive\$script:publicKeyFolder" -childPath "$computername.EncryptionCertificate.cer"         

    $ConfigData=    @{
        AllNodes = @(     
                        @{  
                            # The name of the node we are describing
                            NodeName = "$computerName"

                            # The path to the .cer file containing the
                            # public key of the Encryption Certificate
                            CertificateFile = "$certificatePath"

                            # The thumbprint of the Encryption Certificate
                            # used to decrypt the credentials
                            Thumbprint = $thumbprint
                        };
                    );    
    }

    Write-Verbose "Generate DSC Configuration..."
    CredentialEncryptionExample -ConfigurationData $ConfigData -OutputPath .\CredentialEncryptionExample `
        -credential (Get-Credential -UserName "$env:USERDOMAIN\$env:USERNAME" -Message "Enter credentials for configuration") 

    Write-Verbose "Setting up LCM to decrypt credentials..."
    Set-DscLocalConfigurationManager .\CredentialEncryptionExample -Verbose 

    Write-Verbose "Starting Configuration..."
    Start-DscConfiguration .\CredentialEncryptionExample -wait -Verbose

}


#region HelperFunctions

# The folder name for the exported public keys
$script:publicKeyFolder = "publicKeys"

# Get the certificate that works for encryptions
function Get-EncryptionCertificate
{
    [CmdletBinding()]
    param ($computerName)
    $returnValue= Invoke-Command -ComputerName $computerName -ScriptBlock {
            $certificates = dir Cert:\LocalMachine\my

            $certificates | %{
                    # Verify the certificate is for Encryption and valid
                    if ($_.PrivateKey.KeyExchangeAlgorithm -and $_.Verify())
                    {
                        # Create the folder to hold the exported public key
                        $folder= Join-Path -Path $env:SystemDrive\ -ChildPath $using:publicKeyFolder
                        if (! (Test-Path $folder))
                        {
                            md $folder | Out-Null
                        }

                        # Export the public key to a well known location
                        $certPath = Export-Certificate -Cert $_ -FilePath (Join-Path -path $folder -childPath "EncryptionCertificate.cer") 

                        # Return the thumbprint, and exported certificate path
                        return @($_.Thumbprint,$certPath);
                    }
                  }
        }
    Write-Verbose "Identified and exported cert..."
    # Copy the exported certificate locally
    $destinationPath = join-path -Path "$env:SystemDrive\$script:publicKeyFolder" -childPath "$computername.EncryptionCertificate.cer"
    Copy-Item -Path (join-path -path \\$computername -childPath $returnValue[1].FullName.Replace(":","$"))  $destinationPath | Out-Null

    # Return the thumbprint
    return $returnValue[0]
}
```



