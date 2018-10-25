---
title: Настройка взаимодействия с пользователем с помощью языка шаблонов | Документация Майкрософт
description: Узнайте, как настроить запросы FormFlow и переопределить шаблоны FormFlow, используя язык шаблонов и пакет SDK Bot Builder для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bc0a2819f3adea63b53e464808f3bbaf5b93814a
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998051"
---
# <a name="customize-user-experience-with-pattern-language"></a>Настройка взаимодействия с пользователем с помощью языка шаблонов

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

При настройке запроса или переопределении шаблона по умолчанию можно использовать язык шаблонов, чтобы задать содержимое и (или) отформатировать запрос. 

## <a name="prompts-and-templates"></a>Запросы и шаблоны

[Запрос][promptAttribute] определяет сообщение, которое отправляется пользователю, чтобы запросить фрагмент информации или подтверждение. Запрос можно настроить явно с помощью атрибута [Prompt](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-prompt-attribute) или неявно с помощью [IFormBuilder<T>.Field][field]. 

Формы используют шаблоны, чтобы автоматически создавать запросы и другие элементы, например справку. Шаблон по умолчанию класса или поля можно переопределить с помощью атрибута [Template](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-template-attribute). 

> [!TIP]
> Класс [FormConfiguration.Templates][formConfiguration] определяет набор встроенных шаблонов, которые предоставляют хорошие примеры использования языка шаблонов.

## <a name="elements-of-pattern-language"></a>Элементы языка шаблонов

В языке шаблонов используются фигурные скобки (`{}`) для определения элементов, которые во время выполнения будут заменены фактическими значениями. В таблице ниже перечислены элементы языка шаблонов.

| Элемент | ОПИСАНИЕ |
|----|----|
| `{<format>}` | Отображает значение текущего поля (поля, к которому применяется атрибут). |
| `{&}` | Отображает описание текущего поля (если не указано иное, это имя поля). |
| `{<field><format>}` | Отображает значение именованного поля. | 
| `{&<field>}` | Отображает описание именованного поля. |
| <code>{&#124;&#124;}</code> | Отображает текущие выбранные значения. Это может быть текущее значение поля, значение "no preference" или значения перечисления. |
| `{[{<field><format>} ...]}` | Отображает список значений именованных полей с помощью элементов [Separator][separator] и [LastSeparator][lastSeparator], используемых для разделения отдельных значений в списке. |
| `{*}` | Отображает одну строку для каждого активного поля. Каждая строка содержит описание и текущее значение поля. | 
| `{*filled}` | Отображает одну строку для каждого активного поля, которое содержит фактическое значение. Каждая строка содержит описание и текущее значение поля. |
| `{<nth><format>}` | Обычный описатель формата C#, который относится к аргументу nth шаблона. Список доступных аргументов приведен в описании [TemplateUsage][templateUsage]. |
| `{?<textOrPatternElement>...}` | Условная подстановка. Если все ссылочные элементы шаблона имеют значения, эти значения заменяются и используется все выражение. |

Для элементов, перечисленных выше:

- Заполнитель `<field>` — это имя поля в классе формы. Например, если класс содержит поле с именем `Size`, можно указать `{Size}` в качестве шаблона элемента. 

- Многоточие (`"..."`) в шаблоне элемента указывает, что этот элемент может содержать несколько значений.

- Заполнитель `<format>` — это описатель формата C#. Например, если класс содержит поле с именем `Rating` и типом `double`, можно указать `{Rating:F2}` в качестве элемента шаблона для отображения точности до двух разрядов.

- Заполнитель `<nth>` означает аргумент nth шаблона.

### <a name="pattern-language-within-a-prompt-attribute"></a>Язык шаблонов в атрибуте Prompt

В этом примере используется элемент `{&}`, чтобы показать описание поля `Sandwich`, и элемент `{||}`, чтобы отобразить список вариантов значений для этого поля.

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns1)]

В результате получается следующий запрос.

```console
What kind of sandwich would you like?
1. BLT
2. Black Forest Ham
3. Buffalo Chicken
4. Chicken And Bacon Ranch Melt
5. Cold Cut Combo
6. Meatball Marinara
7. Oven Roasted Chicken
8. Roast Beef
9. Rotisserie Style Chicken
10. Spicy Italian
11. Steak And Cheese
12. Sweet Onion Teriyaki
13. Tuna
14. Turkey Breast
15. Veggie
>
```

## <a name="formatting-parameters"></a>Параметры форматирования

Запросы и шаблоны поддерживают следующие параметры форматирования.

