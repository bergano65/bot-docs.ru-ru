---
ms.openlocfilehash: 3cd58ddac050139b93eb4ee39909b04289afd821
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77446794"
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
>  Для ботов на C# и JavaScript с помощью команды `az bot prepare-deploy` в папке проекта бота создается файл `.deployment`.
> Для ботов на TypeScript с помощью этой команды создается два файла `web.config`. Один из них находится в папке проекта, а другой — в папке **src**, которая содержится в папке проекта. 
