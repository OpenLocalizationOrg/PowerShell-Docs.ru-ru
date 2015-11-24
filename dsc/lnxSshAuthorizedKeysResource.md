#DSC для Linux nxSshAuthorizedKeys ресурсов

**NxAuthorizedKeys** ресурс в PowerShell требуемой состояние конфигурации (DSC) обеспечивает механизм управления авторизованных ssh ключи для указанного пользователя.

##Синтаксис

```
nxAuthorizedKeys <string> #ResourceName
{
    KeyComment = <string>
    [ Ensure = <string> { Absent | Present }  ]
    [ Username = <string> ]
    [ Key = <string> ]
    [ DependsOn = <string[]> ]

}
```

##Свойства

| Свойство| Описание|
|---|---|
| KeyComment| Уникальный комментарий для ключа.Используется для однозначной идентификации ключей.|
| Убедитесь| Указывает, определен ли ключ.Установите это свойство для «Отсутствует», чтобы убедиться, что ключ не существует в файле авторизованных ключей пользователя.Задайте для него значение «Отсутствует», чтобы убедиться, что ключ определен в файл ключа авторизованного пользователя.|
| Имя пользователя| Имя пользователя для управления ssh авторизованных ключи.Если не определен, пользователь по умолчанию — «root».|
| Ключ| Содержимое раздела.Является обязательным, если **Убедитесь, что** задано значение «Отсутствует».|
| DependsOn| Указывает, что конфигурация другого ресурса должна выполняться перед настройкой этого ресурса.Например если **идентификатор** ресурса блок скрипта конфигурации, необходимо запустить сначала — **ResourceName** и его тип является **типа ресурса**, синтаксис для использования этого свойства `DependsOn = "[типа ресурса] ResourceName"`.|

##Пример

В следующем примере определяется авторизованных ssh открытый ключ для пользователя «monuser».

```
Import-DSCResource -Module nx 

Node $node {

nxSshAuthorizedKeys myKey{
   KeyComment = "myKey"
   Ensure = "Present"
   Key = 'ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEA0b+0xSd07QXRifm3FXj7Pn/DblA6QI5VAkDm6OivFzj3U6qGD1VJ6AAxWPCyMl/qhtpRtxZJDu/TxD8AyZNgc8aN2CljN1hOMbBRvH2q5QPf/nCnnJRaGsrxIqZjyZdYo9ZEEzjZUuMDM5HI1LA9B99k/K6PK2Bc1NLivpu7nbtVG2tLOQs+GefsnHuetsRMwo/+c3LtwYm9M0XfkGjYVCLO4CoFuSQpvX6AB3TedUy6NZ0iuxC0kRGg1rIQTwSRcw+McLhslF0drs33fw6tYdzlLBnnzimShMuiDWiT37WqCRovRGYrGCaEFGTG2e0CN8Co8nryXkyWc6NSDNpMzw== rsa-key-20150401'
   UserName = "monuser"
} 
}
```





