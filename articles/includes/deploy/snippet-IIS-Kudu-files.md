---
ms.openlocfilehash: 3a3d1056e9c6f913efbf563131a5ab377a57d2d0
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479314"
---

Перед развертыванием бота нужно подготовить файлы проекта C#, JavaScript или Typescript. При развертывании бота Python этот шаг можно пропустить.

<!-- **C# bots** -->
##### <a name="c"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

Необходимо указать путь к CSPROJ-файлу относительно папки --code-dir. Для этого можно применить аргумент --proj-file-path. Эта команда разрешит аргументы --code-dir и --proj-file-path в значение ./MyBot.csproj.

<!-- **JavaScript bots** -->
##### <a name="javascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

Эта команда получает файл web.config, который необходим для работы приложений Node.js со службами IIS в Службе приложений Azure. Убедитесь, что файл web.config сохранен в корневой каталог бота.

<!-- **TypeScript bots** -->
##### <a name="typescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

Эта команда работает так же, как предложенный выше код JavaScript, но применяется для бота на Typescript.

---

> [!NOTE]
>  Для ботов C# команда `az bot prepare-deploy` создаст файл `.deployment` в папке проекта бота.
> Для ботов JavaScript эта команда создаст файл `web.config` в папке проекта бота.
> Для ботов TypeScript эта команда создаст два файла `web.config`. Один из них находится в папке проекта, а другой — в папке **src**, которая содержится в папке проекта.


