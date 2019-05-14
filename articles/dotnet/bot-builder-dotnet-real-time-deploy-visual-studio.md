---
redirect_url: https://aka.ms/realTimeMediaCalling-repo
ms.openlocfilehash: 1df0192632cdb9b35259b8ce1ec5c8b3be46c750
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032940"
---
<a name="--"></a><!--
---
заголовок: Развертывание в Azure интерактивного мультимедийного бота для Skype | Документация Майкрософт description: Сведения о том, как развернуть в Azure интерактивный бот для аудио- и видеозвонков Skype с помощью встроенной функции публикации Visual Studio.
author: MalarGit ms.author: malarch manager: ssulzer ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 12/13/17 monikerRange: 'azure-bot-service-3.0'
---

# <a name="deploy-a-real-time-media-bot-from-visual-studio-to-azure"></a>Развертывание в Azure интерактивного мультимедийного бота из Visual Studio

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Интерактивный мультимедийный бот можно разместить на виртуальной машине Azure IaaS или в классической облачной службе Azure. В этой статье объясняется, как развернуть бот, размещенный в рабочей роли облачной службы Azure, из Visual Studio с помощью встроенной возможности публикации.

## <a name="prerequisites"></a>Предварительные требования

Для развертывания бота в Azure вам потребуется подписка Microsoft Azure. Если у вас еще нет подписки, вы можете зарегистрироваться для получения <a href="https://azure.microsoft.com/en-us/free/" target="_blank">бесплатной учетной записи</a>. Кроме того, чтобы выполнить описанный в этой статье процесс, потребуется Visual Studio. Если у вас нет Visual Studio, вы можете скачать <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017 Community</a> бесплатно.

### <a name="certificate-from-a-valid-certificate-authority"></a>Сертификат из допустимого центра сертификации
При настройке бота нужно использовать действующий сертификат из доверенного центра сертификации. Именем субъекта или последней записью альтернативного имени субъекта (SAN) сертификата должно быть имя облачной службы. Групповые сертификаты не поддерживаются. Если для указания облачной службы используется запись CNAME, этой записью должно быть имя субъекта или последняя запись SAN сертификата.

## <a name="configure-application-settings"></a>Настройка параметров приложения
Для надлежащей работы бота в облачной среде необходимо проверить правильность параметров приложения. В частности, задайте следующие значения ключа в файле app.config рабочей роли:
> <ul><li>MicrosoftAppId;</li><li>MicrosoftAppPassword.</li></ul>

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** для бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

## <a name="create-worker-role-in-the-azure-portal"></a>Создание рабочей роли на портале Azure
### <a name="step-1-create-cloud-serviceclassic"></a>Шаг 1. Создание облачной службы (классическая)
Войдите на <a href="https://portal.azure.com">портал Azure</a>. В левой части страницы щелкните **+** и выберите **Облачные службы (классические)**. Введите необходимые сведения в форму и нажмите кнопку **Создать**.

![Создание облачной службы](../media/real-time-media-bot-portal-service-creation.png)

> [!NOTE]
> DNS-имя бота должно быть указано в URL-адресе для регистрации бота.

### <a name="step-2-upload-the-certificate-for-the-bot"></a>Шаг 2. Отправка сертификата для бота
После создания бота отправьте для него сертификат.

![Передача сертификата](../media/real-time-media-bot-portal-certificates.png)

## <a name="modify-service-configuration-with-worker-role-details"></a>Изменение конфигурации службы с указанием сведений о рабочей роли
Полное доменное имя (FQDN) бота не предоставляется с помощью API-интерфейсов Azure RoleEnvironment. Поэтому для бота нужно указать FQDN. Также необходимо указать сертификат для HTTPS. Эти параметры можно настроить в файле конфигурации службы (.cscfg) рабочей роли.

> [!TIP]
> Если вы развертываете пример BotBuilder-RealTimeMediaCalling из репозитория Git,
> - замените $DnsName$ именем облачной службы или записью CNAME, если она используется в конфигурации службы.
>   ```xml
>      <Setting name="ServiceDnsName" value="$DnsName$" />
>   ```
> 
> - Замените $CertThumbprint$ отпечатком сертификата, отправленным боту, в следующих строках файла конфигурации:
>   ```xml
>      <Setting name="DefaultCertificate" value="$CertThumbprint$" />
>      <Certificate name="Default" thumbprint="$CertThumbprint$" thumbprintAlgorithm="sha1" />
>   ```

## <a name="publish-the-bot-from-visual-studio"></a>Публикация бота из Visual Studio
### <a name="step-1-launch-the-microsoft-azure-publishing-wizard-in-visual-studio"></a>Шаг 1. Запуск мастера публикации Microsoft Azure в Visual Studio

Откройте проект в Visual Studio. В обозревателе решений щелкните правой кнопкой мыши проект облачной службы и выберите пункт **Опубликовать**. Запустится мастер публикации Microsoft Azure. Войдите в соответствующую подписку, указав свои учетные данные.

![Щелкните правой кнопкой мыши проект и выберите пункт "Опубликовать", чтобы запустить мастер публикации Microsoft Azure.](../media/real-time-media-bot-publish-signin.png)

### <a name="step-2-publish-the-bot"></a>Шаг 2. Публикация бота

Щелкните **Далее**. Откроется вкладка **Параметры**. Укажите облачную службу, среду, конфигурацию сборки и конфигурацию службы для развертывания бота.

![Нажатие кнопки "Далее" и переход на вкладку "Параметры"](../media/real-time-media-bot-publish-settings.png)

При необходимости можно выбрать **Дополнительные параметры** и указать учетную запись хранения для журналов развертывания (которые можно использовать для отладки проблем).

![Выбор вкладки "Дополнительные параметры"](../media/real-time-media-bot-publish-advanced-settings.png)

Проверьте конфигурацию на вкладке **Сводка** и щелкните **Опубликовать**, чтобы развернуть бот в Microsoft Azure.
-->