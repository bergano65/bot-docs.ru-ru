---
title: Основные функции FormFlow | Документация Майкрософт
description: Узнайте об управлении потоками беседы с помощью FormFlow из пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 710a8ce315faa02a72eaeb753c44b9b212524ec3
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224679"
---
# <a name="basic-features-of-formflow"></a>Основные функции FormFlow

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[Диалоговые окна](bot-builder-dotnet-dialogs.md) являются очень мощной и гибкой формой интерактивного общения (например, заказ сандвича), хоть и требующей некоторых усилий при обработке. В каждой точке беседы существует множество возможностей того, что произойдет дальше. Например, может потребоваться уточнение неоднозначности, предоставление справки, возврат и отображение хода выполнения задания. Использование **FormFlow** из пакета SDK Bot Builder для .NET позволяет значительно упростить процесс управления интерактивной беседой, как описано далее. 

FormFlow автоматически генерирует диалоговые окна, которые необходимы для управления интерактивной беседой, в зависимости от указанных вами рекомендаций. Хоть при использовании FormFlow "жертвуется" некоторая гибкость, в противовес тому, что вы можете получить путем создания диалоговых окон и управления ими самостоятельно, проектирование интерактивной беседы с использованием FormFlow может значительно снизить время, необходимое для разработки бота. Кроме того, можно создать бот с помощью сочетания диалоговых окон, сформированных FormFlow, и диалоговых окон других типов. Например, диалоговое окно FormFlow может направлять пользователя в процессе заполнения формы, в то время как [LuisDialog][LuisDialog] может оценить входные данные пользователя для определения намерения.

В этой статье описывается, как создать бот, который использует основные функции FormFlow для сбора сведений пользователя.

## <a id="forms-and-fields"></a>Формы и поля

Для создания бота, используя FormFlow, необходимо указать, какую именно информацию должен собирать бот от пользователя. Например, если целью бота является получение заказа на сандвич для пользователя, то необходимо определить форму, содержащую необходимые боту поля данных для выполнения заказа. Формы можно определить, создав класс C#, содержащий один или несколько открытых свойств для представления данных, которые бот будет собирать от пользователя. Каждое свойство должно быть одним из этих типов данных:

- Целочисленный (sbyte, byte, short, ushort, int, uint, long, ulong).
- Число с плавающей запятой (float, double).
- Строка
- Datetime
- Перечисление.
- Список перечислений.

Любой из типов данных может допускать значение NULL, которое можно использовать для моделирования того, что поле не имеет значения. Если поле формы имеет свойство перечисления, не допускающее значение NULL, значение **0** в перечислении представляет **NULL** (т. е. указывает, что поле не имеет значения), то вам следует начать со значения перечислений **1**. FormFlow игнорирует все другие типы свойств и методов.

Для составных объектов необходимо создать форму для класса C# верхнего уровня и другую форму для составного объекта. Вы можете компоновать формы вместе, используя типичную семантику [диалогового окна](bot-builder-dotnet-dialogs.md). Кроме того, возможно определить форму непосредственно путем реализации [Advanced.IField][iField] или используя [Advanced.Field][field] с заполнением словарей внутри него. 

> [!NOTE]
> Форму можно определить с помощью класса C# или схемы JSON. В этой статье описывается, как определить форму с помощью класса C#. Дополнительные сведения об использовании схемы JSON см. в статье [Define a form using JSON schema](bot-builder-dotnet-formflow-json-schema.md) (Определение формы с помощью схемы JSON).

## <a name="simple-sandwich-bot"></a>Простой сандвич-бот

Рассмотрим следующий пример простого сандвич-бота, который принимает информацию о заказе сандвича. 

### <a id="create-class"></a> Создание формы

Класс `SandwichOrder` определяет форму и перечисления параметров для создания сандвича. Класс также включает статический метод `BuildForm`, который использует [FormBuilder][formBuilder] для создания формы и определения простого приветственного сообщения. 

Чтобы использовать FormFlow, сначала необходимо импортировать пространство имен `Microsoft.Bot.Builder.FormFlow`.

