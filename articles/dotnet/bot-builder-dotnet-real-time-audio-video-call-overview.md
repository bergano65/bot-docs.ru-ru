---
title: Создание бота мультимедиа в реальном времени для Skype | Документы Майкрософт
description: Сведения о создании бота, который выполняет аудио- и видеозвонки Skype в реальном времени, с помощью пакета SDK построителя ботов для .NET и пакета SDK построителя ботов RealTimeMediaCalling для .NET.
author: MalarGit
ms.author: malarch
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6ceeca9adc9cad9e60a73c1c7c91bea43b97fdd9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997933"
---
# <a name="build-a-real-time-media-bot-for-skype"></a>Создание бота мультимедиа в реальном времени для Skype

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Платформа обработки мультимедиа в реальном времени для ботов представляет собой дополнительную функцию, которая позволяет ботам отправлять и получать аудио- и видеоконтент по кадрам. Бот получает доступ к необработанным данным модальностей общего доступа к экрану, видео и голоса в режиме реального времени. Эта статья содержит общие сведения о создании бота, выполняющего аудио- и видеозвонки, а также о доступе к модальностям реального времени.

В этой статье бот выполняется в облачной службе Azure как веб-роль или рабочая роль с самостоятельным размещением платформы веб-API ASP.NET.

> [!IMPORTANT]
> Содержимое этой статьи является предварительным. Чтобы получить полный код примера бота мультимедиа в реальном времени, обратитесь к папке Samples в репозитории GitHub <a href="https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling">BotBuilder RealTimeMediaCalling</a>.

## <a name="configure-the-service-hosting-the-real-time-media-bot"></a>Настройка службы размещения бота мультимедиа в реальном времени

Чтобы использовать платформу мультимедиа в реальном времени, необходимы следующие конфигурации службы.

* Боту должно быть известно полное доменное имя (FQDN) его службы. Оно не предоставляется API Azure RoleEnvironment. Полное доменное имя должно храниться в конфигурации облачной службы бота и считываться во время ее запуска.

* Службе ботов требуется сертификат, выданный признанным центром сертификации. Отпечаток сертификата должен храниться в конфигурации облачной службы бота и считываться при запуске службы.

* Должна быть подготовлена общедоступная <a href="/azure/cloud-services/cloud-services-enable-communication-role-instances#instance-input-endpoint">входная конечная точка экземпляра</a>. Это позволит назначить уникальный общедоступный порт каждому экземпляру виртуальной машины в службе бота. Этот порт используется платформой мультимедиа в реальном времени для обмена данными с облаком вызовов Skype.
  ```xml
  <InstanceInputEndpoint name="InstanceMediaControlEndpoint" protocol="tcp" localPort="20100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="20200" min="20101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

  Кроме того, рекомендуется создать еще одну входную конечную точку экземпляра для обратных вызовов и уведомлений, связанных с вызовами. Использование входной конечной точки экземпляра обеспечивает доставку обратных вызовов и уведомлений на одну виртуальную машину в развертывании службы, где размещается сеанс мультимедиа в реальном времени для вызова.
  ```xml
  <InstanceInputEndpoint name="InstanceCallControlEndpoint" protocol="tcp" localPort="10100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="10200" min="10101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

* Каждый экземпляр виртуальной машины должен иметь общедоступный IP-адрес уровня экземпляра (ILPIP). Во время запуска боту необходимо обнаружить адрес ILPIP, назначенный для каждого экземпляра службы. Дополнительные сведения о получении и настройке ILPIP-адреса см. в разделе <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip">ILPIP</a>.
  ```xml
  <NetworkConfiguration>
  <AddressAssignments>
    <InstanceAddress roleName="WorkerRole">
    <PublicIPs>
        <PublicIP name="InstancePublicIP" domainNameLabel="InstancePublicIP" />
    </PublicIPs>
    </InstanceAddress>
  </AddressAssignments>
  </NetworkConfiguration>
  ```

* Во время запуска экземпляра службы сценарий `MediaPlatformStartupScript.bat` (предоставляемый в составе пакета NuGet) должен запускаться как задача запуска с повышенными правами. Сценарий должен быть выполнен до вызова метода инициализации платформы. 

