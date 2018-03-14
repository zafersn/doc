# doc

Uygulama başlatma, aşağıda gösterilen App_init işlevinde yapılır:
```
void App_init( void )
{
    mAppEvent = OSA_EventCreate(TRUE);
    /* The initial application state */
    gState = stateInit;
    /* Reset number of pending packets */
    mcPendingPackets = 0;
    
    /* Prepare input queues.*/
    MSG_InitQueue(&mMlmeNwkInputQueue); 
    MSG_InitQueue(&mMcpsNwkInputQueue);
    
    /* Initialize the MAC 802.15.4 extended address */
    Mac_SetExtendedAddress( (uint8_t*)&mExtendedAddress, macInstance );

    /* register keyboard callback function */
    KBD_Init(App_HandleKeys);
    
    /* Initialize the serial terminal interface so that we can print out status messages */
    Serial_InitInterface(&interfaceId, APP_SERIAL_INTERFACE_TYPE, APP_SERIAL_INTERFACE_INSTANCE);
    Serial_SetBaudRate(interfaceId, gUARTBaudRate115200_c);
    Serial_SetRxCallBack(interfaceId, UartRxCallBack, NULL);
    
    /*signal app ready*/  
    LED_StartSerialFlash(LED1);
    
    Serial_Print(interfaceId, "\n\rPress any switch on board to start running the application.\n\r", gAllowToBlock_d);  
}
```
PHY, MAC ve tüm çerçeve başlatıldıktan sonra App_init () işlevi main_task () işlevi içinden çağrılır:
```
hardware_init();

MEM_Init();

TMR_Init();

LED_Init();

SerialManager_Init();

Phy_Init();

RNG_Init(); /* RNG must be initialized after the PHY is Initialized */
MAC_Init();

/* Bind to MAC layer */
macInstance = BindToMAC( (instanceId_t)0 );
Mac_RegisterSapHandlers( MCPS_NWK_SapHandler, MLME_NWK_SapHandler, macInstance );
App_init();

```
Başlatma ve kurulumdan sonra, uygulama, kullanıcının Koordinatör panosundaki bir anahtara basmasını bekler. Klavye İşleyicisi, App_HandleKeys, uygulama görevini, uygulamayı başlatan bir gAppEvtDummyEvent_c olayı gönderecektir.

## 2.4.2. Resetting

Bu noktadan sonra, MAC (ve ayrıca PHY) katmanını aşağıdaki kodda gösterildiği gibi "MLME-RESET.request" servisini kullanarak sıfırlamak her zaman güvenlidir.

```
mlmeMessage_t mlmeReset;
/* Create and execute the Reset request */
mlmeReset.msgType = gMlmeResetReq_c;
mlmeReset.msgData.resetReq.setDefaultPib = TRUE;
(void)NWK_MLME_SapHandler( &mlmeReset, macInstance );

```

Bu hizmeti çağırarak MAC katmanı sıfırlanır ve MAC_Init () çağrıldıktan hemen sonra olduğu gibi aynı duruma getirilir. SetDefaultPib parametresi MAC katmanına PIB özelliklerinin varsayılan değerlerine (IEEE 802.15.4 Standardında belirtildiği gibi) ayarlanıp ayarlanmayacağını veya sıfırlamadan sonra değişmeden kalması gerektiğini söyler. Sıfırlama ilkelini talep ederken, sıfırlama çağrısı senkronize edilir. Yani, sıfırlama isteğinde bir onay mesajı yoktur ve NWK_MLME_SapHandler () işlevi gSuccess_c'yi döndürür. Arama senkron olduğundan, mesaj yapısının MEM_BufferAlloc () aracılığıyla ayrılmasına gerek yoktur, ancak yığına ayrılabilir.
Her koşulda, mesajı dağıtmak için çağrı yapan kuruluşun sorumluluğu budur. MAC'yi MAC_Init () çağrıldıktan hemen sonra sıfırlamanın gerekli olmadığına dikkat edin.

MyWirelessApp Demo Framework projesinin uygulama kaynak kodu MLME-RESET kullanarak bir cihazı sıfırlamak için herhangi bir kod örneği içermemektedir. istek. Bununla birlikte, MAC ve PHY katmanlarını bilinen bir duruma geri getirmenin iyi tanımlanmış bir yoludur.

## 2.4.3. SAP'leri uygulamak

Projeyi derleyebilmeden önce, mesajları MAC katmanından sonraki üst katmana işleyen SAP işleyicileri oluşturmak gerekir. Şimdilik boş işlevler oluşturun ve bu kodda gösterildiği gibi daha sonra işlevselliği uygulayın.

```
/* The following functions are called by the MAC
to put messages into the Application's queue.
They need to be defined even if they are not
used in order to avoid linker errors. */
resultType_t MLME_NWK_SapHandler(nwkMessage_t *pMsg , instanceId_t instanceId)
{
/* This only serves as a temporary way of avoiding a compiler warning.
Later this will be changed so that we return gSuccess_c instead. */
return gSuccess_c;
}
resultType_t MCPS_NWK_SapHandler(mcpsToNwkMessage_t *pMsg , instanceId_t instanceId)
{
/* This only serves as a temporary way of avoiding a compiler warning.
Later this will be changed so that we return gSuccess_c instead. */
return gSuccess_c;
}
```