[!code-csharp[Define form](../includes/code/dotnet-formflow.cs#defineForm)]

### <a name="connect-the-form-to-the-framework"></a>Подключение формы к платформе 

Для подключения формы к платформе необходимо добавить ее к контроллеру. В этом примере метод `Conversation.SendAsync` вызывает статический метод `MakeRootDialog`, который в свою очередь вызывает метод `FormDialog.FromForm` для создания формы `SandwichOrder`. 

[!code-csharp[Connect form to framework](../includes/code/dotnet-formflow.cs#connectToFramework)]

### <a name="see-it-in-action"></a>Пример применения

Просто определив форму с классом C# и подключив ее к платформе, тем самым вы включили FormFlow с автоматическим управлением общением между ботом и пользователем. Пример взаимодействий, показанный ниже, демонстрирует возможности бота, который создается с помощью основных функций FormFlow. В каждом взаимодействии символ **>** указывает точку, с которой пользователь вводит ответ. 

#### <a name="display-the-first-prompt"></a>Отображение первого запроса 

Эта форма заполняет свойство `SandwichOrder.Sandwich`. Форма автоматически создает запрос "Please select a sandwich", где слово "sandwich" в запросе является производным от имени свойства `Sandwich`. Перечисление `SandwichOptions` определяет варианты, которые предоставляются пользователю, каждое значение перечисления автоматически разбивается на слова в зависимости от изменений в регистре и знаки подчеркивания.

```console
Please select a sandwich
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

#### <a name="provide-guidance"></a>Предоставление рекомендаций

Пользователь в любой момент в беседе может ввести "help", чтобы получить рекомендации по заполнению формы. Например, если пользователь вводит "help" в запросе, бот ответит согласно этим рекомендациям. 

```console
> help
* You are filling in the sandwich field. Possible responses:
* You can enter a number 1-15 or words from the descriptions. (BLT, Black Forest Ham, Buffalo Chicken, Chicken And Bacon Ranch Melt, Cold Cut Combo, Meatball Marinara, Oven Roasted Chicken, Roast Beef, Rotisserie Style Chicken, Spicy Italian, Steak And Cheese, Sweet Onion Teriyaki, Tuna, Turkey Breast, and Veggie)
* Back: Go back to the previous question.
* Help: Show the kinds of responses you can enter.
* Quit: Quit the form without completing it.
* Reset: Start over filling in the form. (With defaults from your previous entries.)
* Status: Show your progress in filling in the form so far.
* You can switch to another field by entering its name. (Sandwich, Length, Bread, Cheese, Toppings, and Sauce).
```

#### <a name="advance-to-the-next-prompt"></a>Переход к следующему запросу

Если пользователь вводит "2" в ответ на начальной запрос, бот затем отображает запрос для следующего свойства, которое определено формой: `SandwichOrder.Length`.

```console
Please select a sandwich
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
> 2
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="return-to-the-previous-prompt"></a>Возврат к предыдущему запросу 

Если пользователь вводит "back" на этом месте в беседе, бот вернет предыдущий запрос. Запрос отображает текущий выбор пользователя ("Black Forest Ham"); пользователь может изменить свой выбор, указав другой номер, или подтвердить выбор, введя "c".

```console
> back
Please select a sandwich(current choice: Black Forest Ham)
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
> c
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="clarify-user-input"></a>Уточнение входных данных пользователя

Если пользователь для указания выбора отвечает текстом (а не числом), бот автоматически запросит разъяснение, если входные данные пользователя соответствуют нескольким вариантам выбора. 

```console
Please select a bread
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> nine grain
By "nine grain" bread did you mean (1. Nine Grain Honey Oat, 2. Nine Grain Wheat)
> 1
```

Если входные данные пользователя не соответствуют непосредственно ни одному из допустимых вариантов выбора, бот автоматически запросит разъяснение у пользователя.

```console
Please select a cheese (1. American, 2. Monterey Cheddar, 3. Pepperjack)
> amercan
"amercan" is not a cheese option.
> american smoked
For cheese I understood American. "smoked" is not an option.
```

Если входные данные пользователя указывают несколько вариантов для свойства и бот не распознает ни один из них, то он автоматически запросит разъяснение у пользователя.

```console
Please select one or more toppings
 1. Banana Peppers
 2. Cucumbers
 3. Green Bell Peppers
 4. Jalapenos
 5. Lettuce
 6. Olives
 7. Pickles
 8. Red Onion
 9. Spinach
 10. Tomatoes
> peppers, lettuce and tomato
By "peppers" toppings did you mean (1. Green Bell Peppers, 2. Banana Peppers)
> 1
```

#### <a name="show-current-status"></a>Отображение текущего состояния

Если пользователь вводит "status" в любом месте заказа, в ответе бота указываются значения, которые уже заданы и которые остается указать. 

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> status
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Unspecified  
```

#### <a name="confirm-selections"></a>Подтверждение выбора

Когда пользователь завершил заполнение формы, бот попросит его подтвердить свой выбор.

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> 1
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
>
```

Если пользователь отвечает "no", то бот позволяет пользователю изменить любой из предыдущих выборов. Если пользователь отвечает "yes", то работа с формой завершается и управление возвращается к вызванному диалоговому окну. 

```console
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> no
What do you want to change?
 1. Sandwich(Black Forest Ham)
 2. Length(Six Inch)
 3. Bread(Nine Grain Honey Oat)
 4. Cheese(American)
 5. Toppings(Lettuce, Tomatoes, and Green Bell Peppers)
 6. Sauce(Honey Mustard)
> 2
Please select a length (current choice: Six Inch) (1. Six Inch, 2. Foot Long)
> 2
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Foot Long
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> y
```

## <a name="handling-quit-and-exceptions"></a>Обработка выхода из программы и исключений

Если пользователь вводит в форме "quit" или возникает исключение в определенной точке беседы, вашему боту необходимо знать, на каком шаге произошло это событие, какое состояние формы в этот момент и какие шаги формы были успешно выполнены до этого события. Форма возвращает эти сведения с помощью класса `FormCanceledException<T>`. 

В данном примере кода показано, как перехватить исключение и вывести сообщение в соответствии с произошедшим событием. 

[!code-csharp[Handle exception or quit](../includes/code/dotnet-formflow.cs#handleExceptionOrQuit)]

## <a name="summary"></a>Сводка

В данной статье описывается, как использовать основные функции FormFlow для создания чат-бота, который может:

- Автоматически создавать беседу и управлять ею.
- Предоставлять четкие рекомендации и справку.
- Распознавать как числа, так и текст.
- Предоставлять пользователю обратную связь в соответствии с тем, что распознано, а что нет. 
- Задавать уточняющие вопросы при необходимости. 
- Разрешить пользователю переходить между шагами.

Несмотря на то что в некоторых случаях достаточно основного функционала FormFlow, рассмотрите потенциальные преимущества внедрения в бот более сложных функций FormFlow. Дополнительные сведения см. в статьях [Расширенные функции FormFlow](bot-builder-dotnet-formflow-advanced.md) и [Customize a form using FormBuilder](bot-builder-dotnet-formflow-formbuilder.md) (Настройка формы с помощью FormBuilder).

## <a name="sample-code"></a>Пример кода

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="next-steps"></a>Дополнительная информация

FormFlow упрощает разработку диалогового окна. Дополнительные функции FormFlow позволяют настраивать поведение объекта FormFlow.

> [!div class="nextstepaction"]
> [Дополнительные функции FormFlow](bot-builder-dotnet-formflow-advanced.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Настройка формы с помощью FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Локализация содержимого формы](bot-builder-dotnet-formflow-localize.md)
- [Определение формы с помощью схемы JSON](bot-builder-dotnet-formflow-json-schema.md)
- [Customize user experience with pattern language](bot-builder-dotnet-formflow-pattern-language.md) (Настройка взаимодействия с помощью языка шаблонов)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>

[LuisDialog]: /dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1
