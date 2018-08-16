---
title: Определение формы с помощью схемы JSON и FormFlow | Документация Майкрософт
description: Узнайте, как определить форму с помощью схемы JSON и FormFlow, используя пакет SDK Bot Builder для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a7f6e3f186e0c4b9f6096cad72a91ef6f3fdffd4
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306095"
---
# <a name="define-a-form-using-json-schema"></a>Определение формы с помощью схемы JSON

Если при создании бота с помощью FormFlow вы используете [класс C#](bot-builder-dotnet-formflow.md#create-class) для определения формы, то эта форма является производной от статического определения типа в C#. Кроме того, вы можете определить форму с помощью <a href="http://json-schema.org/documentation.html" target="_blank">схемы JSON</a>. Формой, которая определена с помощью схемы JSON, полностью управляют данные. Ее (и, следовательно, поведение бота) можно изменить, просто изменив схему. 

Схема JSON описывает поля в <a href="http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm" target="_blank">JObject</a> и содержит заметки, которые управляют запросами, шаблонами и терминами. Чтобы использовать схему JSON с FormFlow, необходимо добавить пакет NuGet `Microsoft.Bot.Builder.FormFlow.Json` в проект и импортировать пространство имен `Microsoft.Bot.Builder.FormFlow.Json`.

## <a name="standard-keywords"></a>Стандартные ключевые слова 

FormFlow поддерживает приведенные ниже ключевые слова стандартной <a href="http://json-schema.org/documentation.html" target="_blank">схемы JSON</a>.

| Ключевое слово | ОПИСАНИЕ | 
|----|----|
| Тип | Определяет тип данных в поле. |
| enum | Определяет допустимые значения для поля. |
| minimum | Определяет минимальное числовое значение поля (как описано в [NumericAttribute][numericAttribute]). |
| maximum | Определяет максимальное числовое значение поля (как описано в [NumericAttribute][numericAttribute]). |
| обязательно | Определяет, какие поля являются обязательными. |
| pattern | Проверяет строковые значения (как описано в [PatternAttribute][patternAttribute]). |

## <a name="extensions-to-json-schema"></a>Расширения схемы JSON

FormFlow расширяет стандартную <a href="http://json-schema.org/documentation.html" target="_blank">схему JSON</a> для поддержки нескольких дополнительных свойств.

### <a name="additional-properties-at-the-root-of-the-schema"></a>Дополнительные свойства в корне схемы

| Свойство | Значение |
|----|----|
| OnCompletion | Сценарий C# с аргументами `(IDialogContext context, JObject state)` для заполнения формы. |
| Ссылки | Ссылки, добавляемые в сценарий. Например, `[assemblyReference, ...]`. Пути должны быть абсолютными или относительными для текущего каталога. По умолчанию сценарий включает в себя `Microsoft.Bot.Builder.dll`. |
| Импорт | Элементы, импортируемые в сценарий. Например, `[import, ...]`. По умолчанию в сценарий добавляются элементы `Microsoft.Bot.Builder`, `Microsoft.Bot.Builder.Dialogs`, `Microsoft.Bot.Builder.FormFlow`, `Microsoft.Bot.Builder.FormFlow.Advanced`, `System.Collections.Generic` и пространства имен `System.Linq`. |

### <a name="additional-properties-at-the-root-of-the-schema-or-as-peers-of-the-type-property"></a>Дополнительные свойства в корне схемы или свойства, аналогичные свойству type

| Свойство | Значение |
|----|----|
| Шаблоны | `{ TemplateUsage: { Patterns: [string, ...], <args> }, ...}` |
| prompt | `{ Patterns:[string, ...] <args>}` |

Чтобы указать шаблоны и запросы в схеме JSON, используйте словарь, который определен в [TemplateAttribute][templateAttribute] и [PromptAttribute][promptAttribute]. Имена и значения свойств в схеме должны соответствовать именам и значениям свойств в базовом перечислении C#. Например, этот фрагмент схемы определяет шаблон, который переопределяет шаблон `TemplateUsage.NotUnderstood` и указывает `TemplateBaseAttribute.ChoiceStyle`. 

```json
"Templates":{ "NotUnderstood": { "Patterns": ["I don't get it"], "ChoiceStyle":"Auto"}}
```

### <a name="additional-properties-as-peers-of-the-type-property"></a>Дополнительные свойства, аналогичные свойству type

|   Свойство   |          Оглавление           |                                                   ОПИСАНИЕ                                                    |
|--------------|-----------------------------|------------------------------------------------------------------------------------------------------------------|
|   Datetime   |            bool             |                                  Указывает, является ли поле полем `DateTime`.                                  |
|   Describe   |      Строка или объект       |                  Описание поля, как описано в [DescribeAttribute][describeAttribute].                  |
|    Термины     |       `[string,...]`        |                  Регулярные выражения для сопоставления значения поля, как описано в TermsAttribute.                  |
|  MaxPhrase   |             int             |                  Обрабатывает ваши термины с помощью `Language.GenerateTerms(string, int)`, чтобы дополнить их.                   |
|    Значения    | { string: {Describe:string |                                  object, Terms:[string, ...], MaxPhrase}, ...}                                  |
|    Активна    |           script            | Сценарий C# с аргументами `(JObject state)->bool` для проверки, является ли поле, сообщение или подтверждение активным.  |
|   Проверка   |           script            |      Сценарий C# с аргументами `(JObject state, object value)->ValidateResult` для проверки значения поля.      |
|    Define    |           script            |        Сценарий C# с аргументами `(JObject state, Field<JObject> field)` для динамического определения поля.        |
|     Далее     |           script            | Сценарий C# с аргументами `(object value, JObject state)` для определения следующего шага после заполнения поля. |
|    До    |          [confirm          |                                                  message, ...]                                                  |
|    после     |          [confirm          |                                                  message, ...]                                                  |
| Зависимости |        [string, ...]        |                           Поля, от которых зависит это поле, сообщения или подтверждение.                           |

Укажите `{Confirm:script|[string, ...], ...templateArgs}` в значении свойства **Before** или **After**, чтобы определить подтверждение с помощью сценария C# с аргументом `(JObject state)` или набора шаблонов, которые будут выбираться случайным образом и использоваться с необязательными аргументами шаблона.

Укажите `{Message:script|[string, ...] ...templateArgs}` в значении свойства **Before** или **After**, чтобы определить сообщение с помощью сценария C# с аргументом `(JObject state)` или набора шаблонов, которые будут выбираться случайным образом и использоваться с необязательными аргументами шаблона.

## <a name="scripts"></a>Сценарии

Некоторые из свойств, описанных выше, содержат сценарий в качестве значения. Сценарием может быть любой фрагмент кода C#, который обычно может быть определен в методе. Можно добавить ссылки с помощью свойства **References** и (или) свойства **Imports**. Ниже перечислены специальные глобальные переменные.

| Переменная | ОПИСАНИЕ |
|----|----|
| choice | Внутренняя диспетчеризации для выполнения сценария. |
| state | Состояние формы `JObject` для всех сценариев. |
| ifield | Объект `IField<JObject>`, разрешающий обоснование по текущему полю для всех сценариев, за исключением построителей запросов сообщения и подтверждения. |
| value | Проверяемое значение свойства **Validate**. |
| поле | Объект `Field<JObject>`, разрешающий динамическое обновление поля в свойстве **Define**. |
| context | Контекст `IDialogContext`, разрешающий отправку результатов в **OnCompletion**. |

Поля, определенные с помощью схемы JSON, обеспечивают те же возможности программного расширения и переопределения определений, как и любое другое поле. Они локализуется таким же образом.

## <a name="json-schema-example"></a>Пример схемы JSON

Самый простой способ определить форму — определить все, включая код C#, непосредственно в схеме JSON. В этом примере показана схема JSON для бота для заказа сандвичей с заметками, описанного в разделе [Настройка формы с помощью FormBuilder](bot-builder-dotnet-formflow-formbuilder.md).

```json
{
  "References": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.dll" ],
  "Imports": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.Resource" ],
  "type": "object",
  "required": [
    "Sandwich",
    "Length",
    "Ingredients",
    "DeliveryAddress"
  ],
  "Templates": {
    "NotUnderstood": {
      "Patterns": [ "I do not understand \"{0}\".", "Try again, I don't get \"{0}\"." ]
    },
    "EnumSelectOne": {
      "Patterns": [ "What kind of {&} would you like on your sandwich? {||}" ],
      "ChoiceStyle": "Auto"
    }
  },
  "properties": {
    "Sandwich": {
      "Prompt": { "Patterns": [ "What kind of {&} would you like? {||}" ] },
      "Before": [ { "Message": [ "Welcome to the sandwich order bot!" ] } ],
      "Describe": { "Image": "https://placeholdit.imgix.net/~text?txtsize=16&txt=Sandwich&w=125&h=40&txttrack=0&txtclr=000&txtfont=bold" },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "BLT",
        "BlackForestHam",
        "BuffaloChicken",
        "ChickenAndBaconRanchMelt",
        "ColdCutCombo",
        "MeatballMarinara",
        "OvenRoastedChicken",
        "RoastBeef",
        "RotisserieStyleChicken",
        "SpicyItalian",
        "SteakAndCheese",
        "SweetOnionTeriyaki",
        "Tuna",
        "TurkeyBreast",
        "Veggie"
      ],
      "Values": {
        "RotisserieStyleChicken": {
          "Terms": [ "rotis\\w* style chicken" ],
          "MaxPhrase": 3
        }
      }
    },
    "Length": {
      "Prompt": {
        "Patterns": [ "What size of sandwich do you want? {||}" ]
      },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "SixInch",
        "FootLong"
      ]
    },
    "Ingredients": {
      "type": "object",
      "required": [ "Bread" ],
      "properties": {
        "Bread": {
          "type": [
            "string",
            "null"
          ],
          "Describe": {
            "Title": "Sandwich Bot",
            "SubTitle": "Bread Picker"
          },
          "enum": [
            "NineGrainWheat",
            "NineGrainHoneyOat",
            "Italian",
            "ItalianHerbsAndCheese",
            "Flatbread"
          ]
        },
        "Cheese": {
          "type": [
            "string",
            "null"
          ],
          "enum": [
            "American",
            "MontereyCheddar",
            "Pepperjack"
          ]
        },
        "Toppings": {
          "type": "array",
          "items": {
            "type": "integer",
            "enum": [
              "Everything",
              "Avocado",
              "BananaPeppers",
              "Cucumbers",
              "GreenBellPeppers",
              "Jalapenos",
              "Lettuce",
              "Olives",
              "Pickles",
              "RedOnion",
              "Spinach",
              "Tomatoes"
            ],
            "Values": {
              "Everything": { "Terms": [ "except", "but", "not", "no", "all", "everything" ] }
            }
          },
          "Validate": "var values = ((List<object>) value).OfType<string>(); var result = new ValidateResult {IsValid = true, Value = values} ; if (values != null && values.Contains(\"Everything\")) { result.Value = (from topping in new string[] {  \"Avocado\", \"BananaPeppers\", \"Cucumbers\", \"GreenBellPeppers\", \"Jalapenos\", \"Lettuce\", \"Olives\", \"Pickles\", \"RedOnion\", \"Spinach\", \"Tomatoes\"} where !values.Contains(topping) select topping).ToList();} return result;",
          "After": [ { "Message": [ "For sandwich toppings you have selected {Ingredients.Toppings}." ] } ]
        },
        "Sauces": {
          "type": [
            "array",
            "null"
          ],
          "items": {
            "type": "string",
            "enum": [
              "ChipotleSouthwest",
              "HoneyMustard",
              "LightMayonnaise",
              "RegularMayonnaise",
              "Mustard",
              "Oil",
              "Pepper",
              "Ranch",
              "SweetOnion",
              "Vinegar"
            ]
          }
        }
      }
    },
    "Specials": {
      "Templates": {
        "NoPreference": { "Patterns": [ "None" ] }
      },
      "type": [
        "string",
        "null"
      ],
      "Active": "return (string) state[\"Length\"] == \"FootLong\";",
      "Define": "field.SetType(null).AddDescription(\"cookie\", DynamicSandwich.FreeCookie).AddTerms(\"cookie\", Language.GenerateTerms(DynamicSandwich.FreeCookie, 2)).AddDescription(\"drink\", DynamicSandwich.FreeDrink).AddTerms(\"drink\", Language.GenerateTerms(DynamicSandwich.FreeDrink, 2)); return true;",
      "After": [ { "Confirm": "var cost = 0.0; switch ((string) state[\"Length\"]) { case \"SixInch\": cost = 5.0; break; case \"FootLong\": cost=6.50; break;} return new PromptAttribute($\"Total for your sandwich is {cost:C2} is that ok?\");" } ]
    },
    "DeliveryAddress": {
      "type": [
        "string",
        "null"
      ],
      "Validate": "var result = new ValidateResult{ IsValid = true, Value = value}; var address = (value as string).Trim(); if (address.Length > 0 && (address[0] < '0' || address[0] > '9')) {result.Feedback = DynamicSandwich.BadAddress; result.IsValid = false; } return result;"
    },
    "PhoneNumber": {
      "type": [ "string", "null" ],
      "pattern": "(\\(\\d{3}\\))?\\s*\\d{3}(-|\\s*)\\d{4}"
    },
    "DeliveryTime": {
      "Templates": {
        "StatusFormat": {
          "Patterns": [ "{&}: {:t}" ],
          "FieldCase": "None"
        }
      },
      "DateTime": true,
      "type": [
        "string",
        "null"
      ],
      "After": [ { "Confirm": [ "Do you want to order your {Length} {Sandwich} on {Ingredients.Bread} {&Ingredients.Bread} with {[{Ingredients.Cheese} {Ingredients.Toppings} {Ingredients.Sauces} to be sent to {DeliveryAddress} {?at {DeliveryTime}}?" ] } ]
    },
    "Rating": {
      "Describe": "your experience today",
      "type": [
        "number",
        "null"
      ],
      "minimum": 1,
      "maximum": 5,
      "After": [ { "Message": [ "Thanks for ordering your sandwich!" ] } ]
    }
  },
  "OnCompletion": "await context.PostAsync(\"We are currently processing your sandwich. We will message you the status.\");"
}
```

## <a name="implement-formflow-with-json-schema"></a>Реализация FormFlow со схемой JSON

Чтобы реализовать FormFlow со схемой JSON, используйте `FormBuilderJson`, который поддерживает тот же текучий интерфейс, что и `FormBuilder`. В этом примере кода показан, как реализовать схему JSON для бота для заказа сандвичей с заметками, описанного в разделе [Настройка формы с помощью FormBuilder](bot-builder-dotnet-formflow-formbuilder.md).

[!code-csharp[Use JSON schema](../includes/code/dotnet-formflow-json-schema.cs#useSchema)]

## <a name="sample-code"></a>Пример кода

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные функции FormFlow](bot-builder-dotnet-formflow.md)
- [Дополнительные функции FormFlow](bot-builder-dotnet-formflow-advanced.md)
- [Настройка формы с помощью FormBuilder](bot-builder-dotnet-formflow-formbuilder.md)
- [Локализация содержимого формы](bot-builder-dotnet-formflow-localize.md)
- [Настройка взаимодействия с пользователем с помощью языка шаблонов](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочник по пакету SDK Bot Builder для .NET</a>

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute
