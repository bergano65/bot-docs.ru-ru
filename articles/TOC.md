# [Документация по службе Azure Bot](index.yml)
# Обзор
## [Общие сведения о службе Azure Bot](bot-service-overview-introduction.md)
## [Новые возможности](what-is-new.md)
# Краткое руководство
## [Создание бота с помощью .NET](dotnet/bot-builder-dotnet-sdk-quickstart.md)
## [Создание бота с помощью JavaScript](javascript/bot-builder-javascript-quickstart.md)
## [Создание бота с помощью Python](python/bot-builder-python-quickstart.md)
## [Создание бота с помощью службы Azure Bot](v4sdk/abs-quickstart.md)
# Учебники
## [1. Создание и развертывание простого бота](v4sdk/bot-builder-tutorial-basic-deploy.md)
## [2. Добавление QnA Maker и повторное развертывание бота](v4sdk/bot-builder-tutorial-add-qna.md)
## [Добавление аутентификации в бот](bot-builder-tutorial-authentication.md)
# Примеры
## [Репозиторий примеров для Bot Framework на сайте GitHub](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md)
# Основные понятия
## [Принципы работы бота](v4sdk/bot-builder-basics.md)
## [Управление состоянием](v4sdk/bot-builder-concept-state.md)
## [Библиотека диалогов](v4sdk/bot-builder-concept-dialog.md)
## [ПО промежуточного слоя](v4sdk/bot-builder-concept-middleware.md)
## Аутентификация
### [Аутентификация бота](v4sdk/bot-builder-concept-authentication.md)
### [Поставщики удостоверений](v4sdk/bot-builder-concept-identity-providers.md)
### [Единый вход](v4sdk/bot-builder-concept-sso.md)
## [Управление ресурсами бота](v4sdk/bot-file-basics.md)
## [Принцип работы ботов для Microsoft Teams](v4sdk/bot-builder-basics-teams.md)
## [Сведения о навыках](v4sdk/skills-conceptual.md)
<!-- [Language understanding](v4sdk/bot-builder-concept-luis.md) -->
## [Шаблоны службы Bot](bot-service-concept-templates.md)
## [Cognitive Services](bot-service-concept-intelligence.md)
## [Основные сценарии для ботов](bot-service-scenario-overview.md)
### [Бот для коммерческих операций](bot-service-scenario-commerce.md)
### [Бот для навыков Cortana](bot-service-scenario-cortana-skill.md)
### [Бот для повышения производительности на предприятии](bot-service-scenario-enterprise-productivity.md)
### [Информационный бот](bot-service-scenario-informational.md)
### [Бот для Интернета вещей](bot-service-scenario-internet-things.md)
# Инструкции
## [Конструктор](design/TOC.md)
## Разработка
<!-- ## [Best practice for welcoming the user](v4sdk/bot-builder-welcome-user.md) -->
### [Отправка и получение текстовых сообщений](v4sdk/bot-builder-howto-send-messages.md)
### [Добавление мультимедиа в сообщения](v4sdk/bot-builder-howto-add-media-attachments.md)
### [Добавление кнопок для управления действиями пользователя](v4sdk/bot-builder-howto-add-suggested-actions.md)
### [Сохранение данных пользователя и диалога](v4sdk/bot-builder-howto-v4-state.md)
### [Запрос на ввод данных пользователем](v4sdk/bot-builder-primitive-prompts.md)
### [Отправка приветственного сообщения пользователям](v4sdk/bot-builder-send-welcome-message.md)
### [Отправка упреждающих уведомлений пользователям](v4sdk/bot-builder-howto-proactive-message.md)
### [Реализация процесса общения](v4sdk/bot-builder-dialog-manage-conversation-flow.md)
### [Добавление возможности распознавания естественного языка в функционал бота](v4sdk/bot-builder-howto-v4-luis.md)
### [Использование QnA Maker для ответов на вопросы пользователя](v4sdk/bot-builder-howto-qna.md)
### [Использование нескольких моделей LUIS и QnA](v4sdk/bot-builder-tutorial-dispatch.md)
### [Создание сложной последовательной беседы с использованием ветвей и циклов](v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)
### [Повторное использование диалогов](v4sdk/bot-builder-compositcontrol.md)
<!--#### [Implement a greeting dialog](v4sdk/bot-builder-dialogs-greeting.md)--TODO: Add once there's a sample.-->
### [Обработка прерываний диалога пользователем](v4sdk/bot-builder-howto-handle-user-interrupt.md)
### [Непосредственная запись в хранилище](v4sdk/bot-builder-howto-v4-storage.md)
### [Добавление аутентификации в веб-приложение](v4sdk/bot-builder-authentication.md)
### [Реализация пользовательского хранилища для бота](v4sdk/bot-builder-custom-storage.md)
### Навыки
#### [Реализация навыка](v4sdk/skill-implement-skill.md)
#### [Реализация потребителя навыков](v4sdk/skill-implement-consumer.md)
<!--
#### [Add claims validation](v4sdk/skill-add-claims-validation.md)
#### [Manage multiple skills](#)
-->
### [Добавление данных телеметрии в бот](v4sdk/bot-builder-telemetry.md)
### [Добавление данных телеметрии в код чат-бота для ответов на вопросы](v4sdk/bot-builder-telemetry-QnAMaker.md)
### [Анализ данных телеметрии бота](v4sdk/bot-builder-telemetry-analytics-queries.md)
### [Использование канала "Речь Direct Line" в боте](directline-speech-bot.md)
### [.NET](dotnet/TOC.md)
### [Node.js](nodejs/TOC.md)
## Тест
### [Модульное тестирование ботов](v4sdk/unit-test-bots.md)
### [Добавление действий трассировки в код бота](v4sdk/using-trace-activities.md)
## [Отладка](debug/TOC.md)
## Развертывание
### [Развертывание бота в Azure](bot-builder-deploy-az-cli.md)
### [Настройка непрерывного развертывания](bot-service-build-continuous-deployment.md)
## [Управление](manage/TOC.md)
## [анализа](v4sdk/migration/TOC.md)
# Справочник
## [Пакет SDK версии 4 для .NET](https://aka.ms/botframework-v4-cs-sdk)
## [Пакет SDK версии 4 для JavaScript](https://aka.ms/bot-jssdk-v4)
## [Пакет SDK для Python версии 4](https://aka.ms/botframework-v4-python-sdk)
## [REST](rest-api/TOC.md)
## [Пакет SDK версии 3 для .NET](https://aka.ms/botframework-v3-cs-sdk)
## [Пакет SDK версии 3 для Node.js](https://aka.ms/bot-jssdk-v3)
## Справочные материалы по работе с инструментом командной строки BF
### [Общие сведения об инструменте командной строки BF](v4sdk/bf-cli-overview.md)
### [Справочник по инструменту командной строки BF](v4sdk/bf-cli-reference.md)
## [Сущности и типы действий](bot-service-activities-entities.md)
# [Ресурсы](resources/TOC.md)
