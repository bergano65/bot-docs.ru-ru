---
ms.openlocfilehash: 8073e5f635d20de457e1bf5880c1b5c4c564fab4
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2019
ms.locfileid: "70385992"
---
Перед развертыванием бота нужно подготовить файлы проекта. 
<!-- **C# bots** -->
##### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

Необходимо указать путь к CSPROJ-файлу относительно папки --code-dir. Для этого можно применить аргумент --proj-file-path. Эта команда разрешит аргументы --code-dir и --proj-file-path в значение ./MyBot.csproj.

<!-- **JavaScript bots** -->
##### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

Эта команда получает файл web.config, который необходим для работы приложений Node.js со службами IIS в Службе приложений Azure. Убедитесь, что файл web.config сохранен в корневой каталог бота.

<!-- **TypeScript bots** -->
##### <a name="typescripttabtypescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

Эта команда работает так же, как предложенный выше код JavaScript, но применяется для бота на Typescript.

---

> [!NOTE]
> Команда `az bot prepare-depoloy` создаст файл `.deployment` в папке проекта бота.