```xml
<Startup>
<Task commandLine="MediaPlatformStartupScript.bat" executionContext="elevated" taskType="simple" />      
</Startup> 
```

## <a name="initialize-the-media-platform-on-service-startup"></a>Инициализация платформы мультимедиа при запуске службы

При запуске экземпляра службы должна быть инициализирована платформа мультимедиа в реальном времени. Это необходимо сделать только один раз, после чего бот сможет принимать аудио- и видеозвонки Skype для этого экземпляра. Для инициализации платформы мультимедиа требуется предоставление различных параметров конфигурации службы, включая полное доменное имя службы, общедоступный ILPIP-адрес, номер порта входной конечной точки экземпляра и **идентификатор приложения Майкрософт** бота.

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** вашего бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

```cs
var mediaPlatformSettings = new MediaPlatformSettings()
{
    MediaPlatformInstanceSettings = new MediaPlatformInstanceSettings()
    {
        CertificateThumbprint = certificateThumbprint,
        InstanceInternalPort = instanceMediaControlEndpointInternalPort,
        InstancePublicIPAddress = instancePublicIPAddress,
        InstancePublicPort = instanceMediaControlEndpointPublicPort,
        ServiceFqdn = serviceFqdn
    },

    ApplicationId = MicrosoftAppId
};

MediaPlatform.Initialize(mediaPlatformSettings);            
```

## <a name="register-to-receive-incoming-call-requests"></a>Регистрация для получения запросов на входящие вызовы

Определите класс `CallController`. Это позволяет службе ботов подписываться на входящие вызовы Skype и обеспечивает перенаправление запросов на обратные вызовы и уведомления в соответствующий объект `RealTimeMediaCall`.

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallController : ApiController
{
    static CallController()
    {
        RealTimeMediaCalling.RegisterRealTimeMediaCallingBot(
            c => { return new RealTimeMediaCall(c); },
            new RealTimeMediaCallingBotServiceSettings()
        );
    }

    [Route("call")]
    public async Task<HttpResponseMessage> OnIncomingCallAsync()
    {
        // forwards the incoming call to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.IncomingCall);
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> OnCallbackAsync()
    {
        // forwards the incoming callback to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.CallingEvent);
    }

    [Route("notification")]
    public async Task<HttpResponseMessage> OnNotificationAsync()
    {
        // forwards the incoming notification to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.NotificationEvent);
    }
}
```

`RealTimeMediaCallingBotServiceSettings` реализует `IRealTimeMediaCallServiceSettings` и предоставляет ссылки веб-перехватчика для обратных вызовов и уведомлений.

## <a name="register-for-incoming-events-for-the-call"></a>Подписка на входящие события для вызова

Создайте класс `RealTimeMediaCall`, реализующий `IRealTimeMediaCall`. Для каждого вызова, который принимается ботом, экземпляр `RealTimeMediaCall` создается платформой Bot Framework. Объект `IRealTimeMediaCallService`, переданный в конструктор, позволяет боту подписываться на события для обработки событий, связанных с вызовом мультимедиа в реальном времени.

```cs
internal class RealTimeMediaCall : IRealTimeMediaCall
{
     public RealTimeMediaCall(IRealTimeMediaCallService callService)
     {
         if (callService == null)
             throw new ArgumentNullException(nameof(callService));

         CallService = callService;
         CorrelationId = callService.CorrelationId;
         CallId = CorrelationId + ":" + Guid.NewGuid().ToString();

         // Register for the call events
         CallService.OnIncomingCallReceived += OnIncomingCallReceived;
         CallService.OnAnswerAppHostedMediaCompleted += OnAnswerAppHostedMediaCompleted;
         CallService.OnCallStateChangeNotification += OnCallStateChangeNotification;
         CallService.OnRosterUpdateNotification += OnRosterUpdateNotification;
         CallService.OnCallCleanup += OnCallCleanup;
     }
}
```

## <a name="create-audio-and-video-sockets"></a>Создание аудио- и видеосокетов
Прежде чем бот сможет принимать входящие аудио- или видеозвонки Skype, ему необходимо создать объекты `AudioSocket` и `VideoSocket` для поддержки отправки и получения мультимедиа в реальном времени. (Если боту не требуется поддержка видео, то необходимо создать только AudioSocket.)

Необходимо заранее решить, какие модальности будет поддерживать бот, и создать соответствующие объекты AudioSocket и VideoSocket. После принятия входящего вызова поддерживаемые ботом модальности для данного вызова уже нельзя будет изменить.

Для каждого AudioSocket и VideoSocket указывается, будет ли бот поддерживать и отправку, и получение мультимедиа, или же только отправку или только получение. Например, боту необходимо отправить и получить аудиоданные (Sendrecv), но отправить только видеоданные (Sendonly). Необходимо также указать, какие форматы мультимедиа будут поддерживаться для каждого сокета мультимедиа. Поддерживаемый в настоящее время формат для AudioSocket — Pcm16K: (подписан), 16-разрядное PCM-кодирование, скорость выборки 16 кГц. Для VideoSocket форматы мультимедиа для отправки и получения задаются по отдельности. Для получения видео поддерживается только формат NV12, в то время как для отправки поддерживаются несколько разных форматов.

```cs
_audioSocket = new AudioSocket(new AudioSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    SupportedAudioFormat = AudioFormat.Pcm16K,
    CallId = correlationId
});

