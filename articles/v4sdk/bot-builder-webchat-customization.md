---
title: Настройка веб-чата — Служба Azure Bot
description: Дополнительные сведения о настройке веб-чата Bot Framework.
keywords: bot framework, webchat, chat, samples, react, reference
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/07/2019
ms.openlocfilehash: 5016c5810d2c3623d8e8556b0a96d717ee83000d
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791166"
---
# <a name="web-chat-customization"></a>Настройка веб-чата

В этой статье подробно описано, как настроить примеры веб-чата в соответствии с требованиями бота.

## <a name="integrate-web-chat-into-your-website"></a>Интеграция веб-чата в веб-сайт

Следуйте инструкциям на [обзорной странице](bot-builder-webchat-overview.md) для интеграции элемента управления "Веб-чат" в веб-сайт.

## <a name="customizing-styles"></a>Настройка стилей

Последняя версия элемента управления "Веб-чат" предоставляет широкие возможности настройки: вы можете изменять цвета, размеры, размещение элементов, добавлять пользовательские элементы и взаимодействовать с веб-страницей для размещения. Ниже приведены несколько примеров того, как настраивать эти элементы пользовательского интерфейса веб-чата.

Полный список всех параметров, которые можно легко изменять в веб-чате, см. в файле [`defaultStyleOptions.js`](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js).

Эти параметры создадут _набор стилей_ — набор правил CSS, дополненный [glamor](https://github.com/threepointone/glamor). Вы можете найти полный список стилей CSS, созданных в наборе стилей в [`createStyleSet.js` файле](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/createStyleSet.js).

## <a name="set-the-size-of-the-web-chat-container"></a>Установка размера контейнера веб-чата

Теперь можно настроить размер контейнера веб-чата с помощью `styleSetOptions`. В следующем примере есть цвет фона `body` — `paleturquoise` — для отображения контейнера веб-чата (раздел на белом фоне).

```js
…
<head>
  <style>
    html, body { height: 100% }
    body {
      margin: 0;
      background-color: paleturquoise;
    }

    #webchat {
      height: 100%;
      width: 100%;
    }
  </style>
</head>
<body>
  <div id="webchat" role="main"></div>
  <script>
    (async function () {
    window.WebChat.renderWebChat({
      directLine: window.WebChat.createDirectLine({ token }),
      styleOptions: {
        rootHeight: '100%',
        rootWidth: '50%'
      }
    }, document.getElementById('webchat'));
    })()
  </script>
…
```

Результат:

<img alt="Web Chat with root height and root width set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/rootHeightWidth.png" width="600"/>

## <a name="change-font-or-color"></a>Изменение шрифта или цвета

Вместо использования цвета фона по умолчанию и шрифтов, используемых внутри пузырьков чата, вы можете настроить их в соответствии со стилем целевой веб-страницы. Приведенный ниже фрагмент кода позволяет изменить цвет фона сообщений от пользователя и от бота.

<img alt="Screenshot with custom style options" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-options.png" width="396" />

Если вам нужно задать простые параметры стиля, сделайте это с помощью `styleOptions`. В параметрах стиля приводится набор предварительно определенных стилей, которые можно изменять напрямую. На их основе веб-чат будет вычислять всю таблицу стилей.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         const styleOptions = {
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleOptions' when rendering Web Chat
               styleOptions
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-css-manually"></a>Изменение CSS вручную

Помимо цвета вы можете изменить шрифты, используемые для отображения сообщений:

<img alt="Screenshot with custom style set" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-style-set.png" width="396" />

Для более глубокой настройки стилей также можно изменить стиль, заданный вручную, напрямую задав правила CSS.

> Так как правила CSS тесно связаны со структурой дерева DOM, вероятно, эти правила необходимо обновить для работы с более новой версией веб-чата.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         // "styleSet" is a set of CSS rules which are generated from "styleOptions"
         const styleSet = window.WebChat.createStyleSet({
            bubbleBackground: 'rgba(0, 0, 255, .1)',
            bubbleFromUserBackground: 'rgba(0, 255, 0, .1)'
         });

         // After generated, you can modify the CSS rules
         styleSet.textContent = {
            ...styleSet.textContent,
            fontFamily: "'Comic Sans MS', 'Arial', sans-serif",
            fontWeight: 'bold'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),

               // Passing 'styleSet' when rendering Web Chat
               styleSet
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

## <a name="change-the-avatar-of-the-bot-within-the-dialog-box"></a>Изменение аватара бота в диалоговом окне

В последней версии Web Chat поддерживаются аватары, которые можно настроить с помощью `botAvatarInitials` и `userAvatarInitials` в свойстве `styleOptions`.

<img alt="Screenshot with avatar initials" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-avatar-initials.png" width="396" />

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         const styleOptions = {
            botAvatarInitials: 'BF',
            userAvatarInitials: 'WC'
         };

         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  secret: 'YOUR_BOT_SECRET'
               }),
               styleOptions
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

