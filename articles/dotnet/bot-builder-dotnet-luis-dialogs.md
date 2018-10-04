---
title: Распознавание намерений и сущностей с помощью LUIS | Документы Майкрософт
description: Узнайте, как "научить" бот понимать естественный язык с помощью диалогов LUIS в пакете SDK Bot Builder для .NET.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f95335149fa2c896d905834832089ffbfa960bf2
ms.sourcegitcommit: d4afc924b0e1907c4d6f7a6fc5ac1fe521aeef7e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47447395"
---
# <a name="recognize-intents-and-entities-with-luis"></a>Распознавание намерений и сущностей с помощью LUIS 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В этой статье используется пример бота для ведения заметок. Этот пример показывает, как реализовать естественную реакцию бота на входные данные на естественном языке с помощью распознавания речи ([LUIS][LUIS]). Бот понимает, что хочет сделать пользователь, определяя **намерение** пользователя. Это намерение определяется на основе голосовых или текстовых входных данных, или **высказываний**. Намерение сопоставляет высказывания с действиями, которые выполняет бот. Например, бот для заметок распознает намерение `Notes.Create` для вызова функции создания заметки. Боту также может потребоваться извлекать **сущности** — важные слова в высказываниях. Для бота, который ведет заметки, сущность `Notes.Title` определяет заголовок каждой заметки.

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Создание бота для распознавания речи с помощью службы Azure Bot