_videoSocket = new VideoSocket(new VideoSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    ReceiveColorFormat = VideoColorFormat.NV12,
    SupportedSendVideoFormats = new VideoFormat[]
    {
        VideoFormat.Yuy2_1280x720_30Fps,
        VideoFormat.Yuy2_720x1280_30Fps,
    },
    CallId = correlationId
});

_audioSocket.AudioMediaReceived += OnAudioMediaReceived;
_audioSocket.AudioSendStatusChanged += OnAudioSendStatusChanged;
_audioSocket.DominantSpeakerChanged += OnDominantSpeakerChanged;
_videoSocket.VideoMediaReceived += OnVideoMediaReceived;
_videoSocket.VideoSendStatusChanged += OnVideoSendStatusChanged;
```                             

## <a name="create-a-mediaconfiguration"></a>Создание MediaConfiguration
После создания сокетов мультимедиа бот должен создать объект `MediaConfiguration`, который необходим для связи сокетов мультимедиа с входящим аудио- или видеозвонком Skype.

```cs
var mediaConfiguration = MediaPlatform.CreateMediaConfiguration(_audioSocket, _videoSocket);
```

##  <a name="answer-an-incoming-audiovideo-call"></a>Ответ на входящий аудио- или видеозвонок
Событие `OnIncomingCallReceived` вызывается, чтобы бот мог принять входящий аудио- или видеозвонок Skype. Для этого бот создает объект `AnswerAppHostedMedia` с объектом `MediaConfiguration`. Бот подписывается на уведомления, связанные с этим вызовом Skype.

```cs
private Task OnIncomingCallReceived(RealTimeMediaIncomingCallEvent incomingCallEvent)
{
    // ... create Audio/VideoSocket objects and MediaConfiguration ...

    incomingCallEvent.RealTimeMediaWorkflow.Actions = new ActionBase[]
    {
        new AnswerAppHostedMedia
        {
            MediaConfiguration = mediaConfiguration,
            OperationId = Guid.NewGuid().ToString()
        }
    };

    // subscribe for roster and call state changes
    incomingCallEvent.RealTimeMediaWorkflow.NotificationSubscriptions = new NotificationType[]
    {
        NotificationType.CallStateChange,
        NotificationType.RosterUpdate
    };
}
```

## <a name="outcome-of-the-call"></a>Результат вызова
`OnAnswerAppHostedMediaCompleted` вызывается при завершении действия `AnswerAppHostedMedia`. Свойство `Outcome` в `AnswerAppHostedMediaOutcomeEvent` указывает на успешное или неуспешное завершение события. Если не удается установить вызов, бот должен ликвидировать объекты AudioSocket и VideoSocket, созданные им для вызова.

## <a name="receive-audio-media"></a>Получение аудиоданных
Если `AudioSocket` был создан с возможностью получения аудиоданных, то событие `AudioMediaReceived` будет вызываться каждый раз при получении кадра аудиоданных. Бот должен обрабатывать это событие приблизительно 50 раз в секунду, независимо от однорангового узла, который может быть источником аудиоданных (поскольку буферы устранения шума создаются локально, если аудиоданные не получены от однорангового узла). Каждый пакет аудиоданных доставляется в объекте `AudioMediaBuffer`. Этот объект содержит указатель на собственный буфер памяти, выделенный в куче, содержащий декодированные аудиоданные. 

```cs
void OnAudioMediaReceived(
            object sender,
            AudioMediaReceivedEventArgs args)
{
   var buffer = args.Buffer;

   // native heap-allocated memory containing decoded content
   IntPtr rawData = buffer.Data;            
}
```

Обработчик событий должен быстро возвращать данные. Рекомендуется, чтобы очередь приложения `AudioMediaBuffer` обрабатывалась асинхронно. События `OnAudioMediaReceived` будут сериализованы с помощью платформы мультимедиа в реальном времени (то есть следующее событие не будет вызываться до возврата текущего). Как только `AudioMediaBuffer` будет использовано, приложение должно вызвать метод Dispose буфера таким образом, чтобы базовая неуправляемая память была освобождена платформой мультимедиа. 

```cs
   // release/dispose buffer when done 
   buffer.Dispose();