В свойство `styleOptions` Web Chat мы добавили `botAvatarInitials` и `userAvatarInitials`.

```js
botAvatarInitials: 'BF',
userAvatarInitials: 'WC'
```

`botAvatarInitials` задаст текст внутри аватара в левой части окна. Если присвоено ложное значение, аватар на стороне бота будет скрыт. Напротив, `userAvatarInitials` задаст текст аватара в правой части окна.

## <a name="custom-rendering-activity-or-attachment"></a>Пользовательские действия или вложения преобразования для просмотра

С помощью последней версии веб-чата можно также преобразовать для просмотра действия или вложения, которые веб-чат не поддерживает. Преобразование для просмотра действий и вложений отправляется через настраиваемый конвейер, смоделированный после ПО промежуточного слоя [Redux ](https://redux.js.org/api/applymiddleware). Конвейер достаточно гибкий, чтобы вы могли выполнить следующие задачи:

-  Дополнить имеющиеся действия или вложения.
-  Добавить новые действия и вложения.
-  Заменить имеющиеся действия и вложения (или удалить их).
-  Поместить ПО промежуточного слоя в цепочку репликации.

### <a name="show-github-repository-as-an-attachment"></a>Отображение репозитория GitHub как вложения

Если вы хотите отобразить колоду карт репозитория GitHub, можно создать компонент React для репозитория GitHub и добавить его в качестве ПО промежуточного слоя для вложения.

<img alt="Screenshot with custom GitHub repository attachment" src="https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/sample-custom-github-repository-attachment.png" width="396" />

```jsx
import ReactWebChat from 'botframework-webchat';
import ReactDOM from 'react-dom';

// Create a new React component that accept render a GitHub repository attachment
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);

// Creating a new middleware pipeline that will render <GitHubRepositoryAttachment> for specific type of attachment
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};

ReactDOM.render(
   <ReactWebChat
      // Prepending the new middleware pipeline
      attachmentMiddleware={attachmentMiddleware}
      directLine={window.WebChat.createDirectLine({ token })}
   />,
   document.getElementById('webchat')
);
```

Полный пример можно найти в [примере customization-card-components](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/10.a.customization-card-components).

В этом примере мы добавляем новый компонент React, называемый `GitHubRepositoryAttachment`:

```jsx
const GitHubRepositoryAttachment = props => (
   <div
      style={{
         fontFamily: "'Calibri', 'Helvetica Neue', Arial, sans-serif",
         margin: 20,
         textAlign: 'center'
      }}
   >
      <svg
         height="64"
         viewBox="0 0 16 16"
         version="1.1"
         width="64"
         aria-hidden="true"
      >
         <path
            fillRule="evenodd"
            d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"
         />
      </svg>
      <p>
         <a
            href={`https://github.com/${encodeURI(props.owner)}/${encodeURI(
               props.repo
            )}`}
            target="_blank"
         >
            {props.owner}/<br />
            {props.repo}
         </a>
      </p>
   </div>
);
```

Затем создаем ПО промежуточного слоя, которое преобразует для просмотра новый компонент React, когда бот отправляет вложение с типом содержимого `application/vnd.microsoft.botframework.samples.github-repository`. В противном случае он будет по-прежнему работать на ПО промежуточного слоя путем вызова `next(card)`.

```jsx
const attachmentMiddleware = () => next => card => {
   switch (card.attachment.contentType) {
      case 'application/vnd.microsoft.botframework.samples.github-repository':
         return (
            <GitHubRepositoryAttachment
               owner={card.attachment.content.owner}
               repo={card.attachment.content.repo}
            />
         );

      default:
         return next(card);
   }
};
```

Действие, отправленное из бота, выглядит следующим образом:

```json
{
   "type": "message",
   "from": {
      "role": "bot"
   },
   "attachmentLayout": "carousel",
   "attachments": [
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-WebChat"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-Emulator"
         }
      },
      {
         "contentType": "application/vnd.microsoft.botframework.samples.github-repository",
         "content": {
            "owner": "Microsoft",
            "repo": "BotFramework-DirectLineJS"
         }
      }
   ]
}
```
