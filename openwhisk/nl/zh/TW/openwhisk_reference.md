---

 

copyright:

  years: 2016

 

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock:.codeblock}
{:screen:.screen}
{:pre: .pre}

# {{site.data.keyword.openwhisk_short}} 系統詳細資料
{: #openwhisk_reference}
*前次更新：2016 年 4 月 14 日*
{: .last-updated}

下列各節提供有關 {{site.data.keyword.openwhisk}} 系統的其他詳細資料。
{: shortdesc}

## {{site.data.keyword.openwhisk_short}} 實體
{: #openwhisk_entities}

### 名稱空間及套件

{{site.data.keyword.openwhisk_short}} 動作、觸發程式及規則屬於名稱空間，而且可以選擇性地屬於套件。

套件可以包含動作及資訊來源。套件不能包含另一個套件，因此不容許巢狀套件。實體也不需要包含在套件中。

在 Bluemix 中，組織+空間配對對應至 {{site.data.keyword.openwhisk_short}} 名稱空間。例如，組織 `BobsOrg` 及空間 `dev` 將對應至 {{site.data.keyword.openwhisk_short}} 名稱空間 `/BobsOrg_dev`。

如果您已獲授權，請自行建立名稱空間。`/whisk.system` 名稱空間是保留供隨 {{site.data.keyword.openwhisk_short}} 系統一起配送的實體使用。


### 完整名稱

實體的完整名稱是 `/namespaceName[/packageName]/entityName`。請注意，`/` 是用來區隔名稱空間、套件及實體。名稱空間的前面也必須加上 `/`。

為方便起見，您可以停止使用者的 *預設名稱空間*。

例如，請考慮預設名稱空間為 `/myOrg` 的使用者。以下範例說明多個實體的完整名稱及其別名。

| 完整名稱 | 別名 | 名稱空間 | 套件 | 名稱 |
| --- | --- | --- | --- | --- |
| `/whisk.system/cloudant/read` |  | `/whisk.system` | `cloudant` | `read` |
| `/myOrg/video/transcode` | `video/transcode` | `/myOrg` | `video` | `transcode` |
| `/myOrg/filter` | `filter` | `/myOrg` |  | `filter` |

在其他位置中，使用 {{site.data.keyword.openwhisk_short}} CLI 時，將會使用此命名方法。

### 實體名稱

所有實體（包括動作、觸發程式、規則、套件及名稱空間）的名稱都是遵循下列格式的一串字元：

* 第一個字元必須是英數字元、數字或底線。
* 後續字元可以是英數字元、數字、空格或下列任何一項：`_`、`@`、`.`、`-`。
* 最後一個字元不能是空格。

更精確地來說，名稱必須符合下列正規表示式（使用 Java meta 字元語法表示）：`\A([\w]|[\w][\w@ .-]*[\w@.-]+)\z`。


## 動作語意
{: #openwhisk_semantics}

下列各節說明 {{site.data.keyword.openwhisk_short}} 動作的詳細資料。

### 無狀態

動作實作應該是無狀態，或*等冪*。雖然系統不會強制執行此內容，但是不保證在呼叫之間可以使用動作所維護的任何狀態。

此外，動作可能會有多個說明實例，而且每個說明實例都有自己的狀態。動作呼叫可能會分派至所有這些說明實例。

### 呼叫輸入及輸出

動作的輸入及輸出是鍵值組的定義檔。索引鍵是字串，而值是有效的 JSON 值。

### 動作的呼叫順序
{: #openwhisk_ordering}

動作不是依序進行呼叫。如果使用者從指令行或 REST API 呼叫動作兩次，則可能會先執行第二次呼叫，再執行第一次呼叫。如果動作有負面影響，則可能會依任意順序觀察到它們。

此外，不保證自動執行動作。可以同時執行兩個動作，而其負面影響可能會交錯。OpenWhisk 無法確保任何特定並行一致性模型是否有負面影響。所有並行性負面影響都是相依實作。

### 最多一次語意
{: #openwhisk_atmostonce}

系統支援最多呼叫一次動作。

收到呼叫要求時，系統會記錄要求，並分派啟動。

系統會傳回啟動 ID（如果是非封鎖呼叫），以確認已收到呼叫。請注意，即使沒有此回應（可能是因為網路連線中斷），還是可能會收到呼叫。

系統嘗試呼叫該動作一次，而導致下列四種結果的其中一種：
- *成功*：順利完成動作呼叫。
- *應用程式錯誤*：動作呼叫成功，但動作故意傳回錯誤值（例如，因為引數的前置條件不相符）。
- *動作開發人員錯誤*：已呼叫動作，但異常完成（例如，動作未捕捉到異常狀況，或有語法錯誤）。
- *Whisk 內部錯誤*：系統無法呼叫動作。
結果會記錄在啟動記錄的 `status` 欄位中（如下節所記載）。

每個成功收到的呼叫以及每個可能向使用者收費的呼叫，最後都會有一筆啟動記錄。


## 啟動記錄
{: #openwhisk_ref_activation}

每一個動作呼叫及觸發程式發動都會導致啟動記錄。

啟動記錄包含下列欄位：

- *activationId*：啟動 ID。
- *start* 及 *end*：記錄啟動開始及結束的時間戳記。值為 [UNIX 時間格式](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap04.html#tag_04_15)。
- *namespace* 及 `name`：實體的名稱空間及名稱。
- *logs*：字串陣列，內含動作在其啟動期間所產生的日誌。每一個陣列元素都對應至動作之 stdout 或 stderr 的行輸出，並且包括日誌輸出的時間及串流。結構如下：`TIMESTAMP STREAM: LOG_OUTPUT`。
- *response*：定義索引鍵 `success`、`status` 及 `result` 的字典：
  - *status*：啟動結果，可能是下列其中一個值："success"、"application error"、"action developer error"、"whisk internal error"。
  - *success*：假設並且只有在狀態為 `"success"` 時，才會為 `true`
- *result*：包含啟動結果的字典。如果啟動成功，這會包含動作所傳回的值。如果啟動失敗，`result` 一定會包含 `error` 索引鍵，通常還會有失敗的說明。


## JavaScript 動作
{: #openwhisk_ref_javascript}

### 函數原型

{{site.data.keyword.openwhisk_short}} JavaScript 動作是在 Node.js 運行環境（目前為 0.12.9 版）中執行。

以 JavaScript 撰寫的動作必須限制於單一檔案。此檔案可以包含多個函數，但依慣例，必須要有稱為 `main` 的函數，而且此函數是在呼叫動作時呼叫。例如，以下是具有多個函數的動作的範例。

```
function main() {return { payload: helper() }
}

function helper() {
    return new Date();
}
```
{: codeblock}

動作輸入參數被當作 `main` 函數的 JSON 物件參數來傳遞。成功啟動的結果也是 JSON 物件，但會根據動作是同步還是非同步（如下節所述），而以不同的方式傳回。


### 同步及非同步行為

JavaScript 函數常見於在回呼函數中繼續執行，即使是在傳回之後也是一樣。考慮到這種情況，JavaScript 動作的啟動可以是*同步* 或*非同步*。

如果 main 函數在下列其中一種狀況下結束，則 JavaScript 動作的啟動是**同步**：

- main 函數結束，而未執行 `return` 陳述式。
- main 函數結束，因為所執行的 `return` 陳述式傳回 `whisk.async()` *除外* 的任何值。

以下是兩個同步動作範例。

```
// a synchronous action
function main() {
  return {payload: 'Hello, World!'};
}
```
{: codeblock}

```
// an action in which each path results in a synchronous activation
function main(params) {
  if (params.payload == 0) {return;
  } else if (params.payload == 1) {
     return whisk.done();    // indicates normal completion
  } else if (params.payload == 2) {
    return whisk.error();   // indicates abnormal completion
  }
}
```
{: codeblock}

如果 main 函數因為呼叫 `return whisk.async();` 而結束，則 JavaScript 動作的啟動為**非同步**。在此情況下，除非動作執行下列其中一項，否則系統會假設動作仍在執行中：
- `return whisk.done();`
- `return whisk.error();`

以下是非同步執行的動作範例。

```
function main() {setTimeout(function() {
        return whisk.done({done: true});
    }, 100);
    return whisk.async();
}
```
{: codeblock}

某個動作在部分輸入上可能為同步，但在其他輸入上則為非同步。以下是範例。

```
function main(params) {
     if (params.payload) {
         setTimeout(function() {
            return whisk.done({done: true});
         }, 100);
         return whisk.async();  // asynchronous activation
      }  else {
         return whisk.done();   // synchronous activation
      }
  }
```
{: codeblock}

- 在此情況下，`main` 函數應該會傳回 `whisk.async()`。啟動結果可供使用時，應該呼叫 `whisk.done()` 函數，並將結果作為 JSON 物件來傳遞。這稱為*非同步* 啟動。

請注意，不論啟動是同步還是非同步，動作的呼叫都可以是封鎖或非封鎖。

### 其他 SDK 方法

`whisk.invoke()` 函數會呼叫另一個動作。它接受定義下列參數的字典作為引數：

- *name*：要呼叫之動作的完整名稱。
- *parameters*：JSON 物件，代表所呼叫動作的輸入。如果省略，預設值為空的物件。
- *apiKey*：用來呼叫動作的授權金鑰。
預設值為 `whisk.getAuthKey()`。
- *blocking*：應該以封鎖還是非封鎖模式呼叫動作。預設值為 `false`，這指出非封鎖呼叫。
- *next*：要在呼叫完成時執行的選用回呼函數。

`next` 的簽章是 `function(error, activation)`，其中：

- 如果呼叫成功，則 `error` 是 `false`，如果失敗，則為實際的值，這通常是說明錯誤的字串。
- 發生錯誤時，可能是未定義 `activation`（視失敗模式而定）。
- 若已定義，則 `activation` 是包含下列欄位的字典：
  - *activationId*：啟動 ID：
  - *result*：如果已使用封鎖模式來呼叫動作，則動作結果為 JSON 物件，否則為 `undefined`。

`whisk.trigger()` 函數會發動觸發程式。它接受包含下列參數的 JSON 物件作為引數：

- *name*：要呼叫的觸發程式的完整名稱。
- *parameters*：JSON 物件，代表觸發程式的輸入。如果省略，預設值為空的物件。
- *apiKey*：用來發動觸發程式的授權金鑰。預設值為 `whisk.getAuthKey()`。
- *next*：要在發動完成時執行的選用回呼。

`next` 的簽章是 `function(error, activation)`，其中：

- 如果發動成功，則 `error` 是 `false`，如果失敗，則為實際的值，這通常是說明錯誤的字串。
- 發生錯誤時，可能是未定義 `activation`（視失敗模式而定）。
- 若已定義，則 `activation` 是 `activationId` 欄位包含啟動 ID 的字典。

`whisk.getAuthKey()` 函數會傳回用來執行動作的授權金鑰。您通常不需要直接呼叫此函數，因為 `whisk.invoke()` 及 `whisk.trigger()` 函數會隱含地使用它。

### 運行環境
{: #openwhisk_ref_runtime_environment}

JavaScript 動作是在 Node.js 0.12.14 版環境中執行，該環境中含有可供動作使用的下列套件：

- apn
- async
- body-parser
- btoa
- cheerio
- cloudant
- commander
- consul
- cookie-parser
- cradle
- errorhandler
- express
- express-session
- gm
- jade
- log4js
- merge
- moment
- mustache
- nano
- node-uuid
- oauth2-server
- process
- request
- rimraf
- semver
- serve-favicon
- socket.io
- socket.io-client
- superagent
- swagger-tools
- tmp
- watson-developer-cloud
- when
- ws
- xml2js
- xmlhttprequest
- yauzl


## Docker 動作
{: #openwhisk_ref_docker}

Docker 動作是在 Docker 容器中執行使用者提供的二進位檔。二進位檔是在根據 Ubuntu 14.04 LTD 的 Docker 映像檔中執行，因此二進位檔必須與此發行套件相容。

動作輸入 "payload" 參數會被當作位置引數傳遞至二進位程式，而且會在 "result" 參數中傳回執行程式的標準輸出。

Docker 架構是建置 {{site.data.keyword.openwhisk_short}} 相容 Docker 映像檔的一種簡便方式。您可以使用 `wsk sdk install docker` CLI 指令來安裝架構。

應該將主要二進位程式複製到 `dockerSkeleton/client/action` 檔。任何伴隨的檔案或程式庫都可以位於 `dockerSkeleton/client` 目錄中。

透過修改 `dockerSkeleton/Dockerfile`，也可以包含任何編譯步驟或相依關係。例如，如果動作是 Python Script，您可以安裝 Python。


## REST API
{: #openwhisk_ref_restapi}

系統中的所有功能都可透過 REST API 來使用。其中有動作、觸發程式、規則、套件、啟動和名稱空間的集合和實體端點。

以下是集合端點：

- `https://{BASE URL}/api/v1/namespaces`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/actions`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/triggers`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/rules`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/packages`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/activations`

`{BASE URL}` 是 OpenWhisk API 主機名稱（亦即 openwhisk.ng.bluemix.net、172.17.0.1 等等）

針對 `{namespace}`，字元 `_` 可以用來指定使用者的 *預設名稱空間*（亦即，電子郵件位址）

您可以在集合端點上執行 GET 要求，以提取集合中的實體清單。

每一種實體類型都有實體端點：

- `https://{BASE URL}/api/v1/namespaces/{namespace}`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/actions/[{packageName}/]{actionName}`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/triggers/{triggerName}`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/rules/{ruleName}`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/packages/{packageName}`
- `https://{BASE URL}/api/v1/namespaces/{namespace}/activations/{activationName}`

名稱空間和啟動端點僅支援 GET 要求。動作、觸發程式、規則和套件端點可支援 GET、PUT 及 DELETE 要求。動作、觸發程式和規則的端點也可支援 POST 要求（用來呼叫動作和觸發程式，以及啟用或停用規則）。如需詳細資料，請參閱 [API 參考資料](https://new-console.{DomainName}/apidocs/98)。

所有 API 都是透過 HTTP 基本鑑別進行保護。基本鑑別認證位於 `~/.wskprops` 檔案的 `AUTH` 內容中，以冒號區隔。您也可以在 [CLI 配置步驟](../README.md#setup-cli)中擷取這些認證。

下列範例是使用 cURL 指令來取得 `whisk.system` 名稱空間中的所有套件清單：

```
curl -u USERNAME:PASSWORD https://openwhisk.ng.bluemix.net/api/v1/namespaces/whisk.system/packages
```
{: pre}
```
[
  {
    "name": "slack",
    "binding": false,
    "publish": true,
    "annotations": [
      {
        "key": "description",
        "value": "Package which contains actions to interact with the Slack messaging service"
      }
    ],
    "version": "0.0.9",
    "namespace": "whisk.system"
  },
  ...
]
```
{: screen}

OpenWhisk API 支援 Web 用戶端發出的「要求/回應」呼叫。OpenWhisk 會以「跨原點資源共用」標頭來回應 `OPTIONS` 要求。目前可允許所有原點（亦即，Access-Control-Allow-Origin 為 "`*`"），而 Access-Control-Allow-Header 會產生 Authorization 和 Content-Type。

**由於 OpenWhisk 目前針對每個帳戶僅支援一個金鑰，因此不建議在簡單實驗之外使用 CORS。您的金鑰需要內嵌在用戶端程式碼中，以公開顯示。請謹慎使用。**

## 系統限制
{: #openwhisk_syslimits}

{{site.data.keyword.openwhisk_short}} 有一些系統限制，包括動作所使用的記憶體數量，以及每小時允許的動作呼叫次數。下表列出預設限制。

| 限制 | 說明 | 可配置 | 單位 | 預設值 |
| ----- | ----------- | ------------ | -----| ------- |
| timeout | 不容許容器的執行時間超過 N 毫秒 | 每個動作 |  毫秒 | 60000 |
| memory | 不容許容器配置超過 N MB 的記憶體 | 每個動作 | MB | 256 |
| concurrent | 每個名稱空間不容許超過 N 個並行啟動 | 每個名稱空間 | 數字 | 100 |
| minuteRate | 使用者每分鐘不能呼叫超過這麼多動作 | 每位使用者 | 數字 | 120 |
| hourRate | 使用者每小時不能呼叫超過這麼多動作 | 每位使用者 | 數字 | 3600 |

### 每個動作逾時（毫秒）（預設值：60 秒）
* 逾時限制 N 落在 [100 毫秒..300000 毫秒] 的範圍內，並且是針對每個動作設定（毫秒）。
* 使用者可以在建立動作時變更此限制。
* 終止執行時間超過 N 毫秒的容器。

### 每個動作記憶體 (MB)（預設值：256MB）
* 記憶體限制 M 落在 [128MB..512MB] 的範圍內，並且是針對每個動作設定 (MB)。
* 使用者可以在建立動作時變更此限制。
* 配置給容器的記憶體不能超過限制。

### 每個動作構件 (MB)（固定：1MB）
* 動作的程式碼大小上限為 1MB。
* 建議 JavaScript 動作使用工具，將所有原始碼（包括相依關係）連結成單一組合檔。

### 每個啟動有效負載大小 (MB)（固定：1MB）
* POST 內容大小，加上為動作呼叫或觸發程式發動而附帶的任何參數，上限為 1MB。

### 每個名稱空間並行呼叫（預設值：100）
* 目前針對名稱空間所處理的啟動次數不能超過 100。
* Whisk 可以在 consul kvstore 中靜態配置預設限制。
* 使用者目前無法變更限制。

### 每分鐘/小時的呼叫次數（固定：120/3600）
* 速率限制 N 設定為 120/3600，並限制一分鐘/小時的時間範圍內的動作呼叫次數。
* 使用者無法在建立動作時變更此限制。
* 超過此限制的 CLI 呼叫會收到與 TOO_MANY_REQUESTS 對應的錯誤碼。

### 每個 Docker 動作開啟檔案 ulimit（固定：64:64）
* 開啟檔案的數量上限為 64（這同時套用於硬性和軟性限制）。
* docker run 指令使用引數 `--ulimit nofile=64:64`。
* 如需開啟檔案 ulimit 的相關資訊，請參閱 [docker run](https://docs.docker.com/engine/reference/commandline/run) 文件。

### 每個 Docker 動作程序數 ulimit（固定：512:512）
* 可供使用者使用的程序數上限為 512（這同時套用於硬性和軟性限制）。
* docker run 指令使用引數 `--ulimit nproc=512:512`。
* 如需程序數上限 ulimit 的相關資訊，請參閱 [docker run](https://docs.docker.com/engine/reference/commandline/run) 文件。
