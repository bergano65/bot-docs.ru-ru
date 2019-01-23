---
title: Локализация содержимого формы | Документация Майкрософт
description: Сведения о локализации содержимого формы с помощью FormFlow и пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3ec3d12a7d35f65adca901395edff2db3ab71c66
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225589"
---
# <a name="localize-form-content"></a>Локализация содержимого формы

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Язык для локализации формы определяется значениями [CurrentUICulture](https://msdn.microsoft.com/library/system.threading.thread.currentuiculture(v=vs.110).aspx) и [CurrentCulture](https://msdn.microsoft.com/library/system.threading.thread.currentculture(v=vs.110).aspx) для текущего потока.
По умолчанию язык и региональные параметры определяются в поле **Locale** для текущего сообщения, но вы можете переопределить это поведение.
В зависимости от структуры вашего бота локализованную информацию можно получать из трех разных источников:

- встроенные локализации для **PromptDialog** и **FormFlow**;
- файл ресурсов, созданный для статических строк в форме;
- файл ресурсов, созданный для строк в динамически вычисляемых полях, сообщениях или подтверждениях.

## <a name="generate-a-resource-file-for-the-static-strings-in-your-form"></a>Создание файла ресурсов для статических строк в форме

К статическим строкам относятся те строки в форме, которые она создает на основе параметров класса C#, и те, которые вы определяете в качестве запросов, шаблонов, сообщений и (или) подтверждений.
Строки, создаваемые из встроенных шаблонов, не считаются статическими, так как они уже локализованы.
Так как многие такие строки создаются в форме автоматически, для них нельзя напрямую использовать обычные строки ресурсов C#.
Вместо этого вы можете создать файл ресурсов для статических строк в форме, используя метод `IFormBuilder.SaveResources` или средство **RView**, которое входит в пакет SDK Bot Builder для .NET.

### <a name="use-iformbuildersaveresources"></a>Использование IFormBuilder.SaveResources

Чтобы создать файл ресурсов, можно вызвать для формы метод [IFormBuilder.SaveResources][saveResources], который сохраняет строки в RESX-файл.

### <a name="use-rview"></a>Использование RView

Кроме того, можно создать файл ресурсов на основе существующих файлов .dll или .exe с помощью средства <a href="https://aka.ms/v3-cs-RView-library" target="_blank">RView</a>, которое входит в пакет SDK Bot Builder для .NET.
Чтобы создать RESX-файл, запустите **rview** и укажите сборку, которая содержит статический метод создания формы, и путь к этому методу.
В этом фрагменте кода показано, как создать файл ресурсов `Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.resx` с помощью **RView**.

```csharp
rview -g Microsoft.Bot.Sample.AnnotatedSandwichBot.dll Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.BuildForm
```

Этот фрагмент содержит часть RESX-файла, созданного при выполнении команды **rview**.

```xml
<data name="Specials_description;VALUE" xml:space="preserve">
<value>Specials</value>
</data>
<data name="DeliveryAddress_description;VALUE" xml:space="preserve">
<value>Delivery Address</value>
</data>
<data name="DeliveryTime_description;VALUE" xml:space="preserve">
<value>Delivery Time</value>
</data>
<data name="PhoneNumber_description;VALUE" xml:space="preserve">
<value>Phone Number</value>
</data>
<data name="Rating_description;VALUE" xml:space="preserve">
<value>your experience today</value>
</data>
<data name="message0;LIST" xml:space="preserve">
<value>Welcome to the sandwich order bot!</value>
</data>
<data name="Sandwich_terms;LIST" xml:space="preserve">
<value>sandwichs?</value>
</data>
```

## <a name="configure-your-project"></a>Настройка проекта

Создав файл ресурсов, добавьте его в проект и задайте нейтральный язык: 

1. Щелкните проект правой кнопкой мыши и выберите **Приложение**.
2. Щелкните **Сведения о сборке.**
3. Выберите значение **Нейтральный язык**, которое обозначает язык разработки бота.

При создании формы метод [IFormBuilder.Build][build] будет автоматически искать ресурсы с правильным именем типа формы и применять их для локализации статических строк в этой форме. 

> [!NOTE]
> Динамически вычисляемые поля, которые определяются с помощью [Advanced.Field.SetDefine][setDefine] (как описано в руководстве по [использованию динамических полей](bot-builder-dotnet-formflow-formbuilder.md#dynamically-define-field-values-confirmations-and-messages)) нельзя локализовать как статические поля, так как строки для динамически вычисляемых полей создаются во время заполнения формы. Но вы можете локализовать динамически вычисляемые поля с помощью обычных механизмов локализации C#.

### <a name="localize-resource-files"></a>Локализация файлов ресурсов 

Добавив в проект файлы ресурсов, вы можете локализовать их с помощью <a href="https://developer.microsoft.com/windows/develop/multilingual-app-toolkit" target="_blank">набора средств для многоязычных приложений (MAT)</a>. Установите и включите для проекта **MAT**:

1. Выберите проект в обозревателе решений Visual Studio.
2. Щелкните **Средства**, **Набор средств для многоязычных приложений** и **Включить**.
3. Щелкните проект правой кнопкой мыши и выберите **Набор средств для многоязычных приложений**, **Добавить переводы**, чтобы выбрать нужные переводы. Это действие создаст стандартные <a href="https://en.wikipedia.org/wiki/XLIFF" target="_blank">XLF</a>-файлы, которые можно перевести автоматически или вручную.

> [!NOTE]
> Далее в этой статье описано, как локализовать содержимое с помощью набора средств для многоязычных приложений. Но вы можете реализовать локализацию с помощью других средств.

## <a name="see-it-in-action"></a>Пример применения

Этот пример кода основан на примере из руководства по [настройке формы с помощью FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) и реализует для него локализацию, как описано выше. В этом примере класс `DynamicSandwich` (здесь не показан) содержит сведения о локализации для динамически вычисляемые поля, сообщений и подтверждений.

[!code-csharp[Build localized form](../includes/code/dotnet-formflow-localize.cs#buildLocalizedForm)]

В этом фрагменте показано взаимодействие между ботом и пользователем, если `CurrentUICulture` имеет значение **French** (французский язык).

```console
Bienvenue sur le bot d'ordre "sandwich" !
Quel genre de "sandwich" vous souhaitez sur votre "sandwich"?
 1. BLT
 2. Jambon Forêt Noire
 3. Poulet Buffalo
 4. Faire fondre le poulet et Bacon Ranch
 5. Combo de coupe à froid
 6. Boulette de viande Marinara
 7. Poulet rôti au four
 8. Rôti de boeuf
 9. Rotisserie poulet
 10. Italienne piquante
 11. Bifteck et fromage
 12. Oignon doux Teriyaki
 13. Thon
 14. Poitrine de dinde
 15. Veggie
> 2

Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> ?
* Vous renseignez le champ longueur.Réponses possibles:
* Vous pouvez saisir un numéro 1-2 ou des mots de la description. (Six pouces, ou Pied Long)
* Retourner à la question précédente.
* Assistance: Montrez les réponses possibles.
* Abandonner: Abandonner sans finir
* Recommencer remplir le formulaire. (Vos réponses précédentes sont enregistrées.)
* Statut: Montrer le progrès en remplissant le formulaire jusqu'à présent.
* Vous pouvez passer à un autre champ en entrant son nom. ("Sandwich", Longueur, Pain, Fromage, Nappages, Sauces, Adresse de remise, Délai de livraison, ou votre expérience aujourd'hui).
Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> 1

Quel genre de pain vous souhaitez sur votre "sandwich"?
 1. Neuf grains de blé
 2. Neuf grains miel avoine
 3. Italien
 4. Fromage et herbes italiennes
 5. Pain plat
> neuf
Par pain "neuf" vouliez-vous dire (1. Neuf grains miel avoine, ou 2. Neuf grains de blé)
```

## <a name="sample-code"></a>Пример кода

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные функции FormFlow](bot-builder-dotnet-formflow.md)
- [Дополнительные функции FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Customize a form using FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) (Настройка формы с помощью FormBuilder)
- [Define a form using JSON schema](bot-builder-dotnet-formflow-json-schema.md) (Определение формы с помощью схемы JSON)
- [Customize user experience with pattern language](bot-builder-dotnet-formflow-pattern-language.md) (Настройка взаимодействия с помощью языка шаблонов)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>

[build]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1.build 

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[saveResources]: /dotnet/api/microsoft.bot.builder.formflow.iform-1.saveresources
