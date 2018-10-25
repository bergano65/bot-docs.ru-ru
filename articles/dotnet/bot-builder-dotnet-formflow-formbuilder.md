---
title: Настройка формы с помощью FormBuilder | Документация Майкрософт
description: Узнайте, как динамически изменять и настраивать ход и содержимое общения с помощью FormBuilder для пакета SDK Bot Builder для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1c4e60f76ecebfa01664500b8343d60ccff0064c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997701"
---
# <a name="customize-a-form-using-formbuilder"></a>Настройка формы с помощью FormBuilder

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В разделе [Основные функции FormFlow](bot-builder-dotnet-formflow.md) описывается базовая реализация FormFlow, которая обеспечивает относительно общие возможности взаимодействия с пользователем. В разделе [Дополнительные функции FormFlow](bot-builder-dotnet-formflow-advanced.md) описывается, как можно настроить взаимодействие с пользователем с помощью бизнес-логики и атрибутов. В этой статье описывается, как можно использовать [FormBuilder][formBuilder] для дополнительной настройки взаимодействия с пользователем, указав последовательность, в которой форма выполняет действия, и динамически определяя значения полей, подтверждения и сообщения. 

## <a name="dynamically-define-field-values-confirmations-and-messages"></a>Динамическое определение значений полей, подтверждений и сообщений

С помощью FormBuilder можно динамически определять значения полей, подтверждения и сообщения.

### <a name="dynamically-define-field-values"></a>Динамическое определение значений полей 

Бот для заказа сандвичей, который предназначен для добавления бесплатного напитка или печенья в любой заказ футового сандвича, использует поле `Sandwich.Specials` для хранения данных о бесплатных позициях. В этом случае значение поля `Sandwich.Specials` следует динамически задавать для каждого заказа в соответствии с тем, содержит ли он футовый сандвич. 

Поле `Specials` задано как необязательное, и значение "None" используется в качестве текста для выбора, что указывает на отсутствие предпочтения.

[!code-csharp[Field definition](../includes/code/dotnet-formflow-formbuilder.cs#fieldDefinition)]

Данный пример кода показывает, как динамически задавать значение поля `Specials`. 

[!code-csharp[Define value](../includes/code/dotnet-formflow-formbuilder.cs#defineValue)]

В этом примере метод [Advanced.Field.SetType][setType] указывает тип поля (`null` представляет поле перечисления). Метод [Advanced.Field.SetActive][setActive] указывает, что поле должно быть включено, только если длина сандвича равна `Length.FootLong`. Наконец, метод [Advanced.Field.SetDefine][setDefine] указывает асинхронный делегат, который определяет это поле. Этот делегат передает текущий объект состояния и поле [Advanced.Field][field], которое определяется динамически. Делегат использует текучие методы поля для динамического определения значений. В этом примере значения являются строками, а методы `AddDescription` и `AddTerms` определяют описания и термины для каждого значения.

> [!NOTE]
> Чтобы динамически определять значение поля, можно или самостоятельно реализовать [Advanced.IField][iField], или оптимизировать процесс, воспользовавшись классом [Advanced.FieldReflector][FieldReflector], как показано в приведенном выше примере. 

### <a name="dynamically-define-messages-and-confirmations"></a>Динамическое определение сообщений и подтверждений

С помощью FormBuilder можно также динамически определять сообщения и подтверждения. Каждое сообщение и подтверждение выполняется только в том случае, если предыдущие шаги в форме неактивны или завершены. 

Данный пример кода показывает динамически создаваемое подтверждение, в котором вычисляется стоимость сандвича. 

[!code-csharp[Define confirmation](../includes/code/dotnet-formflow-formbuilder.cs#defineConfirmation)]

## <a name="customize-a-form-using-formbuilder"></a>Настройка формы с помощью FormBuilder

Данный пример кода использует FormBuilder для определения шагов формы, [проверки выбранных параметров](bot-builder-dotnet-formflow-advanced.md#add-business-logic) и [динамического определения значения поля и подтверждения](#dynamically-define-field-values-confirmations-and-messages). По умолчанию шаги в форме выполняются в порядке, в котором они перечислены. Тем не менее можно пропускать шаги для полей, которые уже содержат значения, или если указан явный переход. 

[!code-csharp[FormBuilder form](../includes/code/dotnet-formflow-formbuilder.cs#formBuilderForm)]

В этом примере форма выполняет следующие шаги.

- Отображает приветственное сообщение. 
- Заполняет `SandwichOrder.Sandwich`. 
- Заполняет `SandwichOrder.Length`. 
- Заполняет `SandwichOrder.Bread`. 
- Заполняет `SandwichOrder.Cheese`. 
- Заполняет `SandwichOrder.Toppings` и добавляет недостающие значения, если пользователь выбрал `ToppingOptions.Everything`. -. Отображает сообщение, подтверждающее выбранные ингредиенты. 
- Заполняет `SandwichOrder.Sauces`. 
- [Динамически определяет](#dynamically-define-field-values) значение поля `SandwichOrder.Specials`. 
- [Динамически определяет](#dynamically-define-messages-and-confirmations) подтверждение стоимости сандвича. 
- Заполняет `SandwichOrder.DeliveryAddress` и [проверяет](bot-builder-dotnet-formflow-advanced.md#add-business-logic) полученную строку. Если адрес не начинается с цифры, форма отображает сообщение. 
- Заполняет `SandwichOrder.DeliveryTime` с помощью пользовательского запроса. 
- Подтверждает заказ. 
- Добавляет все остальные поля, которые были определены в классе, но не были явно указаны с помощью `Field`. (Если бы пример не вызывал метод `AddRemainingFields`, форма не содержала бы поля, которые не были явно указаны.) 
- Отображает сообщение с благодарностью. 
- Определяет обработчик `OnCompletionAsync` для обработки заказа. 

## <a name="sample-code"></a>Пример кода

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные функции FormFlow](bot-builder-dotnet-formflow.md)
- [Дополнительные функции FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Локализация содержимого формы](bot-builder-dotnet-formflow-localize.md)
- [Определение формы с помощью схемы JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Customize user experience with pattern language](bot-builder-dotnet-formflow-pattern-language.md) (Настройка взаимодействия с помощью языка шаблонов)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочник по пакету SDK Bot Builder для .NET</a>

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1

[setType]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.settype

[setActive]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setactive

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[FieldReflector]: /dotnet/api/microsoft.bot.builder.formflow.advanced.fieldreflector-1
