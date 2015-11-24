#Создание ресурса DSC в C

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

Как правило настраиваемый ресурс конфигурации состояния требуемой Windows PowerShell (DSC) реализуется в скрипте PowerShell. Тем не менее также можно реализовать настраиваемый ресурс DSC функциональные возможности, добавив командлеты в C#. Вводные сведения о создании командлетов в C# см. в разделе [записи командлета Windows PowerShell](https://technet.microsoft.com/en-us/library/dd878294.aspx).

Помимо реализации ресурса в C# как командлеты, процесс создания схемы MOF, создание структуры папок, импорта и использования настраиваемый ресурс DSC такие же, как описано в [написания пользовательских DSC ресурсов с помощью MOF](authoringResourceMOF.md).

##Записи ресурсов на основе командлетов

В этом примере мы будем реализовывать простой ресурс, который управляет текстового файла и его содержимое.

###Записи схемы MOF

Ниже приводится определение ресурсов MOF.

```
[ClassVersion("1.0.0"), FriendlyName("xDemoFile")]
class MSFT_XDemoFile : OMI_BaseResource
{
                [Key, Description("path")] String Path;
                [Write, Description("Should the file be present"), ValueMap{"Present","Absent"}, Values{"Present","Absent"}] String Ensure;
                [Write, Description("Contentof file.")] String Content;                   
};
```

###Настройка проекта Visual Studio

####Настройка проекта командлета

1. Откройте Visual Studio.
1. Создание проекта C# и укажите имя.
1. Выберите **библиотеки классов** из доступных шаблонов проектов.
1. Щелкните **Ok**.
1. Добавьте ссылку на System.Automation.Management.dll сборку в проект.
1. Измените имя сборки должно соответствовать имени ресурса. В этом случае сборка должна называться **MSFT_XDemoFile**.

###Написание кода командлета

Следующий код C# реализует **Get TargetResource**, **TargetResource набор**, и **TargetResource тест** командлетов.

```C#
Get-TargetResource
       [OutputType(typeof(System.Collections.Hashtable))]
       [Cmdlet(VerbsCommon.Get, "TargetResource")]
       public class GetTargetResource : PSCmdlet
       {
              [Parameter(Mandatory = true)]
              public string Path { get; set; }

///<summary>
/// Implement the logic to write the current state of the resource as a 
/// Hash table with keys being the resource properties 
/// and the Values are the corresponding current values on the target machine.

///</summary>
              protected override void ProcessRecord()
              {
// Download the zip file at the end of this blog to see sample implementation.
 }

Set-TargetResouce
         [OutputType(typeof(void))]
    [Cmdlet(VerbsCommon.Set, "TargetResource")]
    public class SetTargetResource : PSCmdlet
    {
        privatestring _ensure;
        privatestring _content;

[Parameter(Mandatory = true)]
        public string Path { get; set; }

[Parameter(Mandatory = false)]      
       [ValidateSet("Present", "Absent", IgnoreCase = true)]
       public string Ensure {
            get
            {
                // set the default to present.
               return (this._ensure ?? "Present");
            }
            set
            {
                this._ensure = value;
            }
           } 
            public string Content {
            get { return (string.IsNullOrEmpty(this._content) ? "" : this._content); }
            set { this._content = value; }
        }

///<summary>
        /// Implement the logic to set the state of the machine to the desired state.
        ///</summary>
        protected override void ProcessRecord()
        {
//Implement the set method of the resource 
/* Uncomment this section if your resource needs a machine reboot.
PSVariable DscMachineStatus = new PSVariable("DSCMachineStatus", 1, ScopedItemOptions.AllScope);
                this.SessionState.PSVariable.Set(DscMachineStatus);
*/     
  }
    }
Test-TargetResource    
       [Cmdlet("Test", "TargetResource")]
    [OutputType(typeof(Boolean))]
    public class TestTargetResource : PSCmdlet
    {   

        private string _ensure;
        private string _content;

        [Parameter(Mandatory = true)]
        public string Path { get; set; }

        [Parameter(Mandatory = false)]
        [ValidateSet("Present", "Absent", IgnoreCase = true)]
        public string Ensure
        {
            get
            {
                // set the default to present.
                return (this._ensure ?? "Present");
            }
            set
            {
                this._ensure = value;
            }
        }

        [Parameter(Mandatory = false)]
        public string Content
        {
            get { return (string.IsNullOrEmpty(this._content) ? "“:this._content);}
            set { this._content = value; }
        }

///<summary>
/// Write a Boolean value which indicates whether the current machine is in    
/// desired state or not.
        ///</summary>
        protected override void ProcessRecord()
        {
                // Implement the test method of the resource.
        }
}
```

###Развертывание ресурсов

Скомпилированные dll-файла должны сохраняться в структуре файла аналогично ресурсов на основе сценария. Ниже приведен структуру папок для этого ресурса.

```
$env: psmodulepath (folder)
    |- MyDscResources (folder)
        |- DSCResources (folder)
            |- MSFT_XDemoFile (folder)
                |- MSFT_XDemoFile.psd1 (file, optional)
                |- MSFT_XDemoFile.dll (file, required)
                |- MSFT_XDemoFile.schema.mof (file, required)
```

###См. также

####Основные понятия

[Написание настраиваемых DSC ресурсов с помощью MOF](authoringResourceMOF.md)
####Другие ресурсы

[Написание командлета Windows PowerShell](https://msdn.microsoft.com/en-us/library/dd878294.aspx)



