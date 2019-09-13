---
ms.openlocfilehash: 28f49af1610f3c154e76d1cfcdb1333f08374f6c
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386103"
---
Регистрация приложения означает, что вы сможете использовать Azure AD для аутентификации пользователей и для запроса доступа к ресурсам пользователей. Боту нужно зарегистрированное в Azure приложение, которое предоставит боту доступ к службе Bot Framework для отправки и получения сообщений с проверкой подлинности. Чтобы зарегистрировать приложение с помощью Azure CLI, выполните следующую команду:

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| Параметр   | ОПИСАНИЕ |
|:---------|:------------|
| display-name | Отображаемое имя приложения. |
| password | Пароль приложения, также называемый "секрет клиента". Пароль должен содержать не менее 16 символов, среди которых есть хотя бы по одной букве в верхнем и нижнем регистрах и хотя бы один специальный символ.|
| available-to-other-tenants| Указывает, может ли приложение использоваться из любого клиента Azure AD. Укажите `true`, чтобы разрешить боту работать с каналами службы Azure Bot.|

Приведенная выше команда выводит код JSON с ключом `appId`. Сохраните значение этого ключа для развертывания с помощью ARM, где этот ключ нужно указать в параметре `appId`. Предоставленный пароль будет использоваться в параметре `appSecret`.

> [!NOTE] 
> Если вы хотите использовать существующую регистрацию приложения, можно использовать эту команду:
> ``` cmd
> az bot create --kind webapp --resource-group "<name-of-resource-group>" --name "<name-of-web-app>" --appid "<existing-app-id>" --password "<existing-app-password>" --lang <Javascript|Csharp>
> ```
