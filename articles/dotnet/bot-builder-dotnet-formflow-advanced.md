---
title: Дополнительные функции FormFlow — Служба Azure Bot
description: Сведения о настройке взаимодействия с пользователем с помощью FormFlow и пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5620fd3a0e26cf7b56772e6bc8f47b8ceac596cd
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75796673"
---
# <a name="advanced-features-of-formflow"></a>Дополнительные функции FormFlow

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В статье [Основные функции FormFlow](bot-builder-dotnet-formflow.md) описывается базовая реализация FormFlow, которая предоставляет довольно общее взаимодействие с пользователем. Чтобы предоставить более настроенное взаимодействие с пользователем с помощью FormFlow, вы можете указать начальное состояние формы, добавить бизнес-логику для управления взаимозависимостью между полями и обработкой входных данных и с помощью атрибутов настроить запросы, переопределить шаблоны, назначить необязательные поля, сопоставить и проверить входные данные пользователя. 

## <a name="specify-initial-form-state-and-entities"></a>Установка исходного состояния и сущностей формы

Во время запуска [FormDialog][formDialog] при необходимости вы можете передать экземпляр состояния. Если вы передадите экземпляр своего состояния, то по умолчанию FormFlow пропустит шаги настройки для любых полей, которые уже содержат значения; пользователю не нужно будет настраивать эти поля. Чтобы форма отправляла пользователю запрос на настройку всех полей (включая те поля, которые уже содержат исходные значения), запуская `FormDialog` передайте метод [FormOptions.PromptFieldsWithValues][promptFieldsWithValues]. Если поле содержит исходное значение, запрос будет использовать это значение в качестве значения по умолчанию.

