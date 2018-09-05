---
title: Внедрение бота в приложение | Документация Майкрософт
description: Узнайте о том, как разработать бот, который будет внедрен в приложение.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/15/2018
ms.openlocfilehash: 68d2d4f7a19aecfcb2c630e5e9e55ca5b8a21d89
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/20/2018
ms.locfileid: "42756639"
---
# <a name="embed-a-bot-in-an-app"></a>Внедрение бота в приложение

Несмотря на то что боты чаще всего находятся за пределами приложения, их также можно интегрировать с приложениями. Например, можно внедрить [бот набора знаний](~/bot-service-design-pattern-knowledge-base.md) в приложение, чтобы пользователи могли быстро найти сведения, которые в противном случае было бы трудно найти в сложной структуре приложения. Бот можно внедрить в приложение службы поддержки, которое будет выступать в качестве первого отвечающего устройства на входящие запросы пользователей. Бот может независимо устранять простые проблемы и [передавать](~/bot-service-design-pattern-handoff-human.md) более сложные агенту-человеку. 

## <a name="integrating-bot-with-app"></a>Интеграция бота с приложением

Способ интеграции бота с приложением зависит от типа приложения. 

### <a name="native-mobile-app"></a>Собственное мобильное приложение

Приложение, создаваемое с использованием машинного кода, может взаимодействовать с Bot Framework через [API для Direct Line][directLineAPI] с помощью REST или подключения Websocket.

### <a name="web-based-mobile-app"></a>Мобильное веб-приложение

Мобильное приложение, созданное с помощью веб-языка и платформ, таких как <a href="https://cordova.apache.org/" target="_blank">Cordova</a>, может взаимодействовать с Bot Framework, используя те же компоненты, что и [бот, внедренный на веб-сайте](~/bot-service-design-pattern-embed-web-site.md), только инкапсулированные в собственной оболочке приложения.

### <a name="iot-app"></a>Приложение Интернета вещей

Приложение Интернета вещей может взаимодействовать с Bot Framework с помощью [API для Direct Line][directLineAPI]. В некоторых сценариях они также могут использовать <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft Cognitive Services</a> для предоставления таких возможностей, как распознавание изображений и речь.

### <a name="other-types-of-apps-and-games"></a>Другие виды приложений и игр

Другие виды приложений и игр могут взаимодействовать с Bot Framework с помощью [API для Direct Line][directLineAPI]. 

## <a name="creating-a-cross-platform-mobile-app-that-runs-a-bot"></a>Создание кроссплатформенного мобильного приложения для выполнения бота

В этом примере создания мобильного приложения для выполнения бота используется <a href="https://www.xamarin.com/" target="_blank">Xamarin</a>, популярное средство для создания кроссплатформенных мобильных приложений. 

Сначала создайте простой веб-компонент для просмотра и используйте его для размещения <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">элемента управления веб-чатом</a>. Затем с помощью портала Azure добавьте канал веб-чата. 

Затем укажите зарегистрированный URL-адрес веб-чата в качестве источника для элемента управления для просмотра в приложении Xamarin:

```cs
public class WebPage : ContentPage
{
    public WebPage()
    {
        var browser = new WebView();
        browser.Source = "https://webchat.botframework.com/embed/<YOUR SECRET KEY HERE>";
        this.Content = browser;
    }
}
```

С помощью этого процесса вы можете создать кроссплатформенное мобильное приложение, преобразовывающее внедренное веб-представление с элементом управления веб-чата для просмотра.

![Обратный канал](~/media/bot-service-design-pattern-embed-app/xamarin-apps.png)

## <a name="sample-code"></a>Пример кода

Полный пример, в котором показано, как создать кроссплатформенное мобильное приложение, выполняющее бот (как описано в этой статье), см. в <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/capability-BotInApps" target="_blank">примере бота в приложениях</a> на сайте GitHub.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [API для Direct Line][directLineAPI]
- <a href="https://www.microsoft.com/cognitive-services/" target="_blank">Microsoft Cognitive Services</a>.

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