1. На [портале Azure](https://portal.azure.com) в колонке меню выберите **Создать ресурс**, а затем **Показать все**.

    ![Создание ресурса](../media/bot-builder-dotnet-use-luis/bot-service-creation.png)

2. В поле поиска введите **Бот веб-приложения**. 

    ![Создание ресурса](../media/bot-builder-dotnet-use-luis/bot-service-selection.png)

3. В колонке **Служба ботов** введите необходимые сведения и нажмите кнопку **Создать**. В Azure будут созданы и развернуты служба ботов и приложение LUIS. 
   * В поле **Имя приложения** укажите имя бота. При развертывании бота в облаке имя используется в качестве поддомена (например, mynotesbot.azurewebsites.net). Это имя также используется как имя приложения LUIS, связанного с ботом. Скопируйте его для последующего использования. Оно поможет найти приложение LUIS, связанное с ботом.
   * Заполните поля "Подписка", [Группа ресурсов](/azure/azure-resource-manager/resource-group-overview), "План службы приложений" и [Расположение](https://azure.microsoft.com/en-us/regions/).
   * В поле **Шаблон бота** выберите шаблон **Распознавание речи (C#)**.

     ![Колонка "Служба ботов"](../media/bot-builder-dotnet-use-luis/bot-service-setting-callout-template.png)

   * Установите этот флажок, чтобы подтвердить условия предоставления услуг.

4. Убедитесь, что служба Bot Service развернута.
    * Щелкните "Notifications" (Уведомления) (значок колокольчика, расположенный в верхней части портала Azure). Уведомление изменится с **Развертывание начато** на **Развертывание выполнено**.
    * После того как уведомление изменится на **Развертывание прошло успешно**, в этом уведомлении щелкните **Go to resource** (Перейти к ресурсу).

## <a name="try-the-bot"></a>Проверка работы бота

Убедитесь, что бот был развернут, проверив раздел **Уведомления**. Уведомление изменится с **Развертывание выполняется...** на **Развертывание выполнено**. Чтобы открыть колонку ресурсов бота, нажмите кнопку **Перейти к ресурсу**.

После регистрации бота выберите пункт **Test in Web Chat** (Тестировать в веб-чате), чтобы открыть панель веб-чата. В веб-чате введите "hello" (привет).

  ![Тестирование бота в веб-чате](../media/bot-builder-dotnet-use-luis/bot-service-web-chat.png)

Бот отвечает: "You have reached Greeting. You said: hello" (Вы находитесь на этапе приветствия. Вы сказали: привет). Этот ответ подтверждает, что бот получил ваше сообщение и передал его в созданное приложение LUIS по умолчанию. Это приложение LUIS по умолчанию обнаружило намерение приветствия.

## <a name="modify-the-luis-app"></a>Изменение приложения LUIS

Войдите в [https://www.luis.ai](https://www.luis.ai) с той же учетной записью, которая используется для входа Azure. Щелкните **Мои приложения**. В списке приложений найдите приложение, которое начинается с имени, указанного в поле **Имя приложения** в колонке **Служба ботов** при создании службы ботов. 

Изначально приложение LUIS содержит 4 намерения: Cancel, Greeting, Help и None. <!-- picture -->

Выполнив следующие действия, вы сможете добавить намерения Note.Create, Note.ReadAloud и Note.Delete: 

1. Щелкните **Предварительно созданные домены** в левом нижнем углу страницы. Найдите домен **Note** и нажмите кнопку **Добавить домен**.

2. В этом учебнике используются не все намерения, включенные в предварительно созданный домен **Note**. На странице **Намерения** щелкните каждое из указанных имен намерений и нажмите кнопку **Удалить намерение**.
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   Ниже перечислены только те намерения, которые должны остаться в приложении LUIS: 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * None
   * Справка
   * Greeting
   * Отмена 

     ![Намерения, отображаемые в приложении LUIS](../media/bot-builder-dotnet-use-luis/luis-intent-list.png)

3. Нажмите кнопку **Обучить** в правом верхнем углу, чтобы обучить приложение.
4. Нажмите кнопку **Опубликовать** на панели навигации вверху, чтобы открыть страницу **Публикация**. Нажмите кнопку **Опубликовать в рабочем слоте**. После успешной публикации скопируйте URL-адрес, который указан в столбце **Конечная точка** и в строке, которая начинается с имени ресурса Starter_Key, на странице **Публикация приложения**. Сохраните этот URL-адрес для использования в коде бота позже. Формат URL-адреса похож на формат, используемый в следующем примере: `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`.

## <a name="modify-the-bot-code"></a>Изменение кода бота

Выберите пункт **Build** (Сборка) и щелкните ссылку **Open online code editor** (Открыть сетевой редактор кода).
    ![Открытие сетевого редактора кода](../media/bot-builder-dotnet-use-luis/bot-service-build.png)

В редакторе кода откройте `BasicLuisDialog.cs`. Он содержит следующий код для обработки намерений из приложения LUIS.
```cs
using System;
using System.Configuration;
using System.Threading.Tasks;

using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace Microsoft.Bot.Sample.LuisBot
{
    // For more information about this template visit http://aka.ms/azurebots-csharp-luis
    [Serializable]
    public class BasicLuisDialog : LuisDialog<object>
    {
        public BasicLuisDialog() : base(new LuisService(new LuisModelAttribute(
            ConfigurationManager.AppSettings["LuisAppId"], 
            ConfigurationManager.AppSettings["LuisAPIKey"], 
            domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
        {
        }

        [LuisIntent("None")]
        public async Task NoneIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        // Go to https://luis.ai and create a new intent, then train/publish your luis app.
        // Finally replace "Greeting" with the name of your newly created intent in the following handler
        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Cancel")]
        public async Task CancelIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Help")]
        public async Task HelpIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        private async Task ShowLuisResult(IDialogContext context, LuisResult result) 
        {
            await context.PostAsync($"You have reached {result.Intents[0].Intent}. You said: {result.Query}");
            context.Wait(MessageReceived);
        }
    }
}
```
### <a name="create-a-class-for-storing-notes"></a>Создание класса для хранения заметок

Добавьте следующий оператор `using` в файл BasicLuisDialog.cs.

```cs
using System.Collections.Generic;
```

Добавьте следующий код в класс `BasicLuisDialog` после определения конструктора.

```cs
        // Store notes in a dictionary that uses the title as a key
        private readonly Dictionary<string, Note> noteByTitle = new Dictionary<string, Note>();
        
        [Serializable]
        public sealed class Note : IEquatable<Note>
        {

            public string Title { get; set; }
            public string Text { get; set; }

            public override string ToString()
            {
                return $"[{this.Title} : {this.Text}]";
            }

            public bool Equals(Note other)
            {
                return other != null
                    && this.Text == other.Text
                    && this.Title == other.Title;
            }

            public override bool Equals(object other)
            {
                return Equals(other as Note);
            }

            public override int GetHashCode()
            {
                return this.Title.GetHashCode();
            }
        }

        // CONSTANTS        
        // Name of note title entity
        public const string Entity_Note_Title = "Note.Title";
        // Default note title
        public const string DefaultNoteTitle = "default";
```

### <a name="handle-the-notecreate-intent"></a>Обработка намерения Note.Create
Для обработки намерения Note.Create добавьте в класс `BasicLuisDialog` следующий код.

```cs
        private Note noteToCreate;
        private string currentTitle;
        [LuisIntent("Note.Create")]
        public Task NoteCreateIntent(IDialogContext context, LuisResult result)
        {
            EntityRecommendation title;
            if (!result.TryFindEntity(Entity_Note_Title, out title))
            {
                // Prompt the user for a note title
                PromptDialog.Text(context, After_TitlePrompt, "What is the title of the note you want to create?");
            }
            else
            {
                var note = new Note() { Title = title.Entity };
                noteToCreate = this.noteByTitle[note.Title] = note;

                // Prompt the user for what they want to say in the note           
                PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");
            }
            
            return Task.CompletedTask;
        }
        
        
        private async Task After_TitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            EntityRecommendation title;
            // Set the title (used for creation, deletion, and reading)
            currentTitle = await result;
            if (currentTitle != null)
            {
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = currentTitle };
            }
            else
            {
                // Use the default note title
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = DefaultNoteTitle };
            }

            // Create a new note object 
            var note = new Note() { Title = title.Entity };
            // Add the new note to the list of notes and also save it in order to add text to it later
            noteToCreate = this.noteByTitle[note.Title] = note;

            // Prompt the user for what they want to say in the note           
            PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");

        }

        private async Task After_TextPrompt(IDialogContext context, IAwaitable<string> result)
        {
            // Set the text of the note
            noteToCreate.Text = await result;
            
            await context.PostAsync($"Created note **{this.noteToCreate.Title}** that says \"{this.noteToCreate.Text}\".");
            
            context.Wait(MessageReceived);
        }
```

### <a name="handle-the-notereadaloud-intent"></a>Обработка намерения Note.ReadAloud
Бот может использовать намерение `Note.ReadAloud` для отображения содержимого заметки или всех заметок, если заголовок заметки не обнаружен.

Скопируйте приведенный ниже код и вставьте его в класс `BasicLuisDialog`.
```cs
        [LuisIntent("Note.ReadAloud")]
        public async Task NoteReadAloudIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                await context.PostAsync($"**{note.Title}**: {note.Text}.");
            }
            else
            {
                // Print out all the notes if no specific note name was detected
                string NoteList = "Here's the list of all notes: \n\n";
                foreach (KeyValuePair<string, Note> entry in noteByTitle)
                {
                    Note noteInList = entry.Value;
                    NoteList += $"**{noteInList.Title}**: {noteInList.Text}.\n\n";
                }
                await context.PostAsync(NoteList);
            }

            context.Wait(MessageReceived);
        }
        
        public bool TryFindNote(string noteTitle, out Note note)
        {
            bool foundNote = this.noteByTitle.TryGetValue(noteTitle, out note); // TryGetValue returns false if no match is found.
            return foundNote;
        }
        
        public bool TryFindNote(LuisResult result, out Note note)
        {
            note = null;

            string titleToFind;

            EntityRecommendation title;
            if (result.TryFindEntity(Entity_Note_Title, out title))
            {
                titleToFind = title.Entity;
            }
            else
            {
                titleToFind = DefaultNoteTitle;
            }

            return this.noteByTitle.TryGetValue(titleToFind, out note); // TryGetValue returns false if no match is found.
        }
```

### <a name="handle-the-notedelete-intent"></a>Обработка намерения Note.Delete
Скопируйте приведенный ниже код и вставьте его в класс `BasicLuisDialog`.

```cs
        [LuisIntent("Note.Delete")]
        public async Task NoteDeleteIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {                             
                // Prompt the user for a note title
                PromptDialog.Text(context, After_DeleteTitlePrompt, "What is the title of the note you want to delete?");                         
            }           
        }

        private async Task After_DeleteTitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            Note note;
            string titleToDelete = await result;
            bool foundNote = this.noteByTitle.TryGetValue(titleToDelete, out note);

            if (foundNote)
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {
                await context.PostAsync($"Did not find note named {titleToDelete}.");
            }

            context.Wait(MessageReceived);
        }
```

## <a name="build-the-bot"></a>Сборка бота
Щелкните правой кнопкой мыши **build.cmd** в редакторе кода и выберите **Run from Console** (Выполнить из консоли).

   ![Запуск build.cmd](../media/bot-builder-dotnet-use-luis/bot-service-run-console.png)

## <a name="test-the-bot"></a>Тестирование бота

На портале Azure щелкните **Test in Web Chat** (Тестировать в веб-чате), чтобы протестировать бот. Попробуйте ввести сообщения, такие как "Create a note", "read my notes" и "delete notes".
   ![Тестирование бота, выполняющего ведение заметок, в веб-чате](../media/bot-builder-dotnet-use-luis/bot-service-test-notebot.png)

> [!TIP]
> Если ваш бот не всегда распознает правильное намерение или сущности, необходимо улучшить производительность приложения LUIS, предоставив ему больше примеров высказываний для обучения. Вы можете повторно обучить приложение LUIS, не изменяя код бота. См. разделы [Добавление примеров высказываний](/azure/cognitive-services/LUIS/add-example-utterances) и [Обучение и тестирование приложений LUIS](/azure/cognitive-services/LUIS/train-test).

> [!TIP]
> В случае проблем при выполнении кода бота проверьте следующее.
> * Вы выполнили [сборку бота](./bot-builder-dotnet-luis-dialogs.md#build-the-bot).
> * К коде бота определен обработчик для каждого намерения в приложении LUIS.

## <a name="next-steps"></a>Дополнительная информация

Опробовав бот, можно увидеть, как намерение LUIS вызывает задачи. Тем не менее в этом простом примере не допускается прерывание активного диалога. Использование и обработка прерываний (например, для вывода справки или отмены действия) — гибкий механизм, учитывающий реальные действия пользователей. Узнайте больше об использовании диалогов с возможностью оценки, позволяющих обрабатывать прерывания.

> [!div class="nextstepaction"]
> [Глобальные обработчики сообщений, использующие диалоги с возможностью оценки](bot-builder-dotnet-scorable-dialogs.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Диалоги](bot-builder-dotnet-dialogs.md)
- [Управление потоком беседы с помощью диалогов](bot-builder-dotnet-manage-conversation-flow.md)
- <a href="https://www.luis.ai" target="_blank">LUIS</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочник по пакету SDK Bot Builder для .NET</a>

[LUIS]: https://www.luis.ai/
[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample
[NotesSampleJSON]: https://github.com/Microsoft/BotFramework-Samples/blob/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample/Notes.json