```

> [!IMPORTANT]
> Бот не должен вызывать метод Dispose буфера до тех пор, пока не завершится сеанс доступа к буферу.

## <a name="receive-video-media"></a>Получение видеоданных
Получение видеоданных в целом схоже с получением аудиоданных, за исключением того, что число буферов в секунду в данном случае зависит от параметра частоты кадров. `VideoMediaBuffer` имеет свойства `VideoFormat` и `OriginalVideoFormat`. `OriginalVideoFormat` представляет исходный формат буфера, когда он использовался в качестве источника. Он доступен только при получении буферов видеоданных с помощью обработчика событий `IVideoSocket.VideoMediaReceived`. Если буфер был изменен до передачи, свойство `OriginalVideoFormat` будет иметь исходные значения ширины и высоты, тогда как `VideoFormat` будет иметь текущие значения ширины и высоты после изменения размера. Если свойства ширины и высоты `OriginalVideoFormat` отличаются от таковых свойства `VideoFormat`, объект-получатель `VideoMediaBuffer` события `VideoMediaReceived` должен изменить размер буфера в соответствии с размером `OriginalVideoFormat`. В настоящее время единственным форматом для получения видео служит NV12.

## <a name="send-audio-media"></a>Отправка аудиоданных
Если `AudioSocket` настроен для отправки мультимедиа, бот должен быть подписан на обработчик событий `AudioSendStatusChanged` для `AudioSocket` для получения уведомлений об изменениях состояния отправки. Боту следует приступать к отправке аудиоданных только после изменения состояния отправки на Active.

```cs
void AudioSocket_OnSendStatusChanged(
             object sender,
             AudioSendStatusChangedEventArgs args)
{
    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

В случае отправки аудиоданных предполагается, что аудиоданные PCM содержатся внутри собственного буфера, выделенного в куче. Бот должен быть производным от абстрактного класса `AudioMediaBuffer`. Данные отправляется асинхронно методом `Send` AudioSocket, и платформа вызывает `Dispose` для `AudioMediaBuffer` после завершения отправки. Бот не должен выпускать (или возвращать в распределитель пула) неуправляемые ресурсы, когда `Send` возвращается. Необходимо дождаться вызова `Dispose`.

## <a name="send-video-media"></a>Отправка видеоданных
Отправка видеоданных в целом схожа с отправкой аудиоданных. Бот должен подписаться на событие `VideoSendStatusChanged` и дождаться состояния `Active` для `MediaSendStatus`. Бот не должен освобождать или выпускать неуправляемые ресурсы буфера до вызова метода `Dispose`. Поддерживаются следующие форматы цвета: RGB24, NV12 и Yuy2.

При поддержке сразу нескольких `VideoFormat` для отправки видеоданных `VideoFormat`, являющийся предпочтительным, передается в бот с помощью события `VideoSendStatusChanged`. Предпочтительный `VideoFormat` для отправки видеоданных может изменяться в зависимости от пропускной способности сети или изменения размера окна видео одноранговым клиентом.

```cs
void VideoSocket_OnSendStatusChanged(
            object sender,
            VideoSendStatusChangedEventArgs args)
{
    VideoFormat preferredVideoFormat;

    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        // bot is recommended to use this format for sourcing video content.
        preferredVideoFormat = args.PreferredVideoSourceFormat;
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

Каждый `VideoMediaBuffer` имеет свойство VideoFormat для указания формата видеоданных для этого буфера. Хотя свойство `VideoFormat` не должно соответствовать свойству `PreferredVideoSourceFormat`, указанному в событии `VideoSendStatusChanged`, настоятельно рекомендуется использовать указанный `PreferredVideoSourceFormat` во избежание ресурсоемких циклов ЦП, связанных с изменением размера кадра видео.

## <a name="call-notifications"></a>Уведомления о вызове
### <a name="call-state-changes"></a>Изменения состояния вызова
Подписавшись на `NotificationType.CallStateChange` в `NotificationSubscriptions` объекта `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow`, бот может получать уведомления об изменении состояния вызова.

```cs
private Task OnCallStateChangeNotification(CallStateChangeNotification callStateChangeNotification)
{
    if (callStateChangeNotification.CurrentState == CallState.Terminated)
    {   
        // stop sending media and dispose the media sockets
        _audioSocket.Dispose();
        _videoSocket.Dispose();
    }

    return Task.CompletedTask;
}
 ```

### <a name="roster-update"></a>Обновление списка
Если бот будет добавлен в конференцию, он может ожидать передачу данных для списка конференции, подписавшись на `NotificationType.RosterUpdate` в `NotificationSubscriptions` объекта `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow`. `RosterUpdateNotification` содержит сведения о всех участниках конференции. Бот может ожидать уведомления от участника, отправляющего видео, а затем подписать (`Subscribe`) один из своих объектов `VideoSocket` на видео участника.

```cs
private async Task OnRosterUpdateNotification(RosterUpdateNotification rosterUpdateNotification)
{
    // Find a video source in the conference to subscribe
    foreach (RosterParticipant participant in rosterUpdateNotification.Participants)
    {
        if (participant.MediaType == ModalityType.Video
            && (participant.MediaStreamDirection == "sendonly" || participant.MediaStreamDirection == "sendrecv")
            )
        {
            var videoSubscription = new VideoSubscription
            {
                ParticipantIdentity = participant.Identity,
                OperationId = Guid.NewGuid().ToString(),
                SocketId = _videoSocket.SocketId,
                VideoModality = ModalityType.Video,
                VideoResolution = ResolutionFormat.Hd720p
            };

            await CallService.Subscribe(videoSubscription).ConfigureAwait(false);
            break;
        }
    }  
}
```

## <a name="end-the-call"></a>Завершение вызова

### <a name="handle-call-termination-from-the-caller"></a>Обработка завершения вызова от вызывающего абонента
Когда пользователь разъединяет вызов к боту в одноранговой беседе или когда бот удаляют из нее, он получает уведомление об изменении состояния вызова с CallState на Terminated. Боту необходимо освободить все созданные им сокеты мультимедиа.

### <a name="terminate-the-call-from-the-bot"></a>Прерывание вызова от бота
Бот может завершить вызов, вызывая `EndCall` для `IRealTimeMediaCallService`.

### <a name="handle-call-clean-up-by-the-bot-framework"></a>Обработка очистки вызовов с помощью Bot Framework
При ошибке (например, если `AnswerAppHostedMediaOutcomeEvent` не был получен в течение приемлемого времени) Bot Framework может завершить вызов. Бот должен подписаться на событие `OnCallCleanup` и освободить сокеты мультимедиа.