Tipik olarak, SAP işleyicileri iletileri sıraya göre tamponlar ve daha sonra bunları işlemek için uygulama görevine bir olay gönderir.

## 2.4.4. Bir PAN başlatmak ve katılmak

MAC, PHY ve çerçeve katmanları başlatıldıktan sonra, MAC katmanının MCPS ve MLME servislerine erişmek artık güvenlidir. MLME'ye erişerek başlayın. Bunun için örnek kod, MyWirelessApp Demo Non Beacon projesinde, App.c kaynak dosyasında bulunur.
Üniteler artık bir PAN başlatmaya hazırdır. İlk olarak, bir PAN koordinatörü kurun, çünkü tüm IEEE 802.15.4 Standart PAN'ların PAN koordinatörü olması gerekir.

## 2.4.5. Çalışan Alan ağı algılama taraması

PAN koordinatörünün gerçekleştirmesi gereken ilk görev, PAN için hangi radyo frekansının kullanılacağını seçmektir. Bu mantıksal kanalı seçerek denir. Mantıksal kanal önceden tanımlanmış bir değere sahip olabilir, ancak daha iyi bir yöntem, diğer birimler tarafından kullanılmayan bir kanalın seçilmesidir. Bu amaçla, PAN koordinatörünün tüm (veya seçili) kanalları taramak için kullanabileceği ilkeller vardır. Buna energy detection taraması denir (ED taraması). Aşağıdaki kod bu görevi gösterir.

```
static uint8_t App_StartScan(macScanType_t scanType, uint8_t appInstance)
{
mlmeMessage_t *pMsg;
mlmeScanReq_t *pScanReq;
Serial_Print(interfaceId,"Sending the MLME-Scan Request message to the MAC...",
gAllowToBlock_d);
/* Allocate a message for the MLME (We should check for NULL). */
pMsg = MSG_AllocType(mlmeMessage_t);
if(pMsg != NULL)
{
/* This is a MLME-SCAN.req command */
pMsg->msgType = gMlmeScanReq_c;
/* Create the Scan request message data. */
pScanReq = &pMsg->msgData.scanReq;
/* gScanModeED_c, gScanModeActive_c, gScanModePassive_c, or gScanModeOrphan_c */
pScanReq->scanType = scanType;
/* ChannelsToScan */
#ifdef gPHY_802_15_4g_d
pScanReq->channelPage = gChannelPageId9_c;
pScanReq->scanChannels[0] = mDefaultValueOfChannel_c;
#else
pScanReq->scanChannels = mDefaultValueOfChannel_c;
#endif
/* Duration per channel 0-14 (dc). T[sec] = (16*960*((2^dc)+1))/1000000.
A scan duration of 3 on 16 channels approximately takes 2 secs. */
pScanReq->scanDuration = 3;
/* Don't use security */
pScanReq->securityLevel = gMacSecurityNone_c;
/* Send the Scan request to the MLME. */
if( NWK_MLME_SapHandler( pMsg, macInstance ) == gSuccess_c )
{
Serial_Print(interfaceId,"Done\n\r", gAllowToBlock_d);
return errorNoError;
}
else
{
Serial_Print(interfaceId,"Invalid parameter!\n\r", gAllowToBlock_d);
return errorInvalidParameter;
}
}
else
{
/* Allocation of a message buffer failed. */
Serial_Print(interfaceId,"Message allocation failed!\n\r", gAllowToBlock_d);
return errorAllocFailed;
}
}
```

Bu durumda, App_StartScan için scanType parametresi gScanModeED_c olarak ayarlanmalıdır. Diğer tarama türleri bu bölümde daha sonra açıklanmaktadır. PAN koordinatörü o kanalı taradığında kanalda herhangi bir etkinlik varsa, bu tarama sonucunda ortaya çıkar. Tarama sırasında hiçbir etkinlik belirtisi göstermeyen bir kanal, yaklaşık 0x00'lük bir enerji seviyesini gösterir. Sayı ne kadar yüksekse, o kanalda daha fazla aktivite tespit edildi.

Önceki kod örneği, 2.4 GHz bandında mevcut 16 kanalın tamamını tarar. Parametre scanChannels, ayarlanan her bitin bu kanalın taranması gerektiğini belirten bir bit maskesidir.
2.4 GHz bandı 11 ila 26 arasındaki kanal numaralarını içerdiğinden, 11 - 26 bitleri scanChannels bit maskesinde ayarlanır. Alt kanal numaraları göz ardı edilir. Ayrıca, scanDuration parametresi MAC'a her kanalı ne kadar sürede tarayacağını söyler. 0 ile 14 arasındaki sayılar bu parametre için geçerli girişlerdir. Sayı ne kadar yüksekse, tarama süresi o kadar uzun olur. Her kanal için tam tarama süresi bu denklem kullanılarak hesaplanabilir:
```
Scan duration = 15.36 ms · (2scanDuration + 1)
```

Örnekte scanDuration parametresi, 16 kanalın her birinde yaklaşık 0,5 saniyeliğine tarama yapılmasını ister. Tarama isteği mesajı MLME'ye başarılı bir şekilde gönderildikten sonra, tarama başlatılır ve bir tarama onayı (msgType == gNwkScanCnf_c ile bir nwkMessage_t struct), MLME'den NWK'ye gönderilen mesajlar için SAP işleyicisinde asenkronize olarak alınır. Aşağıdaki kod tarama onay mesajını işler.
