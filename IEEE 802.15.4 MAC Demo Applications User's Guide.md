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