Вы также можете передать сущности [LUIS](https://luis.ai/) для привязки к состоянию. Если `EntityRecommendation.Type` — это путь к полю в вашем классе C#, `EntityRecommendation.Entity` будет передаваться через распознаватель, чтобы привязываться к вашему полю. FormFlow пропустит шаги настройки любых полей, привязанных к сущности; пользователю не нужно будет настраивать эти поля. 

## <a name="add-business-logic"></a>Добавление бизнес-логики 

Чтобы обработать взаимозависимости между полями формы или применять конкретную логику в процессе получения или настройки значения поля, укажите бизнес-логику в функции проверки. Функция проверки позволяет управлять состоянием и возвращать объект [ValidateResult][validateResult], который может содержать следующее: 

- строка обратной связи, которая описывает причину недопустимости значения;
- преобразованное значение;
- набор вариантов для уточнения значения;

В этом примере кода показана функция проверки для поля `Toppings`. Если входные данные для поля содержат значение перечисления `ToppingOptions.Everything`, функция гарантирует, что значение поля `Toppings` содержит полный список начинок.

[!code-csharp[Validation function](../includes/code/dotnet-formflow-advanced.cs#validationFunction)]

В дополнение к функции проверки вы можете добавить атрибут [Term](#match-user-input-using-the-terms-attribute) для сопоставления с пользовательскими выражениями, такими как "Все" или "Нет".

[!code-csharp[Terms for Toppings](../includes/code/dotnet-formflow-advanced.cs#toppingsTerms)]

Используя функцию проверки, описанную выше, этот фрагмент кода показывает взаимодействие между ботом и пользователем, когда пользователь вводит запрос: everything but Jalapenos (все, кроме перца халапеньо). 

```console
Please select one or more toppings (current choice: No Preference)
 1. Everything
 2. Avocado
 3. Banana Peppers
 4. Cucumbers
 5. Green Bell Peppers
 6. Jalapenos
 7. Lettuce
 8. Olives
 9. Pickles
 10. Red Onion
 11. Spinach
 12. Tomatoes
> everything but jalapenos
For sandwich toppings you have selected Avocado, Banana Peppers, Cucumbers, Green Bell Peppers, Lettuce, Olives, Pickles, Red Onion, Spinach, and Tomatoes.
```

## <a name="formflow-attributes"></a>Атрибуты FormFlow

Вы можете добавить эти атрибуты C# в свой класс, чтобы настроить поведение диалогового окна FormFlow.

| attribute | Назначение |
|----|----| 
| [Describe][describeAttribute] | Показывает, как изменить поле или значение в шаблоне или карточке |
| [Numeric][numericAttribute] | Ограничивает принятые значения числового поля |
| [Необязательно][optionalAttribute] | Отмечает поле как необязательное |
| [Шаблон][patternAttribute] | Определяет регулярное выражение для проверки строкового поля |
| [Prompt][promptAttribute] | Определяет запрос для поля |
| [Шаблон][templateAttribute] | Определяет шаблон, с помощью которого будут создаваться запросы или значения в запросах |
| [Terms][termsAttribute] | Определяет входные термины, соответствующие полю или значению |

## <a name="customize-prompts-using-the-prompt-attribute"></a>Настройка запросов с помощью атрибута Prompt

Запросы по умолчанию создаются автоматически для каждого поля в вашей форме, но вы можете указать пользовательский запрос для любого поля с помощью атрибута `Prompt`. Например, если для поля `SandwichOrder.Sandwich` запрос по умолчанию: Please select a sandwich (Выберите сэндвич), вы можете добавить атрибут `Prompt`, чтобы указать для этого поля пользовательский запрос.

[!code-csharp[Prompt attribute](../includes/code/dotnet-formflow-advanced.cs#promptAttribute)]

В этом примере для динамического заполнения запроса данными формы в среде выполнения используется [язык общих схем решений](bot-builder-dotnet-formflow-pattern-language.md): `{&}` заменяется описанием поля, а `{||}` — списком вариантов в перечислении. 

> [!NOTE]
> По умолчанию описанием поля является имя поля. Чтобы указать пользовательское описание для поля, добавьте атрибут `Describe`.

В этом фрагменте кода показан настроенный запрос, указанный в примере выше.

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

Атрибут `Prompt` может также указывать параметры, влияющие на отображение формы в запросе. Например, параметр `ChoiceFormat` определяет, как форма преобразовывает для просмотра список вариантов.

[!code-csharp[Prompt attribute ChoiceFormat parameter](../includes/code/dotnet-formflow-advanced.cs#promptChoice)]

В этом примере значение параметра `ChoiceFormat` указывает, что выбор должен отображаться как маркированный список (вместо нумерованного списка).

```console
What kind of sandwich would you like?
- BLT
- Black Forest Ham
- Buffalo Chicken
- Chicken And Bacon Ranch Melt
- Cold Cut Combo
- Meatball Marinara
- Oven Roasted Chicken
- Roast Beef
- Rotisserie Style Chicken
- Spicy Italian
- Steak And Cheese
- Sweet Onion Teriyaki
- Tuna
- Turkey Breast
- Veggie
>
```

## <a name="customize-prompts-using-the-template-attribute"></a>Настройка запроса с помощью атрибута Template

Хотя атрибут `Prompt` позволяет настроить запрос для одного поля, атрибут `Template` позволяет вам заменить шаблоны по умолчанию, используемые FormFlow для автоматического создания запросов. В этом примере кода используется атрибут `Template`, чтобы переопределить, как форма обрабатывает все поля перечисления. Атрибут указывает, что пользователь может выбрать только один элемент, задает текст запроса с помощью [языка общих схем решений](bot-builder-dotnet-formflow-pattern-language.md) и указывает, что форма должна отображать только один элемент в строке. 

[!code-csharp[Template attribute](../includes/code/dotnet-formflow-advanced.cs#templateAttribute)]

В этом фрагменте показаны готовые запросы для полей `Bread` и `Cheese`.

```console
What kind of bread would you like on your sandwich?
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> 

What kind of cheese would you like on your sandwich? 
 1. American
 2. Monterey Cheddar
 3. Pepperjack
> 
```

Если вы используете атрибут `Template` для замены шаблонов по умолчанию, с помощью которых FormFlow создает запросы, вы можете вставить некоторые изменения в запросы и сообщения, создаваемые формой. Для этого вы можете определить несколько текстовых строк с помощью [языка общих схем решений](bot-builder-dotnet-formflow-pattern-language.md), и форма будет случайным образом выбирать из доступных параметров каждый раз, когда нужно будет отображать запрос или сообщение.

Этот пример кода переопределяет шаблон [TemplateUsage.NotUnderstood][notUnderstood], чтобы указать два разных варианта сообщения. Когда боту необходимо сообщить, что он не понимает вводные данные пользователя, он определит содержимое сообщения, случайным образом выбрав одну из двух текстовых строк. 

[!code-csharp[Template variations of message](../includes/code/dotnet-formflow-advanced.cs#templateMessages)]

В этом фрагменте показан пример взаимодействия между ботом и пользователем. 

```console
What size of sandwich do you want? (1. Six Inch, 2. Foot Long)
> two feet
I do not understand "two feet".
> two feet
Try again, I don't get "two feet"
> 
```

## <a name="designate-a-field-as-optional-using-the-optional-attribute"></a>Назначение поля как необязательного с помощью атрибута Optional

Чтобы указать поле необязательным, используйте атрибут `Optional`. В этом примере кода показано, что поле `Cheese` не является обязательным.

[!code-csharp[Optional attribute](../includes/code/dotnet-formflow-advanced.cs#optionalAttribute)]

Если поле является необязательным и значение не указано, текущий выбор будет отображаться как No Preference (Нет предпочтений).

```console
What kind of cheese would you like on your sandwich? (current choice: No Preference)
  1. American
  2. Monterey Cheddar
  3. Pepperjack
 >
```

Если поле является необязательным и пользователь указал значение, значение No Preference (Нет предпочтений) будет отображаться в качестве последнего выбора в списке.

```console
What kind of cheese would you like on your sandwich? (current choice: American)
 1. American
 2. Monterey Cheddar
 3. Pepperjack
 4. No Preference
>
```

## <a name="match-user-input-using-the-terms-attribute"></a>Сопоставление входных данных пользователя с помощью атрибута Terms

Когда пользователь отправляет сообщение боту, который создан с помощью FormFlow, бот пытается определить смысл входных данных пользователя, сопоставляя их со списком терминов. По умолчанию список терминов создается путем применения к полю или значению следующих шагов: 

1. Перерыв в случае изменений регистра и символа подчеркивания (_).
2. Создание каждого значения <a href="https://en.wikipedia.org/wiki/N-gram" target="_blank">N-грамма</a> максимальной длины.
3. Добавление "s?" в конце каждого слова (для поддержки множественного числа). 

Например, значение AngusBeefAndGarlicPizza создаст следующие термины: 

- angus? (ангус?)
- beefs? (говядина?)
- garlics? (чеснок?)
- пицца? (пицца?)
- angus? beefs? (ангус? говядина?)
- garlics? pizzas? (чеснок? пицца?)
- angus beef and garlic pizza (говядина ангус и пицца с чесноком)

Чтобы переопределить это поведение по умолчанию и определить список терминов, которые используются для сопоставления входных данных пользователя с полем или значением в поле, используйте атрибут `Terms`. Например, вы можете использовать атрибут `Terms` (с регулярным выражением), чтобы учесть тот факт, что пользователи могут ошибочно ввести слово rotisserie. 

[!code-csharp[Terms attribute](../includes/code/dotnet-formflow-advanced.cs#termsAttribute)]

Используя атрибут `Terms`, вы увеличиваете вероятность сопоставления входных данных пользователя с одним из допустимых вариантов. Параметр `Terms.MaxPhrase` в этом примере заставляет `Language.GenerateTerms` создавать дополнительные вариации терминов. 

В этом фрагменте показано результирующее взаимодействие между ботом и пользователем, когда пользователь неверно указывает слово Rotisserie.

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
> rotissary checkin
For sandwich I understood Rotisserie Style Chicken. "checkin" is not an option.
```

## <a name="validate-user-input-using-the-numeric-attribute-or-pattern-attribute"></a>Проверка входных данных пользователя с помощью атрибутов Numeric или Pattern

Чтобы ограничить диапазон допустимых значений для числового поля, используйте атрибут `Numeric`. В этом примере кода с помощью атрибута `Numeric` указывается, что в поле `Rating` необходимо ввести число от 1 до 5. 

[!code-csharp[Numeric attribute](../includes/code/dotnet-formflow-advanced.cs#numericAttribute)]

Чтобы указать необходимый формат для значения определенного поля, используйте атрибут `Pattern`. В этом примере кода с помощью атрибута `Pattern` указан необходимый формат для значения поля `PhoneNumber`.

[!code-csharp[Pattern attribute](../includes/code/dotnet-formflow-advanced.cs#patternAttribute)]

## <a name="summary"></a>Сводка

В этой статье описывается, как настроить взаимодействие с пользователем с помощью FormFlow, указав начальное состояние формы, добавив бизнес-логику для управления взаимозависимостью между полями и обработкой входных данных с помощью атрибутов для настройки запросов, переопределения шаблонов, назначения необязательных полей, сопоставления и проверки входных данных пользователя ввод. Сведения о дополнительных способах настройки взаимодействия с пользователем с помощью FormFlow см. в статье [Customize a form using FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) (Настройка формы с помощью FormBuilder).

## <a name="sample-code"></a>Образец кода

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные функции FormFlow](bot-builder-dotnet-formflow.md)
- [Настройка формы с помощью FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Локализация содержимого формы](bot-builder-dotnet-formflow-localize.md)
- [Определение формы с помощью схемы JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Customize user experience with pattern language](bot-builder-dotnet-formflow-pattern-language.md) (Настройка взаимодействия с помощью языка шаблонов)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>

[formDialog]: /dotnet/api/microsoft.bot.builder.formflow.formdialog

[promptFieldsWithValues]: /dotnet/api/microsoft.bot.builder.formflow.formoptions.promptfieldswithvalues

[validateResult]: /dotnet/api/microsoft.bot.builder.formflow.validateresult

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[optionalAttribute]: /dotnet/api/microsoft.bot.builder.formflow.optionalattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[termsAttribute]: /dotnet/api/microsoft.bot.builder.formflow.termsattribute

[notUnderstood]: /dotnet/api/microsoft.bot.builder.formflow.templateusage.notunderstood