| Использование | ОПИСАНИЕ |
|----|----|
| `AllowDefault` | Применяется к элементам шаблона <code>{&#124;&#124;}</code>. Определяет, должна ли форма показывать текущее значение поля для выбора. Если указано значение `true`, то текущее значение поля отображается как возможное значение. Значение по умолчанию — `true`. |
| `ChoiceCase` | Применяется к элементам шаблона <code>{&#124;&#124;}</code>. Определяет, нормализуется ли текст каждого из возможных значений (например, начинается ли каждое слово с прописной буквы). Значение по умолчанию — `CaseNormalization.None`. Возможные значения приведены в описании [CaseNormalization][caseNormalization]. |
| `ChoiceFormat` | Применяется к элементам шаблона <code>{&#124;&#124;}</code>. Определяет, следует ли отображать список возможных значений виде нумерованного или маркированного списка. Для отображения нумерованного списка задайте для `ChoiceFormat` значение `{0}` (по умолчанию). Для отображения маркированного списка задайте для `ChoiceFormat` значение `{1}`. |
| `ChoiceLastSeparator` | Применяется к элементам шаблона <code>{&#124;&#124;}</code>. Определяет, содержит ли встроенный список возможных значений разделитель перед последним значением. |
| `ChoiceParens` | Применяется к элементам шаблона <code>{&#124;&#124;}</code>. Определяет, отображается ли встроенный список возможных значений в скобках. Если указано значение `true`, список возможных значений отображается в скобках. Значение по умолчанию — `true`. |
| `ChoiceSeparator` | Применяется к элементам шаблона <code>{&#124;&#124;}</code>. Определяет, содержит ли встроенный список возможных значений разделитель перед каждым значением, кроме последнего. | 
| `ChoiceStyle` | Применяется к элементам шаблона <code>{&#124;&#124;}</code>. Определяет, как отображается список возможных значений: в одной строке или построчно. По умолчанию используется значение `ChoiceStyleOptions.Auto`, которое указывает среде выполнения, следует ли отображать возможные значения в строке или в виде списка. Возможные значения приведены в описании [ChoiceStyleOptions][choiceStyleOptions]. |
| `Feedback` | Применяется только для запросов. Определяет, дублирует ли форма выбор пользователя, чтобы указать, что форме понятно выбранное значение. По умолчанию используется значение `FeedbackOptions.Auto`, при котором входные данные пользователя выводятся только в том случае, если их часть не распознана. Возможные значения приведены в описании [FeedbackOptions][feedbackOptions]. |
| `FieldCase` | Определяет, нормализуется ли текст описания поля (например, начинается ли каждое слово с прописной буквы). По умолчанию используется значение `CaseNormalization.Lower`, при котором описание преобразуется в нижний регистр. Возможные значения приведены в описании [CaseNormalization][caseNormalization]. |
| `LastSeparator` | Применяется к элементам шаблона `{[]}`. Определяет, содержит ли массив элементов разделитель перед последним элементом. |
| `Separator` | Применяется к элементам шаблона `{[]}`. Определяет, содержит ли массив элементов разделитель перед каждым элементом массива, кроме последнего. |
| `ValueCase` | Определяет, нормализуется ли текст значения поля (например, начинается ли каждое слово с прописной буквы). По умолчанию используется значение `CaseNormalization.InitialUpper`, при котором первая буква каждого слова преобразовывается в верхний регистр. Возможные значения приведены в описании [CaseNormalization][caseNormalization]. |

### <a name="prompt-attribute-with-formatting-parameter"></a>Атрибут Prompt с параметром форматирования 

В этом примере используется параметр, `ChoiceFormat` чтобы указать, что список возможных значений должен отображаться в виде маркированного списка.

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns2)]

В результате получается следующий запрос.

```console
What kind of sandwich would you like?
* BLT
* Black Forest Ham
* Buffalo Chicken
* Chicken And Bacon Ranch Melt
* Cold Cut Combo
* Meatball Marinara
* Oven Roasted Chicken
* Roast Beef
* Rotisserie Style Chicken
* Spicy Italian
* Steak And Cheese
* Sweet Onion Teriyaki
* Tuna
* Turkey Breast
* Veggie
>
```

## <a name="sample-code"></a>Пример кода

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные функции FormFlow](bot-builder-dotnet-formflow.md)
- [Дополнительные функции FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Настройка формы с помощью FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Локализация содержимого формы](bot-builder-dotnet-formflow-localize.md)
- [Определение формы с помощью схемы JSON](bot-builder-dotnet-formflow-json-schema.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочник по пакету SDK Bot Builder для .NET</a>

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[field]: /dotnet/api/microsoft.bot.builder.formflow.iformbuilder-1.field

[formConfiguration]: /dotnet/api/microsoft.bot.builder.formflow.formconfiguration

[separator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.separator

[lastSeparator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.lastseparator

[templateUsage]: /dotnet/api/microsoft.bot.builder.formflow.templateusage

[caseNormalization]: /dotnet/api/microsoft.bot.builder.formflow.casenormalization

[choiceStyleOptions]: /dotnet/api/microsoft.bot.builder.formflow.choicestyleoptions

[feedbackOptions]: /dotnet/api/microsoft.bot.builder.formflow.feedbackoptions
