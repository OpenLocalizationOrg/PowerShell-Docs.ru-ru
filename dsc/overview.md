#Общие сведения о состоянии конфигурации требуемого Windows PowerShell

> Область применения: Windows PowerShell 4.0, Windows PowerShell 5.0

В этом разделе описывается компонент конфигурации состояния требуемой Windows PowerShell (DSC) в Windows PowerShell. В этом разделе можно использовать для получения обзора DSC и поиска документация, необходимые для понимания и использования DSC.

##Описание компонента

DSC представляет новую платформу управления в Windows PowerShell, позволяющий развертывание и управление данные конфигурации для служб программного обеспечения и управление средой, в которой эти службы запускаются.

DSC предоставляет набор расширений языка Windows PowerShell, новые командлеты Windows PowerShell и ресурсы, которые можно использовать для декларативно указать способ настроить среду программного обеспечения. Он также предоставляет средства для обслуживания и управлять существующих конфигураций.

##Практическое применение

Ниже приведены некоторые примеры сценариев, где можно использовать встроенные DSC ресурсы для настройки и управления в автоматического набора компьютеров (также известный как целевые узлы).

* Включение или отключение ролей и компонентов сервера
* Управление параметрами реестра
* Управление файлами и каталогами
* Запуск, остановка и управление процессами и службами
* Управление группами и учетных записей пользователей
* Развертывание нового программного обеспечения
* Управление переменных среды
* Выполнение скриптов Windows PowerShell
* Исправления конфигурации, который за от требуемое состояние
* Обнаружение состояния фактической конфигурации на заданном узле

##Основные понятия

DSC представляют собой декларативные платформу, используется для настройки, развертывания и управления систем. Он состоит из трех основных компонентов:

* [Конфигурации](configurations.md) — декларативный сценариев PowerShell, которые определения и настройки экземпляров ресурсов. После выполнения конфигурации DSC (ресурсы и вызывается в конфигурации) будет просто «сделать это», проверяется, что система существует в состоянии, изложенные в конфигурации. DSC конфигурации также идемпотентными: локальный диспетчер конфигурации (НОК) будут продолжать убедитесь, что машины настроены в любое состояние конфигурации объявляется.
* Ресурсы, императивного строительные блоки DSC, записываемые моделировать различные компоненты подсистему и реализовать их изменение состояний потока управления. Они находятся в пределах модули PowerShell и могут записываться в модели, что-либо как общий файл или процесс Windows или как на сервере IIS или виртуальный компьютер, работающий в Azure.
* Локальный диспетчер конфигурации (НОК) — механизм, с помощью которого DSC облегчает взаимодействие между ресурсами и конфигураций. НОК регулярно опрашивает системы с помощью потока управления, реализуемый ресурсы, чтобы убедиться, что состояние определяется конфигурация поддерживается. Если из состояния системы, НОК использует логику на дополнительные ресурсы позволит "," в зависимости от конфигурации объявления.

DSC также включает ряд новых ключевых слов языка, командлеты и средства, которые разрешить создание конфигураций построения справки DSC ресурсы, вызывать конфигураций и управление НОК. Многие из этих командлетов можно найти в Windows 8.1 в составе модуля PsDesiredStateConfig (включая `начала DscConfiguration`, `набора DscLocalConfigurationManager`, и `Get DscResource`). XDscResourceDesigner (в [PowerShell коллекции](https://www.powershellgallery.com/packages/xDSCResourceDesigner/)) — это набор командлетов, которые упрощают разработку DSC ресурсов.

##См. также

* [DSC конфигураций](configurations.md)
* [Ресурсы DSC](resources.md)
* [Настройка локальных Configuration Manager](metaconfig.md)




