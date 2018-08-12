# BBPOS - SDK Integration
SDK for integrating with the BBPOS Device, within Android Application.

## Table of contents
* [Requirements](#requirements)
* [Downloads](#downloads)
* [Installation](#installation)
* [Getting Started](#getting-started)
* [Bluetooth Callback Parameters](#bluetooth-callback-parameters)
* [Start EMV](#start-emv)
* [BBCommunication (Integration Functions)](#bbcommunication)
    * [Javadoc functions](#javadoc-functions)
    * [Declaration](#declaration)
    * [Initilaize callback](#initilaize-callback)
    * [onCreate - Attach callbacks](#attach-callbacks)
    * [onResume - Re-attach callbacks](#declaration)
    * [Search & Connect BT](#search-&-connec)
    * [Start EMV](#startamoun)
    * [Load Params](#load-param)
    


## Requirement
- BBPos SDK AAR [Min version of the SDK is : 17]
- BBPos (mPos) device.
- Android Host Application.
    - targetSdkVersion XX Status is: [installed] at the SDK Manager 
        ```
            |__SDK Platforms
                |__Android API XX
                    |                           API-LEVEL   Revision        Status
                    |__Android SDK Platform XX    [xx]        [y]         [installed]
                    |__Sources for Android  XX    [xx]        [y]         [installed]
        ```

## Downloads
- BBPos SDK AAR - - [Download Here](https://yaadpay.docs.apiary.io)

## Installation
Place the AAR under :
```
|__PROJECT_NAME_FOLER
    |__app
        |___build
        |__src
        |__libs
            |__bbpos.aar
```
Install using Gradle (Module:app) :
``` java
    android
    {
        ...
        defaultConfig {
            minSdkVersion 17
        }
        ...
    }

    repositories {   
        mavenCentral()
        flatDir {   
            dirs 'libs'   
        }   
    }   

    dependencies {   
      implementation fileTree(dir: 'libs', include: ['*.jar'])  
      implementation 'com.android.support:appcompat-v7:26.1.0'   
      implementation(name: 'bbpos', ext: 'aar')
    }   
```

<strong>Please NOTE: Configuration 'compile' is obsolete and has been replaced with 'implementation' and 'api'. <br>
It will be removed at the end of 2018.</strong>


After adding all these to the gradle press <a>Sync Now</a> or <a>Try Again.</a>


## Getting Started

Getting started with the BBPOS API couldn't be easier.

You need to implement the BBPosCallback class to your Activity and Override onBTResponse.

``` java
public class MainActivity extends Activity implements BBPosCallback
{
  ..
  
    @Override
    public void onBTResponse(byte statusCode, String data)
    {
       switch (statusCode)
        {
            case BBUtilCallBacks.BT_CONNECTED:
                String SERIAL_NUMBER = data;
                getTokenFromServer(SERIAL_NUMBER);
                break;
            case BBUtilCallBacks.BT_DISCONNECTED:
                break;
            case BBUtilCallBacks.BT_FAILD_START:
                break;                
        }
    }
    
    private void getTokenFromServer(String bbposSerial){
        //TODO: Request token from Server API: (@link)[http://yaadpay.yaad.net]
        String tokenFromServer = "GOT_TOKEN_FROM_SERVER";

        //After we get the token we update the library
        bbCommunication.updateToken(tokenFromServer);
    }
    
  ...
  
}
```
- <strong>onBTResponse</strong> - Callback notifier for the Bluetooth state.
- <strong>getTokenFromServer:</strong> In order to get the token you need to make a Request to our server: <br> [Get Token From Server API](https://yaadpay.docs.apiary.io).

#### Bluetooth Callback Parameters 

| Parameter | Description | Data
| ------ | ------ | ------ |
| BBPosCallback.BT_CONNECTED | Bluetooth connected successfully. | Serial_Number |
| BBPosCallback.BT_DISCONNECTED | User is no longer connected. | NULL |
| BBPosCallback.BT_FAILD_START | Couldnt scan for devices. | NULL |

When Bluetooth is connected to the BBPOS Device it is recommanded to save locally the Serial Number, and use it in order to get Token From Yaad Payments servers.


## Start EMV
### Initiate Transaction

``` java
 private int RESULT_CODE = 1111;
 private void populateTransaction(HashMap<String, String> tranHash , boolean flag)
    {
        HashMap<String, String> hash = new HashMap<>();
        if(flag){
        //InputObj accourding to the - SHVA File
        hash.put("inputObj.amount", "100");
        hash.put("inputObj.cashbackAmount", "0");
        hash.put("inputObj.tranType", "01");
        hash.put("inputObj.currency", "376");

        //New lines for backup
        hash.put("yaadObj.notify", "url");
        hash.put("yaadObj.notifyURL", "http://www.mysite.com/emv_trans.php?id=1234");
        }else{
            hash.putAll(tranHash);
        }
        int startEmvResult = BBCommunication.getInstance().startAmount(MainActivity.this, hash, RESULT_CODE);
        if(startEmvResult != BBPosCallback.SUCCESS)
        {
            switch (startEmvResult)
            {
                case BBPosCallback.FAILED:
                    Toast.makeText(this, "Need to load params!", Toast.LENGTH_SHORT).show();
                    loadParametersUsage();
                    break;
                case BBPosCallback.BT_DISCONNECTED:
                    Toast.makeText(this, "Bt is not connected", Toast.LENGTH_SHORT).show();
                    break;
            }
        }
    }

```

- <strong> loadParametersUsage: </strong> When BBPosCallback.FAILED is returned we need to load parameters in order to initiate transaction.


| Parameter | Description | Example |
| ------ | ------ | ------ |
| tranHash | Hashmap that contains transaction details according to SHVA document. | "inputObj.amount",<br> "inputObj.cashbackAmount",<br>  "inputObj.currency",<br>.. etc' <br>  |
| flag | For example usage only, <br> <br> <strong>True:<br> Use premade HashMap. <br> <br> False:<br> Use given HashMap.</strong>  | True / False  |
| startAmount | Trigger the transaction from the BBCommunication class in the SDK  |  [Here](#startamount)  |



## BBCommunication

### JavaDoc functions
All functions can be found in the JavaDocs: [Here](https://yaadpay.yaad.net)

### Declaration
```java
    /** Variable that communicates with our library thgough commands **/
    private BBCommunication bbCommunication;
```
### Initilaize callbacks 
``` java

    private void attachCallbacks(){
        /** Get the current instance of our BBCommunication - Singleton **/
        bbCommunication = BBCommunication.getInstance();

        /** Initiate callbacks to this activity **/
        bbCommunication.init(this, this);
    }
``` 
### onCraete
#### Attach Callbacks
``` java
    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ...
        attachCallbacks();
        ..
    }
``` 
### onResume 
#### Re-Attach Callbacks 
``` java
    
    @Override
    protected void onResume()
    {
        super.onResume();
        /** Reattach callbacks to the current activity **/
        bbCommunication = BBCommunication.getInstance();
        bbCommunication.init(MainActivity.this, this);
        
        //Optional - Search for connection if device disconnected
        connectBt();
    }
```
### Search & Connect 
```java
    private void connectBt()
    {
        /** Checks if the device is not connected before searching **/
        if (!bbCommunication.isBtConnected())
        {
            bbCommunication.connectBT(MainActivity.this);
        }
    }
```

### Disconnect 
```java
 private void disconnectBt()
    {
        bbCommunication.disconnectBt(this);
    }
```

### StartAmount 
```java
    private int RESULT_CODE = 1111;
    private void startEmvTransaction(HashMap<String, String> tranHash)
    {
        int startEmvResult = BBCommunication.getInstance().startAmount(MainActivity.this, tranHash, RESULT_CODE);
        switch (startEmvResult)
            {
                case BBPosCallback.FAILED:
                break;
                case BBPosCallback.BT_DISCONNECTED:
                break;                
            }
    }
```
#### startAmount - function
| Parameter | Description | Example |
| ------ | ------ | ------ |
| Context context | Callback reference | MainActivity.this |
| HashMap<String,String> hash | Hashmap that contains transaction details according to SHVA document. | "inputObj.amount",<br> "inputObj.cashbackAmount",<br>  "inputObj.currency",<br>.. etc' <br>  |
| hash.put("yaadObj.XXX","XXX") | Unique param that can be passed within the HashMap in the initialize transaction and pass along with the transaction - Usage: Client info [id, name, email etc'] and will be shown in the result (without changes) | yaadObj.id="205654912",<br> yaadObj.name="TEST",<br> yaadObj.email="test@testserver.com",<br> yaadObj.XXX =... etc'  |
| int RESULT_CODE | onActivityResult int code  | RESULT_CODE = 1111 |


#### StartAmount has 2 callbacks:

####  <br>1) Res Integer callback.
###### Integer callback that state whether the Transaction Activity has successfully opened or not.
#### startEmvResult 
| Parameter | Description |  
| ------ | ------ |
| BBPosCallback.SUCCESS | Transaction opened successfully.<br><br> <strong>Can be ignored and handle only fail statuses.</strong>  |
| BBPosCallback.FAILED | Need to load parameters from the server |
| BBPosCallback.BT_DISCONNECTED | Bluetooth is disconnected, can not initate transaction |
####  <br>2) Activity For Results
###### When the transaction is done, and activity for result will be triggered.
```java
  @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == RESULT_CODE) {
            if(resultCode == Activity.RESULT_OK){
                String results = data.getStringExtra(TranManagerActivity.EXTRA_RESULT);
            }
            if (resultCode == Activity.RESULT_CANCELED) {
                //Write your code if there's no results
            }

        }
    }
```
#### Example string result:
``` java
             onActivityResult:
             {
                "uid":"17122610103608822862891",
                "LastAction":"True",
                "ashStatusDes":"תקין",
                "Status":"0",
                "id":"38931",
                "tranRecord":"MTcxMjI2MTAxMDM2MDg4MjI4NjI4OTG0G4Ca+PgcaPJ78E2IWsOCVsvMVRm530qXPwLXbFPf23pRZmW8G4BIIj\/ivsKSgrAEOtctPtAw9OBwPIAlYzxS1bxo953O0XAVuPasZDcjHlkOjCmc38ldTqZyxPoJzCiPRmp1M0sF8QVaF95DlaqaGsYJz0+y\/0ESgdcxIhl0FDZhhUNkI7yusnz0LfMitv5Mmh8dPOqcMLzgKJgcc+pduYZrs+KP\/8gOE2QuzhubMpTbMPiJJ6USeDRBMS8fjfxGQ851hL9Js9CAfkNr1NiT6KDf6nQJRCNAg+U2KzAwUJ8T77swRuepODRL+Y\/fHh9DMROnkVQIrA5XgA==",
                "SOAPAction":"AshFull",
                "WSKEY":"3BA6B7465E611EEEE1DDBD43DC2563C7B05188874917EBFAB61AA77D2BEE0649",
                "yaadObj.XXX" : "Client-Info"
             }
```
| Parameter | Description | Example |
| ------ | ------ | ------ |
| uid | SHVA UID for this transaction| 17122610103608822862891 |
| LastAction | Was the last action success or not | True / False |
| ashStatusDes | Description of the transacion result, according to SHVA Document | "תקין" - for success (0) <br> "שגיאה בחיבור לשבא" for connection to shva lost (999)<br>... etc'|
| id | Unique Id record in YaadPayments database - Used for getting full transaction details from Yaad servers | 38931 |
| tranRecord | SHVA Tran record  | MTcxMjI2MTAxMDM2MDg4MjI4NjI4OTG0G4Ca+PgcaPJ78E2IWsOCVsvMVRm530qXPwLXbFPf23pRZmW8G4BIIj\/ivsKSgrAEOtctPtAw9OBwPIAlYzxS1bxo953O0XAVuPasZDcjHlkOjCmc38ldTqZyxPoJzCiPRmp1M0sF8QVaF95DlaqaGsYJz0+y\/0ESgdcxIhl0FDZhhUNkI7yusnz0LfMitv5Mmh8dPOqcMLzgKJgcc+pduYZrs+KP\/8gOE2QuzhubMpTbMPiJJ6USeDRBMS8fjfxGQ851hL9Js9CAfkNr1NiT6KDf6nQJRCNAg+U2KzAwUJ8T77swRuepODRL+Y\/fHh9DMROnkVQIrA5XgA== |
| SOAPAction | The action that was performed in the web-transaction | AshFull / AshEnd |
| WSKEY | WebSocket Key - Used for getting bucket of transactions from Yaad servers | 3BA6B7465E611EEEE1DDBD43DC2563C7B05188874917EBFAB61AA77D2BEE0649 |
|yaadObj.XXX | Unique param that can be passed within the HashMap in the initialize transaction and pass along with the transaction - Usage: Client info [id, name, email etc'] and will be shown in the result (without changes) | yaadObj.id="205654912",<br> yaadObj.name="TEST",<br> yaadObj.email="test@testserver.com",<br> yaadObj.XXX =... etc'  |

<br>
### Load Params 
```java
   private void loadParametersFunction(LoadParamsCallback loadParamsCallback, boolean force)
    {
        //{1} Integer callback that state whether the Loading Activity has successfully opened or not.
        int res =  BBCommunication.getInstance().loadParams(this, force, loadParamsCallback);
        switch(res){
            case BBPosCallback.DEVICE_BUSY:
                Toast.makeText(this, "Device is busy.", Toast.LENGTH_SHORT).show();
                break;
            case BBPosCallback.BT_DISCONNECTED:
                Toast.makeText(this, "Do stuff", Toast.LENGTH_SHORT).show();
                break;
        }
    }
    
    private void loadParametersUsage(){
      // {2} Custom Callback [LoadParamsCallback] that indicates if the loading was success or not.
       loadParametersFunction(new LoadParamsCallback()
            {
                @Override
                public void onParamLoaded(byte statusResult)
                {
                    switch(statusResult){
                        case LoadParamsCallback.LOAD_PARAM_FAILED:
                            Toast.makeText(MainActivity.this, "Load Param Couldnt load try again! #NOTE Not automatically so it wont be stuck in a loop", Toast.LENGTH_SHORT).show();
                            break;
                        case LoadParamsCallback.LOAD_PARAM_SUCCESS:
                            Toast.makeText(MainActivity.this, "Start EMV", Toast.LENGTH_SHORT).show();
                            break;
                    }
                }
            },false);
    }
    
```
#### loadParams - function
| Parameter | Description | Example |
| ------ | ------ | ------ |
| Context context | Callback reference | MainActivity.this |
| boolean Force | Parameter for debugging <br> True - Wipes loading params cache memory (Will result burning information to the mPos device which make the load params slower)   | True / False  |
| LoadParamsCallback loadParamsCallback | Trigger the transaction from the BBCommunication class in the SDK  |  [Here](#startamount)  |


##### Load params has 2 callbacks, 
 
####  <br>1) Res Integer callback.
###### Integer callback that state whether the Loading Activity has successfully opened or not.
| Parameter | Description |  
| ------ | ------ | ------ |
| BBPosCallback.SUCCESS | Transaction opened successfully.<br><br> <strong>Can be ignored and handle only fail statuses.</strong>  |
| BBPosCallback.DEVICE_BUSY | Device is busy |
| BBPosCallback.BT_DISCONNECTED | Device is not connected  |

#### <br> 2) LoadParamsCallback
###### Custom Callback that indicates if the loading was success or not.
| Parameter | Description |  
| ------ | ------ | ------ |
| BBPosCallback.LOAD_PARAM_SUCCESS | Load Parameters loaded successfully.  |
| BBPosCallback.LOAD_PARAM_FAILED | Load Param could not load.<br> <br> Prompt dialog to try load params again is highly recommanded. <br><br> <strong>-NOTE: <br> Do Not automatically intiate load params, doing so will result a loop. </strong>  |

## FAQ
### Connecting to bluetooth shows a scan in the logs, but connection never made what to do?

  Run *SDK MANAGER* from the android studio,  *SDK Platforms*, Check *Show Package Details* 
  and install / update the following : <br> <br> 
  "*Android SDK Platform XX*"<br> 
  "*Sources for Android XX*" <br><br> 
  XX = targetSdkVersion

### In the load parameters screen I get Error(900)

  It means that the token that was provided is invalid or not provided at all.

### Load params error status
    
    Most of the status that there is except (900) are coming directly from SHVA.<br>
    Therefore:<br>
    Status: 999 Means no communication to SHVA
    
  
### Does the load parameters screen can be modified?

    Yes, the screen can be modified via params that we define in our server <br>  i.e. Logo, Background color, Text Colors.
    
### How my server will be notified with the transaction's result?

    There are two ways to be notified:
- <strong> Server-To-Server: </strong> By passing in the initiate transaction Hash the yaadOb.notify and yaadObj.norifyUrl you can be notified directly from our server to the specific url you choose for the response. <strong>[Highly recommanded]</strong> <br>
    
```
    hash.put("yaadObj.notify", "url"); 
    
    hash.put("yaadObj.notifyURL", "http://www.mysite.com/emv_trans.php?id=1234");
```
- <strong> App-To-Server </strong>
    At the end of the transaction the caller Activity will receive the Transaction's response and the send it to the sever. <strong>NOTE: in case of a crush / Exit application without saving the response locally the transacion will be lost.</stong>