# cJSON Parse ve Create işlemleri

Burada 4 adet örnek olacak bu örneklerden ikisi array şeklinde object yapısını oluşturma ve parse etme işlemi diğer ikisi sadece object yapısını oluşturma ve parase etme işlemi.
Biraz daha açıklarsak.
Bu yapıyı ibm blumix bulut tarafında kullanmak için yapmamız gerekmekte.


**Requests**<br>
Requests are formatted as shown in the following code sample:

```
{  "d": {...}, "reqId": "b53eb43e-401c-453c-b8f5-94b73290c056" }
```
**Responses**<br>
Responses are formatted as shown in the following code sample:
```
{
            "rc": 0,
            "message": "success",
            "d": {...},
            "reqId": "b53eb43e-401c-453c-b8f5-94b73290c056"
}
```
BAKINIZ: [IBM BLUMIX Message format](https://console.bluemix.net/docs/services/IoT/devices/mqtt.html#mqtt)

Neyse bu kısa ayrıntıyı geçtikten sonra şimdi ibm blumix te kullanacağımız yapıya göre ilk örneğimizi yapalım. Yani bu ilk örnek array'sız object oluşturma.

# Create (Arraysız)

```
{
    "name": "test1",
    "d": 
        {
            "start":0,
            "mode":1,
            "fanSpeed":1,
            "setPoint":250,
            "roomTemp":252
            
        }
}        
```
Şimdi bu yukarıdaki yapıyı oluşturalım.

## Printing

```
char* create_data(void)
{
	const unsigned int data_numbers[1][6] = {
	        {0, 1, 1, 250, 252,33}

	    };

	 	char *stringPrint = NULL;
	    cJSON *commandName = NULL;
	    cJSON *data = NULL;
	    cJSON *start = NULL;
	    cJSON *mode = NULL;
	    cJSON *fanSpeed = NULL;
	    cJSON *setPoint = NULL;
	    cJSON *roomTemp = NULL;


	    cJSON *monitor = cJSON_CreateObject();
	        if (monitor == NULL)
	        {
	            goto end;
	        }
	    commandName = cJSON_CreateString("get");
	            if (commandName == NULL)
	            {
	                goto end;
	            }

	   cJSON_AddItemToObject(monitor, "name", commandName);

		data = cJSON_CreateObject();
        if (data == NULL)
        {
            goto end;
        }
        cJSON_AddItemToObject(monitor, "d", data);

        start = cJSON_CreateNumber(data_numbers[0][0]);
        if (start == NULL)
        {
            goto end;
        }
        cJSON_AddItemToObject(data, "start", start);

        mode = cJSON_CreateNumber(data_numbers[0][1]);
        if (mode == NULL)
        {
            goto end;
        }
        cJSON_AddItemToObject(data, "mode", mode);

        fanSpeed = cJSON_CreateNumber(data_numbers[0][2]);
       	                	        if (fanSpeed == NULL)
       	                	        {
       	                	            goto end;
       	                	        }
       cJSON_AddItemToObject(data, "fanSpeed", fanSpeed);

     setPoint = cJSON_CreateNumber(data_numbers[0][3]);
								if (setPoint == NULL)
								{
									goto end;
								}
   cJSON_AddItemToObject(data, "setPoint", setPoint);

   roomTemp = cJSON_CreateNumber(data_numbers[0][4]);
  														if (roomTemp == NULL)
  														{
  															goto end;
  														}
   cJSON_AddItemToObject(data, "roomTemp", roomTemp);

    stringPrint = cJSON_Print(monitor);
    if (stringPrint == NULL)
    {
        fprintf(stderr, "Failed to print monitor.\n");
    }

end:
    cJSON_Delete(monitor);
    return stringPrint;
}
```

Eveet bu yapıyı oluşturmuş olduk. Birde böyle bir data geldini varsayalım ve bu datayı parçalayalım(parse) edelim.

## Parsing

```
int parse_data(const char * const monitor)
{
     const cJSON *data = NULL;
     const cJSON *name = NULL;
    int status = 0;
    cJSON *monitor_json = cJSON_Parse(monitor);
    if (monitor_json == NULL)
    {
        const char *error_ptr = cJSON_GetErrorPtr();
        if (error_ptr != NULL)
        {
            fprintf(stderr, "Error before: %s\n", error_ptr);
        }
        status = 0;
        goto end;
    }
    name = cJSON_GetObjectItemCaseSensitive(monitor_json, "name");
    if (cJSON_IsString(name) && (name->valuestring != NULL))
    {
        printf("Checking monitor \"%s\"\n", name->valuestring);
    }


    data = cJSON_GetObjectItemCaseSensitive(monitor_json, "d");
   
    int start = cJSON_GetObjectItemCaseSensitive(data, "start")->valueint;
	int mode = cJSON_GetObjectItemCaseSensitive(data, "mode")->valueint;
	int fanSpeed = cJSON_GetObjectItemCaseSensitive(data, "fanSpeed")->valueint;
	int setPoint = cJSON_GetObjectItemCaseSensitive(data, "setPoint")->valueint;
	int roomTemp = cJSON_GetObjectItemCaseSensitive(data, "roomTemp")->valueint;
	printf("start: %d mode:%d fanspeed:%d setpoint: %d\n",start,mode,fanSpeed,setPoint);
    /*if (!cJSON_IsNumber(start) || !cJSON_IsNumber(mode))
    {
        status = 0;
        goto end;
    }*/
end:
    cJSON_Delete(monitor_json);
    return status;
}
```
Bu işlem bu kadar. Şimdi birde lazım olursa diye. Array şeklinde data yapısı oluşturalım.
```
{
    "name": "Awesome 4K",
    "resolutions": [
        {
            "width": 1280,
            "height": 720
        },
        {
            "width": 1920,
            "height": 1080
        },
        {
            "width": 3840,
            "height": 2160
        }
    ]
}
```
Şimdi yukarıdaki örnek data yapısını oluşturalım:

```
//create a monitor with a list of supported resolutions
char* create_monitor(void)
{
    const unsigned int resolution_numbers[3][2] = {
        {1280, 720},
        {1920, 1080},
        {3840, 2160}
    };
    char *string = NULL;
    cJSON *name = NULL;
    cJSON *resolutions = NULL;
    cJSON *resolution = NULL;
    cJSON *width = NULL;
    cJSON *height = NULL;
    size_t index = 0;

    cJSON *monitor = cJSON_CreateObject();
    if (monitor == NULL)
    {
        goto end;
    }

    name = cJSON_CreateString("Awesome 4K");
    if (name == NULL)
    {
        goto end;
    }
    /* after creation was successful, immediately add it to the monitor,
     * thereby transfering ownership of the pointer to it */
    cJSON_AddItemToObject(monitor, "name", name);

    resolutions = cJSON_CreateArray();
    if (resolutions == NULL)
    {
        goto end;
    }
    cJSON_AddItemToObject(monitor, "resolutions", resolutions);

    for (index = 0; index < (sizeof(resolution_numbers) / (2 * sizeof(int))); ++index)
    {
        resolution = cJSON_CreateObject();
        if (resolution == NULL)
        {
            goto end;
        }
        cJSON_AddItemToArray(resolutions, resolution);

        width = cJSON_CreateNumber(resolution_numbers[index][0]);
        if (width == NULL)
        {
            goto end;
        }
        cJSON_AddItemToObject(resolution, "width", width);

        height = cJSON_CreateNumber(resolution_numbers[index][1]);
        if (height == NULL)
        {
            goto end;
        }
        cJSON_AddItemToObject(resolution, "height", height);
    }

    string = cJSON_Print(monitor);
    if (string == NULL)
    {
        fprintf(stderr, "Failed to print monitor.\n");
    }

end:
    cJSON_Delete(monitor);
    return string;
}
```
Böyle bir dataya ayrıştırma işlemi uygulayalım.

```
/* return 1 if the monitor supports full hd, 0 otherwise */
int supports_full_hd(const char * const monitor)
{
    const cJSON *resolution = NULL;
    const cJSON *resolutions = NULL;
    const cJSON *name = NULL;
    int status = 0;
    cJSON *monitor_json = cJSON_Parse(monitor);
    if (monitor_json == NULL)
    {
        const char *error_ptr = cJSON_GetErrorPtr();
        if (error_ptr != NULL)
        {
            fprintf(stderr, "Error before: %s\n", error_ptr);
        }
        status = 0;
        goto end;
    }

    name = cJSON_GetObjectItemCaseSensitive(monitor_json, "name");
    if (cJSON_IsString(name) && (name->valuestring != NULL))
    {
        printf("Checking monitor \"%s\"\n", name->valuestring);
    }

    resolutions = cJSON_GetObjectItemCaseSensitive(monitor_json, "resolutions");
    cJSON_ArrayForEach(resolution, resolutions)
    {
        cJSON *width = cJSON_GetObjectItemCaseSensitive(resolution, "width");
        cJSON *height = cJSON_GetObjectItemCaseSensitive(resolution, "height");

        if (!cJSON_IsNumber(width) || !cJSON_IsNumber(height))
        {
            status = 0;
            goto end;
        }

        if ((width->valuedouble == 1920) && (height->valuedouble == 1080))
        {
            status = 1;
            goto end;
        }
    }

end:
    cJSON_Delete(monitor_json);
    return status;
}
```
ve işlemler tamam. Detaylı bilgi için bakınız:

KAYNAKLAR: 

(https://github.com/DaveGamble/cJSON/blob/7cc52f60356909b3dd260304c7c50c0693699353/README.md)
