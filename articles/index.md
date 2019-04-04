---
title: Служба Azure Bot | Документация Майкрософт
description: Создание ботов с помощью службы Azure Bot.
author: kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: landing-page
layout: LandingPage
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/28/2019
ms.openlocfilehash: eef74893d4d17536fa8a7c3add15f1fd6a614796
ms.sourcegitcommit: 54a4382add4756346098b286695a9b4791db7139
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2019
ms.locfileid: "58616960"
---
::: moniker range="azure-bot-service-3.0"

<div class="content">
    <h1>Документация по службе Azure Bot</h1>
    <div class="alert is-info">
        <p class="alert-title"><span class="docon docon-status-error-outline"></span> Примечание</p>
        <p>Эта статья содержит сведения о <strong>предыдущей версии пакета SDK (версии 3)</strong>. С документацией по текущей версии пакета SDK (версии 4) можно ознакомиться <a href="https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0" data-linktype="external">здесь</a>.</p>
    </div>
    <div class="intro" style="min-width: 200px">
        <p>Служба Bot предоставляет интегрированную среду, предназначенную для разработки ботов, которая позволяет создавать, подключать, тестировать, развертывать и администрировать интеллектуальные боты из единого центра. В службе Azure Bot используется пакет SDK для Bot Framework с поддержкой .NET и Node.js. Узнайте, как использовать службу Bot, изучив краткое руководство и примеры.</p>
    </div>
<h2 style="margin-top: 18px; margin-bottom: 0px;">5-минутные руководства по началу работы</h2>
<div class="ico48Case">
    <div class="ico48Link">
        <a href="/bot-framework/bot-service-quickstart">
            <img src="media/index/azure_portal.png" alt="">
            <span>Портал Azure</span>
        </a>
    </div>
</div>
 
<h2 style="margin-top: 36px">Примеры</h2>
<p>Сведения о том, как быстро приступить к созданию полезных ботов с широкими возможностями.</p>
<ul>
    <li><a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp">.NET</a></li>
    <li><a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node">Node.js</a></li>
</ul>
<h2 style="margin-top: 36px">Пошаговые руководства</h2>
<p> <a href="/bot-framework/bot-builder-tutorial-authentication">Добавление проверки подлинности в бот</a> </p>
<h2 style="margin-top: 36px">Справочные материалы</h2>
<ul class="panelContent cardsD">
    <li>
        <div class="cardSize">
            <div class="cardPadding">
                <div class="card">
                    <div class="cardText">
                        <h3>Languages</h3>
                        <p><a href="/dotnet/api/?view=botbuilder-3.12.2.4">.NET</a></p>
                        <p><a href="https://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_.html">Node.js</a></p>
                    </div>
                </div>
            </div>
        </div>
    </li>
</ul>
</div>


::: moniker-end

::: moniker range="azure-bot-service-4.0"

<div class="content">
    <h1>Документация по службе Azure Bot</h1>
    <div class="alert is-info">
        <p class="alert-title"><span class="docon docon-status-error-outline"></span> Примечание</p>
        <p>Эта статья содержит сведения о текущей версии пакета SDK (версии 4). С документацией по предыдущей версии пакета SDK (версии 3) можно ознакомиться <a href="https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-3.0" data-linktype="external">здесь.</a></p>
    </div>
    <div class="intro" style="min-width: 200px">
        <p>Служба Azure Bot предоставляет интегрированную среду, предназначенную для разработки ботов, которая позволяет создавать, подключать, тестировать, развертывать и администрировать интеллектуальные боты из единого центра. В службе Azure Bot используется пакет SDK для Bot Framework с поддержкой C# и JavaScript. Узнайте, как использовать службу Bot, изучив краткие инструкции, примеры и руководства.
</p>
</div>

<h2 style="margin-top: 18px; margin-bottom: 0px;">5-минутные руководства по началу работы</h2>
<p style="margin-top: 6px; margin-bottom: 6px;"></p>
<div class="ico48Case">
    <div class="ico48Link">
        <a href="/bot-framework/bot-service-quickstart">
            <img src="media/index/azure_portal.png" alt="">
            <span>Портал Azure</span>
        </a>
    </div>
    <div class="ico48Link">
        <a href="/bot-framework/dotnet/bot-builder-dotnet-sdk-quickstart">
            <img src="v4sdk/media/logo_csharp.svg" alt="">
            <span>C&#35;</span>
        </a>
    </div>
    <div class="ico48Link">
        <a href="/bot-framework/javascript/bot-builder-javascript-quickstart">
            <img src="v4sdk/media/logo_js.svg" alt="">
            <span>JavaScript</span>
        </a>
    </div>
</div>

<h2 style="margin-top: 36px">Пошаговые руководства</h2>
<p><a href="/bot-framework/bot-builder-tutorial-basic-deploy">1. Создание и развертывание простого бота</a></p>
<p><a href="/bot-framework/bot-builder-tutorial-add-qna">2. Добавление QnA Maker и повторное развертывание бота</a></p>
<h2 style="margin-top: 36px">Справочные материалы</h2>
<ul class="panelContent cardsD">
    <li>
        <div class="cardSize">
            <div class="cardPadding">
                <div class="card">
                    <div class="cardText">
                        <h3>Интерфейсы API</h3>
                        <p><a href="https://aka.ms/dotnetsdk4">.NET</a></p>
                        <p><a href="https://aka.ms/jssdk4">JavaScript</a></p>
                    </div>
                </div>
            </div>
        </div>
    </li>
    <li>
        <div class="cardSize">
            <div class="cardPadding">
                <div class="card">
                    <div class="cardText">
                        <h3>Пакеты SDK</h3>
                        <p><a href="https://github.com/Microsoft/botbuilder-dotnet">.NET</a></p>
                        <p><a href="https://github.com/Microsoft/botbuilder-js">JavaScript</a></p>
                    </div>
                </div>
            </div>
        </div>
    </li>
</ul>
</div>

::: moniker-end
