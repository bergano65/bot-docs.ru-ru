---
title: Развертывание бота в Azure | Документация Майкрософт
description: Развертывание бота в облаке Azure.
keywords: развертывание бота, развертывание Azure, регистрация канала ботов, публикации Visual Studio
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 05/14/2018
ms.openlocfilehash: 70a3b7f093bb80dd16c854c65331c141fbba3725
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306074"
---
# <a name="deploy-your-bot-to-azure"></a>Развертывание бота в Azure

После создания и локальной проверки бот можно передать в Azure, чтобы сделать его доступным из любого места. Для этого сначала разверните бот в Службе приложений Azure, а затем настройте бот с помощью службы Azure Bot, используя элемент Службы регистрации каналов ботов.

## <a name="publish-from-visual-studio"></a>Публикация из Visual Studio

Используйте Visual Studio для создания ресурсов в Azure и публикации кода.

В окне обозревателя решений щелкните правой кнопкой мыши узел проекта и выберите "Опубликовать".

![Параметры публикации](media/azure-bot-quickstarts/getting-started-publish-setting.png)

2. Убедитесь, что в диалоговом окне "Выберите целевой объект публикации" в левой части выбрано **Служба приложений**, а в правой части — **Создать**.

3. Нажмите кнопку "Опубликовать".

4. Убедитесь, что в верхнем правом углу диалогового окна отображается правильный идентификатор пользователя для подписки Azure.

![Публикация основы](media/azure-bot-quickstarts/getting-started-publish-main.png)

5. Введите сведения об имени приложения, подписке, группе ресурсов и плане размещения.

6. Когда все будет готово, нажмите кнопку "Создать". Процесс создания может занять несколько минут.

7. После завершения откроется веб-браузер, отображающий общедоступный URL-адрес бота.

8. Скопируйте этот URL-адрес (он будет выглядеть примерно так: https://<yourbotname>.azurewebsites.net/).

> [!NOTE] 
> При регистрации бота необходимо использовать HTTPS-версию URL-адреса. Azure предоставляет поддержку протокола SSL с помощью Службы приложений Azure.

## <a name="create-your-bot-channels-registration"></a>Создание регистрации канала бота
После развертывания бота в Azure необходимо зарегистрировать его с помощью службы Azure Bot.

1. Перейдите на портал Azure по ссылке https://portal.azure.com.

2. Войдите, используя тот же идентификатор, который использовался ранее в Visual Studio для публикации бота.

3. Щелкните "Создать ресурс".

4. В поле поиска по Marketplace введите "Регистрация канала бота" и нажмите клавишу ВВОД.

5. В возвращенном списке выберите "Регистрация канала бота".

![Опубликовать](media/azure-bot-quickstarts/getting-started-bot-registration.png)

6. В раскрывшейся колонке щелкните "Создать".

7. Задайте имя бота.

8. Выберите ту же подписку, в которой был развернут код бота.

9. Выберите существующую группу ресурсов, которая задаст расположение.

10. Для разработки и тестирования можно выбрать ценовую категорию F0.

11. Введите URL-адрес бота. Убедитесь, что адрес начинается с префикса HTTPS, и добавьте /api/messages. Например, https://yourbotname.azurewebsites.net/api/messages.

12. Пока можно отключить Application Insights.

13. Выберите идентификатор приложения Майкрософт и пароль.

14. В новой колонке щелкните "Создать".

15. В новой колонке, которая раскроется справа, щелкните "Создать идентификатор приложения в портале регистрации приложений". Откроется новая вкладка браузера.

![MSA бота](media/azure-bot-quickstarts/getting-started-msa.png)

16. В новой вкладке скопируйте идентификатор приложения и сохраните его. 

17. Нажмите кнопку "Создать пароль приложения, чтобы продолжить".

18. Откроется диалоговое окно браузера, в котором будет предоставлен пароль приложения. Это единственный раз, когда вы его получите. Скопируйте и сохраните этот пароль.

19. После сохранения пароля щелкните "OK".

20. Просто закройте эту вкладку браузера и вернитесь на вкладку портала Azure.

21. Вставьте идентификатор приложения и пароль в соответствующие поля и щелкните "OK".

22. Теперь щелкните "Создать", чтобы настроить регистрацию канала. Это может занять от нескольких секунд до нескольких минут.

## <a name="update-your-bots-application-settings"></a>Обновление параметров приложения бота
Чтобы бот выполнил проверку подлинности с помощью службы Azure Bot, необходимо добавить два параметра для приложения бота в Службе приложений Azure. 

1. Щелкните "Службы приложений". Введите имя бота в текстовое поле подписок. Затем щелкните имя бота в списке.

![службы приложений](media/azure-bot-quickstarts/getting-started-app-service.png)

2. В списке параметров слева в параметрах бота найдите и щелкните "Параметры приложения" в разделе "Параметры".

![Идентификатор бота](media/azure-bot-quickstarts/getting-started-app-settings-1.png)

3. Найдите раздел параметров приложения.

![MSA бота](media/azure-bot-quickstarts/getting-started-app-settings-2.png)

4. Выберите "Добавить новый параметр".

5. В поле имени введите **MicrosoftAppId**, а в поле значения — идентификатор приложения.

6. Выберите "Добавить новый параметр".

7. В поле имени введите **MicrosoftAppPassword**, а в поле значения — пароль.

8. Нажмите кнопку "Сохранить" вверху.

## <a name="test-your-bot-in-production"></a>Тестирование бота в рабочей среде
На этом этапе можно протестировать бот из Azure с помощью встроенного клиента "Веб-чат".

1. Вернитесь в группу ресурсов на портале.

2. Откройте регистрацию бота.

3. В разделе управление ботом выберите "Тестировать в веб-чате".

![Тестирование в веб-чате](media/azure-bot-quickstarts/getting-started-test-webchat.png)

4. Введите сообщение, например, `Hi`, и нажмите клавишу ВВОД. Бот передаст обратно `Turn 1: You sent Hi`.
