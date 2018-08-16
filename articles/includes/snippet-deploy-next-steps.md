## <a name="next-steps"></a>Дополнительная информация
Развернув бот в облаке и убедившись, что развертывание выполнено успешно (проверив бот с помощью Bot Framework Emulator), можно перейти к следующему шагу процесса публикации бота. Он зависит от того, зарегистрировали ли вы уже бот с помощью платформы Bot Framework.

### <a name="if-you-have-already-registered-your-bot-with-the-bot-framework"></a>Если вы уже зарегистрировали бот с помощью Bot Framework, сделайте следующее.

1. Вернитесь на <a href="https://dev.botframework.com" target="_blank">портал Bot Framework</a> и [обновите параметры своего бота](~/bot-service-manage-settings.md), указав **конечную точку обмена сообщениями** для бота.

2. [Настройте бот для работы в одном или нескольких каналах](~/bot-service-manage-channels.md).

### <a name="if-you-have-not-yet-registered-your-bot-with-the-bot-framework"></a>Если вы еще не зарегистрировали бот с помощью Bot Framework, сделайте следующее.

1. [Зарегистрируйте бот на платформе Bot Framework](~/bot-service-quickstart-registration.md).

2. Обновите значения идентификатора приложения Майкрософт и пароля приложения Майкрософт в параметрах конфигурации развернутого приложения, чтобы указать значения **appID** и **password**, которые были созданы для бота во время регистрации. Сведения о том, как найти значения **AppID** и **AppPassword** вашего бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

3. [Настройте бот для работы в одном или нескольких каналах](~/bot-service-manage-channels.md).