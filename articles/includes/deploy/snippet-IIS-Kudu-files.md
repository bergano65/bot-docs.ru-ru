---
ms.openlocfilehash: bbe74a9a82d3bd04593384d825d373bfab35e3db
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039782"
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
>  Для ботов на C# и JavaScript с помощью команды `az bot prepare-depoloy` в папке проекта бота создается файл `.deployment`.
> Для ботов на TypeScript с помощью этой команды создается два файла `web.config`. Один из них находится в папке проекта, а другой — в папке **src**, которая содержится в папке проекта. 
