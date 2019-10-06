# Game Client API

## 忘情辛巴 客戶端 SDK 說明文件

版本: 1.0.0

## SDK 使用需求

* 支援ES2017/ES7 以上的開發還境 (支援Promise與async/await)
* babel, babel-polyfill 轉譯工具
* Webpack/browsify 佈署工具

## SDK 檔案目錄結構

```text
├── sdk.${SdkVersion}.js - GameClient SDK (未經babel-polyfill處理的版本)
├── sdk.polyfill.${SdkVersion}.js - GameClient SDK (經babel-polyfill處理過的版本)
├── doc - 說明文件
│   ├── sdk.${SdkVersion}.html - SDK說明文件，HTML版本
│   ├── sdk.${SdkVersion}.md - - SDK說明文件，Markdown格式原始檔
│   └── sdk.${SdkVersion}.pdf - SDK說明文件，PDF版本
```

**說明**

* `sdk.${SdkVersion}.js` 為未經babel-polyfill處理的版本，內含ES6 module、async/await...等程式碼，適合用於專案內導入(import)之後，最後使用babel-polyfill轉譯

* `sdk.polyfill.${SdkVersion}.js` 為babel-polyfill轉譯處理過的版本，適合直接於瀏覽器內直接引用

## 遊戲前端網址動態處理 parameter

* token => 帳號登入Token
* language => 語系 (預設 **zh-cn**)
* domain => 連結的 domain (預設 **ws://game-test.clctech.co:5800/game/**)

## SDK 測試環境

* 測試Server Websocket URL: **ws://game-test.clctech.co:5800/game/g2**
* 簡易遊戲測試頁面URL: **[http://game-test.clctech.co:5001/g2](http://game-test.clctech.co:5001/g2)**

## SDK 基本使用說明

此SDK提供以下功能：

* Request/Reply(REQ/REP) pattern (呼叫就可以得到 Server 的回應)
  - 登入 (login)
  - 開始遊戲 (startGame)
  - 取得遊戲房間列表 (roomList)
  - 加入房間 (join)
  - 離開房間 (leave)
  - 取得歷史資料 (history)
  - 下注，前端必須判斷當前狀態是否可以執行下注，Server端也會判斷 (bet)

* Publisher/Subscriber(PUB/SUB) pattern (註冊 Event，執行 Callback , 等待 Server 傳資料過來)
  - 監聽 Server 回傳的下注訊息，平均兩秒 Server 會送資料過來更新 (betInfo)
  - 監聽 Server 當前處於狀態 (roomStatus)
    - 0: 新局開始
    - 1: 關閉押注
    - 2: 開獎
  - 監聽 Server 回傳的開獎訊息，中獎金額 (results)
  - 監聽 Server 回傳可能要進行強制斷線，停機維護等等訊息 (error)

使用時需依
**login** =>
**startGame** =>
**roomList** =>
**join** =>
**history** =>
**setOnBetInfoEvent** =>
**setOnRoomStatusEvent** =>
**setOnResultsEvent** =>
**bet**  =>
**leave**  =>
**close**  =>
的順序操作。

SDK內每個方法皆為非同步方法(async. method)，回傳標準`<Promise>`物件或拋出例外錯誤 (throw exception error)。

使用時可利用 Promise 或 async/await 方式操作，最後利用 babel 轉譯成瀏覽器相容的JS Code。

此外，每一個方法都需檢查拋出的例外錯誤，並依錯誤代碼進行相對應的處理。若是使用 Promise 方式操作，需要使用 `catch` 方法處理例外錯誤；若是使用ES7 async/await 方式操作，則需使用標準 `try/catch` 方法接取例外錯誤。

監聽的時候得到的資料都是 Json 格式的物件，請依造遊戲整體邏輯配合觸發 Callback 時，應該採取的行為。

## 資料結構

### Bet

發送給 `GameClient.prototype.bet(betAmounts, betType, betKind)`

**Request ( betRequest 參數)**

| Attribute  | Type   | Description                               |
| ---------- | ------ | ----------------------------------------- |
| betAmounts | number | 押注金額。                                  |
| betType    | string | 押注類型(e.g. 獅子金燈，早上，晚上，舞會。) |
| betKind    | number | 押注還是取消(0: 取消， 1: 押注)。            |

```javascript
const BetRequestJsonSchema = {
    type: 'object',
    properties: {
      betAmounts: {
        description: '押注金額',
        type: 'number',
      },
      betType: {
        description: '押注類型(e.g. 獅子金燈，早上，晚上，舞會。)',
        type: 'string',
      },
      betKind: {
        description: '押注還是取消(0: 取消， 1: 押注)',
        type: 'number',
      }
    }
  }
```

**Reply**

| Attribute     | Type   | Description            |
| ------------- | ------ | ---------------------- |
| status        | number | 狀態(0: 失敗， 1: 成功)。 |
| messageNumber | number | 錯誤狀態碼。             |
| balance       | number | 玩家餘額。             |


```javascript
const BetReplyJsonSchema = {
    type: 'object',
    properties: {
      status: {
        description: '狀態(0: 失敗， 1: 成功)',
        type: 'number',
      },
      messageNumber: {
        description: '錯誤狀態碼',
        type: 'number',
      },
      balance: {
            description: '玩家餘額',
            type: 'uint64le',
      },
      serverTime: {
            description: '當前 Server 的時間',
            type: 'integer',
      }
    }
  }
```

### Login

玩家資訊，經由 `GameClient.prototype.login()` 回傳

```javascript
const PlayerJsonSchema = {
    type: 'object',
    properties: {
        name: {
            description: '顯示於遊戲畫面的玩家名稱',
            type: 'string',
        },
        nickname: {
            description: '顯示於遊戲畫面的玩家暱稱',
            type: 'string',
        },
        language: {
            description: '玩家使用的語系',
            type: 'string',
            enum: ['zh-CN', 'zh-HK', 'zh-TW', 'en-US'],
        },
        balance: {
            description: '目前餘額',
            type: 'integer',
        },
        timestamp: {
          description: '當前 Server 的時間',
          type: 'integer',
        },
    },
};
```

### RoomList

發送給 `GameClient.prototype.roomList()`

**Reply**

| Attribute           | Type   | Description                |
| ------------------- | ------ | -------------------------- |
| roomList            | array  | 不固定長度的物件陣列。     |

```javascript
const RoomListReplyJsonSchema = {
    type: 'object',
    properties: {
      roomList: {
            description: '不固定長度的文字陣列',
            type: 'array',
            items: {
                type: 'object',
                properties: {
                    roomId: {
                        description: '房間編號',
                        type: 'string',
                    },
                    roomPlayers: {
                        description: '房間內有多少玩家',
                        type: 'number',
                    },
                },
            },
        },
        ,
      serverTime: {
            description: '當前 Server 的時間',
            type: 'integer',
      }
    }
  }
```

### Join

發送給 `GameClient.prototype.join(roomID)`

**Request ( joinRequest 參數)**

| Attribute    | Type   | Description                    |
| ------------ | ------ | ------------------------------ |
| roomID       | string | 選擇的房號(如果同時有多間房間)。 |

```javascript
const JoinRequestJsonSchema = {
    roomID: {
        description: '房號',
        type: 'string',
      },
  }
```

**Reply**

| Attribute           | Type   | Description                |
| ------------------- | ------ | -------------------------- |
| status              | number | 狀態(0: 失敗， 1: 成功)。     |
| messageNumber       | number | 錯誤狀態碼。                 |
| L0                  | number | 獅子(無燈)。  |
| L1                  | number | 獅子(金燈)。  |
| L2                  | number | 獅子(紅燈)。  |
| L3                  | number | 獅子(綠燈)。  |
| P0                  | number | 熊貓(無燈)。  |
| P1                  | number | 熊貓(金燈)。  |
| P2                  | number | 熊貓(紅燈)。  |
| P3                  | number | 熊貓(綠燈)。  |
| M0                  | number | 猴子(無燈)。  |
| M1                  | number | 猴子(金燈)。  |
| M2                  | number | 猴子(紅燈)。  |
| M3                  | number | 猴子(綠燈)。  |
| R0                  | number | 兔子(無燈)。  |
| R1                  | number | 兔子(金燈)。  |
| R2                  | number | 兔子(紅燈)。  |
| R3                  | number | 兔子(綠燈)。  |
| Day                 | number | 日。          |
| Night               | number | 夜。          |
| Prom                | number | 舞會。        |
| roomStatus       | number | 當前房間的狀態。                 |
| roomRoundCreateTime | string | 此房此局開始時間(倒數使用)。 |
| countdownNumber | number | 此房此局倒數時間(倒數使用)。 |
| animalRoulettes | string | 動物轉盤列表(e.g. "MRRLRRPRMRPM")。 |
| colourRoulettes | string | 顏色轉盤列表(e.g. "331332223123")。 |
| maxBetAmount    | string | 單動物 最多下注金額。 |
| scenesWinType      | string | 前一局場景開出的獎項內容(e.g. "Prom") |
| animalWinType     | string | 前一局動物開出的獎項內容(e.g. "R3") |
| resultWin         | number | 前一局中獎的金額 |
| balance           | number | 當前玩家餘額 |

```javascript
const JoinReplyJsonSchema = {
    type: 'object',
    properties: {
      status: {
        description: '狀態(0: 失敗， 1: 成功)',
        type: 'number',
      },
      messageNumber: {
        description: '錯誤狀態碼',
        type: 'number',
      },
      L0: {
            description: '獅子(無燈)',
            type: 'uint32le',
      },
      L1: {
          description: '獅子(金燈)',
          type: 'uint32le',
      },
      L2: {
          description: '獅子(紅燈)',
          type: 'uint32le',
      },
      L3: {
          description: '獅子(綠燈)',
          type: 'uint32le',
      },
      P0: {
          description: '熊貓(無燈)',
          type: 'uint32le',
      },
      P1: {
          description: '熊貓(金燈)',
          type: 'uint32le',
      },
      P2: {
          description: '熊貓(紅燈)',
          type: 'uint32le',
      },
      P3: {
          description: '熊貓(綠燈)',
          type: 'uint32le',
      },
      M0: {
          description: '猴子(無燈)',
          type: 'uint32le',
      },
      M1: {
          description: '猴子(金燈)',
          type: 'uint32le',
      },
      M2: {
          description: '猴子(紅燈)',
          type: 'uint32le',
      },
      M3: {
          description: '猴子(綠燈)',
          type: 'uint32le',
      },
      R0: {
          description: '兔子(無燈)',
          type: 'uint32le',
      },
      R1: {
          description: '兔子(金燈)',
          type: 'uint32le',
      },
      R2: {
          description: '兔子(紅燈)',
          type: 'uint32le',
      },
      R3: {
          description: '兔子(綠燈)',
          type: 'uint32le',
      },
      Day: {
          description: '日',
          type: 'uint32le',
      },
      Night: {
          description: '夜',
          type: 'uint32le',
      },
      Prom: {
          description: '舞會',
          type: 'uint32le',
      },
      roomStatus: {
        description: '當前房間的狀態',
        type: 'number',
      },
      roomRoundCreateTime: {
        description: '此房此局開始時間(倒數使用)',
        type: 'string',
      },
      countdownNumber: {
        description: '此房此局倒數時間(倒數使用)',
        type: 'number',
      },
      animalRoulettes: {
          description: '動物轉盤列表(e.g. "[R, P, M, R, L, A, L, R, L, M, P, A, P, M]")',
            type: 'array',
            items: {
                type: 'string',
                minItems: 12,
                maxItems: 12,
            },
      },
      colourRoulettes: {
        description: '顏色轉盤列表(e.g. "[1, 3, 2, 1, 2, 3]")',
          type: 'array',
          items: {
              type: 'string',
              minItems: 12,
              maxItems: 12,
          },
      },
      maxBetAmount: {
        description: '單動物 最多下注金額',
        type: 'number',
      },
      scenesWinType: {
            description: '前一局場景開出的獎項內容(e.g. "Prom")',
            type: 'string',
      },
      animalWinType: {
          description: '前一局動物開出的獎項內容(e.g. "R3")',
          type: 'string',
      },
      resultWin: {
          description: '前一局中獎的金額',
          type: 'number',
      },
      balance: {
          description: '當前玩家餘額',
          type: 'number',
      },
      serverTime: {
            description: '當前 Server 的時間',
            type: 'integer',
      }
    }
	}
```

**場景跟動物代碼對應表**

| Attribute | Description |
| --------- | ----------- |
| L        | 獅子。|
| S       | 獅子博兔。|
| P        | 熊貓。|
| M        | 猴子。|
| R        | 兔子。|
| 1        | 金燈。|
| 2        | 紅燈。|
| 3        | 綠燈。|
| 4        | 彩燈。|

### Leave

玩家資訊，經由 `GameClient.prototype.leave()` 回傳
離開房間 Websocket 還是在持續連線中跟 close 是不一樣的行為。
**Reply**

| Attribute     | Type   | Description            |
| ------------- | ------ | ---------------------- |
| status        | number | 狀態(0: 失敗， 1: 成功)。 |
| messageNumber | number | 錯誤狀態碼。             |

```javascript
const JoinRequestJsonSchema = {
    type: 'object',
    properties: {
      status: {
        description: '狀態(0: 失敗， 1: 成功)',
        type: 'number',
      },
      messageNumber: {
        description: '錯誤狀態碼',
        type: 'number',
      },
      serverTime: {
            description: '當前 Server 的時間',
            type: 'integer',
      }
    }
  };
```

### History

取得這房的歷史遊戲紀錄，經由 `GameClient.prototype.history(roomID)` 回傳。

**Request**

| Attribute | Type   | Description      |
| --------- | ------ | ---------------- |
| roomID      | string | 要查詢的遊戲房間的歷史紀錄。         |

**Reply**

| Attribute           | Type   | Description                |
| ------------------- | ------ | -------------------------- |
| animalTypes         | array  | 固定36長度的文字陣列。     |
| 陣列內的文字          | string | 代表動物顏色組合加上-再加上場景。|

```javascript
const HistoryReplyJsonSchema = {
    type: 'object',
    properties: {
      animalTypes: {
            description: '固定36長度的文字陣列',
            type: 'array',
            items: {
                type: 'string',
                minItems: 36,
                maxItems: 36,
            },
        },
        serverTime: {
              description: '當前 Server 的時間',
              type: 'integer',
        }
    }
  }
```

### setOnBetInfoEvent

註冊 setOnBetInfoEvent 會得到平均每兩秒 Server 會送 betInfo 物件回來。

| openCode | eventType |
| -------- | --------- |
| 0x40     | 0x1       |

```javascript
GameClient.prototype.setOnBetInfoEvent((betInfo) => {
/* have something to do */
});
```

| Attribute | Type   | Description |
| --------- | ------ | ----------- |
| L0        | number | 獅子(無燈)。  |
| L1        | number | 獅子(金燈)。  |
| L2        | number | 獅子(紅燈)。  |
| L3        | number | 獅子(綠燈)。  |
| P0        | number | 熊貓(無燈)。  |
| P1        | number | 熊貓(金燈)。  |
| P2        | number | 熊貓(紅燈)。  |
| P3        | number | 熊貓(綠燈)。  |
| M0        | number | 猴子(無燈)。  |
| M1        | number | 猴子(金燈)。  |
| M2        | number | 猴子(紅燈)。  |
| M3        | number | 猴子(綠燈)。  |
| R0        | number | 兔子(無燈)。  |
| R1        | number | 兔子(金燈)。  |
| R2        | number | 兔子(紅燈)。  |
| R3        | number | 兔子(綠燈)。  |
| Day       | number | 日。          |
| Night     | number | 夜。          |
| Prom      | number | 舞會。        |
| Player1   | object | 第一個要顯示的玩家。          |
| Player2   | object | 第二個要顯示的玩家。          |
| Player3   | object | 第三個要顯示的玩家。        |
| Player4   | object | 第四個要顯示的玩家。        |

```javascript
const BetInfoResultJsonSchema = {
    type: 'object',
    properties: {
      Player1: {
          type: 'object',
          properties: {
            animalType: {
                description: '最高押注的動物或場景',
                type: 'string',
            },
            name: {
                description: '名字',
                type: 'string',
            },
            animalAmount: {
                description: '動物或場景當前押注金額',
                type: 'uint32le',
            },
        },
        Player2: {
          type: 'object',
          properties: {
            animalType: {
                description: '最高押注的動物或場景',
                type: 'string',
            },
            name: {
                description: '名字',
                type: 'string',
            },
            animalAmount: {
                description: '動物或場景當前押注金額',
                type: 'uint32le',
            },
        },
        Player3: {
          type: 'object',
          properties: {
            animalType: {
                description: '最高押注的動物或場景',
                type: 'string',
            },
            name: {
                description: '名字',
                type: 'string',
            },
            animalAmount: {
                description: '動物或場景當前押注金額',
                type: 'uint32le',
            },
        },
        Player4: {
          type: 'object',
          properties: {
            animalType: {
                description: '最高押注的動物或場景',
                type: 'string',
            },
            name: {
                description: '名字',
                type: 'string',
            },
            animalAmount: {
                description: '動物或場景當前押注金額',
                type: 'uint32le',
            },
        },
        L0: {
          description: '獅子(無燈)',
          type: 'number',
        },
        L1: {
          description: '獅子(金燈)',
          type: 'number',
        },
        L2: {
          description: '獅子(紅燈)',
          type: 'number',
        },
        L3: {
          description: '獅子(綠燈)',
          type: 'number',
        },
        P0: {
          description: '熊貓(無燈)',
          type: 'number',
        },
        P1: {
          description: '熊貓(金燈)',
          type: 'number',
        },
        P2: {
          description: '熊貓(紅燈)',
          type: 'number',
        },
        P3: {
          description: '熊貓(綠燈)',
          type: 'number',
        },
        M0: {
          description: '猴子(無燈)',
          type: 'number',
        },
        M1: {
          description: '猴子(金燈)',
          type: 'number',
        },
        M2: {
          description: '猴子(紅燈)',
          type: 'number',
        },
        M3: {
          description: '猴子(綠燈)',
          type: 'number',
        },
        R0: {
          description: '兔子(無燈)',
          type: 'number',
        },
        R1: {
          description: '兔子(金燈)',
          type: 'number',
        },
        R2: {
          description: '兔子(紅燈)',
          type: 'number',
        },
        R3: {
          description: '兔子(綠燈)',
          type: 'number',
        },
        Day: {
          description: '日',
          type: 'number',
        },
        Night: {
          description: '夜',
          type: 'number',
        },
        Prom: {
          description: '舞會',
          type: 'number',
        },
        serverTime: {
              description: '當前 Server 的時間',
              type: 'integer',
        }
     }
};
```

### setOnRoomStatusEvent

註冊 setOnRoomStatusEvent，Server 會送 roomStatus 物件回來更新當前狀態。

| openCode | eventType |
| -------- | --------- |
| 0x40     | 0x2       |

```javascript
GameClient.prototype.setOnRoomStatusEvent((roomStatusInfo) => {
/* have something to do */
});

```

| Attribute           | Type   | Description                                                  |
| ------------------- | ------ | ------------------------------------------------------------ |
| status              | number | 當前處於狀態(0: 新局開始， 1: 關閉押注， 2: 開獎)。 |
| scenesWinType       | string | 場景開出的獎項內容(e.g. "Prom")，狀態2才會是有效值 |
| animalWinType       | string | 動物開出的獎項內容(e.g. "R3")，狀態2才會是有效值 |
| animalRoulettes     | array  | 動物轉盤列表(e.g. "[R, P, M, R, L, A, L, R, L, M, P, A, P, M]")，狀態0,2才會是有效值 |
| animalPosition      | number | 動物轉盤最後指針位子(e.g. 3)，狀態2才會是有效值 |
| colourRoulettes     | array  | 顏色轉盤列表(e.g. "[1, 3, 2, 1, 2, 3]")，狀態0,2才會是有效值 |
| colourPosition      | number | 顏色轉盤最後指針位子(e.g. 3)，狀態2才會是有效值 |
| roomRoundCreateTime | number | 此房此局開始時間(倒數使用)，狀態0才會是有效值 |
| countdownNumber     | number | 此房此局開始時間(倒數使用) |
| maxBetAmount        | number | 單動物 最多下注金額 |

```javascript
const RoomStatusResultJsonSchema = {
    type: 'object',
    properties: {
        status: {
          description: '當前處於狀態(0: 新局開始， 1: 關閉押注， 2: 開獎)',
          type: 'number',
        },
        scenesWinType: {
          description: '場景開出的獎項內容(e.g. "Prom")',
          type: 'string',
        },
        animalWinType: {
          description: '動物開出的獎項內容(e.g. "R3")',
          type: 'string',
        },
        animalRoulettes: {
          description: '動物轉盤列表(e.g. "[R, P, M, R, L, A, L, R, L, M, P, A, P, M]")',
            type: 'array',
            items: {
                type: 'string',
                minItems: 12,
                maxItems: 12,
            },
        },
        animalPosition: {
          description: '動物轉盤最後指針位子(e.g. 3)',
          type: 'number',
        },
        colourRoulettes: {
          description: '顏色轉盤列表(e.g. "[1, 3, 2, 1, 2, 3]")',
            type: 'array',
            items: {
                type: 'string',
                minItems: 12,
                maxItems: 12,
            },
        },
        colourPosition: {
          description: '顏色轉盤最後指針位子(e.g. 3)',
          type: 'number',
        },
        roomRoundCreateTime: {
          description: '此房此局開始時間(倒數使用)',
          type: 'string',
        },
        countdownNumber: {
          description: '此房此局倒數時間(倒數使用)',
          type: 'number',
        },
        maxBetAmount: {
            description: '單動物 最多下注金額',
            type: 'uint32le',
        },
        serverTime: {
              description: '當前 Server 的時間',
              type: 'integer',
        }
     }
};
```

**場景跟動物獎項內容對應表**

| Attribute | Description |
| --------- | ----------- |
| L1        | 獅子(金燈)。|
| L2        | 獅子(紅燈)。|
| L3        | 獅子(綠燈)。|
| L4        | 獅子(彩燈)。|
| S1       | 獅子博兔(金燈)。|
| S2       | 獅子博兔(紅燈)。|
| S3       | 獅子博兔(綠燈)。|
| S4       | 獅子博兔(彩燈)。|
| P1        | 熊貓(金燈)。|
| P2        | 熊貓(紅燈)。|
| P3        | 熊貓(綠燈)。|
| P4        | 熊貓(彩燈)。|
| M1        | 猴子(金燈)。|
| M2        | 猴子(紅燈)。|
| M3        | 猴子(綠燈)。|
| M4        | 猴子(彩燈)。|
| R1        | 兔子(金燈)。|
| R2        | 兔子(紅燈)。|
| R3        | 兔子(綠燈)。|
| R4        | 兔子(彩燈)。|
| A1        | 全動物(金燈)。|
| A2        | 全動物(紅燈)。|
| A3        | 全動物(綠燈)。|
| A4        | 全動物(彩燈)。|
| Day       | 日。        |
| Night     | 夜。        |
| Prom      | 舞會。      |
| L        | 獅子。|
| S        | 獅子博兔。|
| P        | 熊貓。|
| M        | 猴子。|
| R        | 兔子。|
| 1        | 金燈。|
| 2       | 紅燈。|
| 3        | 綠燈。|
| 4        | 彩燈。|

### setOnPlayersInfoEvent

註冊 setOnPlayersInfoEvent，Server 會送 playersInfo 物件回來更新當前狀態。

| openCode | eventType |
| -------- | --------- |
| 0x40     | 0x3       |

```javascript
GameClient.prototype.setOnPlayersInfoEvent((playersInfo) => {
/* have something to do */
});
```

| Attribute      | Type   | Description                                       |
| -------------- | ------ | ------------------------------------------------- |
| type           | number | 離開或是進入(0: 離開， 1: 進入)。  |
| playerNikeName | string | 玩家名稱。       |

```javascript
const PlayersInfoResultJsonSchema = {
    type: 'object',
    properties: {
        type: {
          description: '離開或是進入(0: 離開， 1: 進入)',
          type: 'number',
        },
        playerNikeName: {
          description: '玩家名稱',
          type: 'string',
        },
        serverTime: {
              description: '當前 Server 的時間',
              type: 'integer',
        }
   	}
};
```

### setOnResultsEvent

註冊 setOnResultsEvent，Server 會送 resultsInfo 物件回來通知玩家贏了多少錢。

| openCode | eventType |
| -------- | --------- |
| 0x40     | 0x4       |

```javascript
GameClient.prototype.setOnResultsEvent((resultsInfo) => {
/* have something to do */
});
```

| Attribute | Type   | Description |
| --------- | ------ | ----------- |
| resultWin | number | 贏得的金額。  |
| balance   | number | 玩家餘額。  |

```javascript
const ResultsInfoResultJsonSchema = {
    type: 'object',
    properties: {
        resultWin: {
          description: '贏得的金額',
          type: 'number',
        },
        balance: {
          description: '玩家餘額',
          type: 'number',
        },
        serverTime: {
              description: '當前 Server 的時間',
              type: 'integer',
        }
     }
};
```

### setOnErrorEvent

註冊 setOnErrorEvent  Server 如果有要進行維修或是要強制斷線某位玩家，會送 errorInfo 物件來通知。

| openCode | eventType |
| -------- | --------- |
| 0x40     | 0xFF      |

```javascript
GameClient.prototype.setOnErrorEvent((errorInfo) => {
/* have something to do */
});
```

| Attribute | Type   | Description      |
| --------- | ------ | ---------------- |
| type      | number | 錯誤類型。         |
| message   | string | 額外錯誤文字訊息。 |

```javascript
const ResultsInfoResultJsonSchema = {
    type: 'object',
    properties: {
        type: {
          description: '錯誤類型',
          type: 'number',
        },
        message: {
          description: '額外錯誤文字訊息',
          type: 'string',
        },
        serverTime: {
              description: '當前 Server 的時間',
              type: 'integer',
        }
     }
};
```

## 錯誤代碼

| 代碼 | 類型  | 代號                          | 說明                | 訊息                               |
| ---- | ---- | ---------------------------- | -----------------  | ---------------------------------- |
| 1001 | 1    | AUTH_FAIL                    | 認證失敗            | 您的帳號錯誤，請聯繫客服              |
| 1002 | 1    | SERVER_OFFLINE               | 伺服器離線          | 遊戲維護中，請稍候再連線              |
| 1003 | 1    | USER_BANNED                  | 玩家強制驅離         | 您的帳號錯誤，請聯繫客服             |
| 1004 | 1    | USER_LOGOUT_BY_EC            | 玩家被外部登出       | 您的帳號已被登出，請重新登入           |
|      |      |                              |                    |                                  |
| 2001 | 2    | SERVER_INTERRUPT             | 斷線,重新連線        | 遊戲連線中斷，將為您重新連線       |
| 2002 | 2    | CONNECTION_TIMED_OUT         | 連線逾時            | 遊戲連線中斷，將為您重新連線       |
| 2003 | 2    | INVALID_OPERATION            | 錯誤操作            | 遊戲連線中斷，將為您重新連線       |
|      |      |                              |                    |                                    |
| 3001 | 3    | USER_SPIN_FROZEN             | 不允許BET          | 您的帳號暫時不允許下注，請聯繫客服 |
| 3002 | 3    | USER_LOCKED                  | 玩家帳戶被鎖定       | 您的帳號暫時不允許下注，請聯繫客服 |
| 3003 | 3    | INSUFFICIENT_BALANCE         | 餘額不足            | 您的帳號餘額不足                   |
| 3004 | 3    | INSUFFICIENT_FREE_GAME_TIMES | 免費遊戲數量不足     | 免費遊戲剩餘數量不足                 |

## API 說明

### GameClient `<Class>`

遊戲客戶端類別，提供所有遊戲需要的操作方法。

#### GameClient.errCode `<Object>`

遊戲錯誤代碼列表，key為錯誤代號，value為錯誤代碼。

```js
    GameClient.errCode.AUTH_FAIL === 1001;
    GameClient.errCode.INSUFFICIENT_BALANCE === 3001;
```

#### GameClient(url)

* `url <string>` Game server websocket URL

GameClient建構函式，生成新的game client物件

#### GameClient.prototype.login([token])

* `token <string>` 登入認證token
* Returns: `<Promise>` 回傳Promise物件，內含`<Player>` JSON物件

以`token`登入game server，登入成功之後回傳`<Player>`資訊。登入失敗會拋出(throw) `<ServerError>` 例外錯誤，可利用`ServerError.code`查詢 `錯誤代碼`。

> `token`為選擇性 (Optional) 參數，若未傳入 `token`參數，會取得當前Browser URL中的 `token` query string 做為登入token使用。
>
> 例如: `http://hostname/game.html?token=232`
>
> 此方法會取得 `token` 的值為: 232

#### GameClient.prototype.leave()

* Returns: `<Promise>` 回傳 Promise 物件。

離開遊戲房間。
發生錯誤時拋出 (throw) `<ServerError>` 例外錯誤，可利用`ServerError.code`查詢 `錯誤代碼`。

#### GameClient.prototype.join(roomId)

* Returns: `<Promise>` 回傳 Promise 物件接收單一參數 roomId。

加入遊戲房間。
發生錯誤時拋出 (throw) `<ServerError>` 例外錯誤，可利用`ServerError.code`查詢 `錯誤代碼`。

#### GameClient.prototype.close()

* Returns: `<Promise>.<Event>` 回傳Promise Event物件
關閉遊戲連線

#### GameClient.prototype.roomList()

* Returns: `<Promise>` 回傳 Promise 物件。

查詢當前遊戲有哪些房間。
發生錯誤時拋出 (throw) `<ServerError>` 例外錯誤，可利用`ServerError.code`查詢 `錯誤代碼`。

#### GameClient.prototype.startGame()

* Returns: `<Promise>` 回傳 Promise 物件。

開始此次遊戲回合，此方法必需在呼叫`login`方法之後呼叫，在其餘操作之前呼叫。
發生錯誤時拋出 (throw) `<ServerError>` 例外錯誤，可利用`ServerError.code`查詢 `錯誤代碼`。

#### GameClient.prototype.bet(betRequest)

* Returns: `<Promise>` 回傳 Promise 物件，接收單一參數 betRequest。

押注遊戲的方式。
發生錯誤時拋出 (throw) `<ServerError>` 例外錯誤，可利用`ServerError.code`查詢 `錯誤代碼`。

#### GameClient.prototype.setOnBetInfoEvent(callback)

* `callback <Function>` Callback，接收單一參數 betInfo`

註冊 setOnBetInfoEvent 會得到平均每兩秒 Server 會送 betInfo 物件回來。

註冊處理函式，平均每兩秒觸發一次。

**程式範例**

```js
client.setOnBetInfoEvent((betInfo) => {
    // 更新遊戲桌面押注資訊, 人頭資訊
});
```

#### GameClient.prototype.setOnRoomStatusEvent(callback)

* `callback <Function>` Callback，接收單一參數 roomStatusInfo`。

註冊 setOnRoomStatusEvent，Server 會送 roomStatus 物件回來更新當前狀態。

註冊處理函式，當遊戲狀態改變都會觸發。

**程式範例**

```js
client.setOnRoomStatusEvent((roomStatusInfo) => {
    // 更新遊戲狀態，如果是停止下注狀態就不能在下注了
});
```

#### GameClient.prototype.setOnPlayersInfoEvent(callback)

* `callback <Function>` Callback，接收單一參數 playersInfo`。

註冊 setOnPlayersInfoEvent，Server 會送 playersInfo 物件回來更新當前狀態。

註冊處理函式，如果有玩家進來這個房間或是出去這個房間都會被觸發。

**程式範例**

```js
client.setOnPlayersInfoEvent((playersInfo) => {
    // 有玩家進來這個遊戲房間了
});
```

#### GameClient.prototype.setOnResultsEvent(callback)

* `callback <Function>` Callback，接收單一參數 resultsInfo`。

註冊 setOnResultsEvent，Server 會送 resultsInfo 物件回來通知玩家贏了多少錢。

註冊處理函式，當玩家贏錢的時候就會被觸發。

**程式範例**

```js
client.setOnResultsEvent((resultsInfo) => {
    // 玩家贏錢了 Server 告知目前金額。
});
```

#### GameClient.prototype.onClose(callback)

* `callback <Function>` Callback，接收單一參數`error`

註冊處理函式，當發生連線關閉事件時觸發。

處理函式會接收單一參數`error`，`error`內含`code`與`reason`兩個參數

**程式範例**

```js
client.onClose((err) => {
    logger.info(`Client connection closed,  code: ${err.code} reason: ${err.reason}`);
});
```

#### GameClient.prototype.setOnErrorEvent(callback)

* `callback <Function>` Callback，接收單一參數`error`

註冊處理函式，當發生錯誤事件時觸發。

處理函式會接收單一參數`error`，`error`內含`code`與`reason`兩個屬性

**程式範例**

```js
client.setOnErrorEvent((err) => {
    logger.info(`Client got error event, code: ${err.code} reason: ${err.reason}`);
});
```

### GameClient.ServerError `<Class>`

伺服器錯誤類別，繼承標準 `Error` 錯誤類別，增加 `code` 一個額外屬性。

#### GameClient.ServerError.prototype.code `<number>`

錯誤代碼，數值定義詳見 `<錯誤代碼>` 章節。

#### GameClient.ServerError.prototype.gameId `<number>`

遊戲ID

## SDK範例

### 基本登入下注流程範例

```js
    import GameClient from 'path/to/sdk.js';

    const token = '232';
    const client = new GameClient('ws://game-test.clctech.co:5800/game/g2');
    let roomId;

    client.login(token).then((player) => {
        // 取得玩家資訊
        // 進行進一步遊戲初始化動作
        return client.startGame();
    })
    .then(() => {
        // 先取得房間編號
        return client.roomList();
    })
    .then((roomList) => {
        // 進入房間，記錄進入哪個房間
        roomId = roomList[0].roomId;
        return client.join(roomList[0].roomId);
    })
    .then((reelInfo) => {
        // 取得房間資訊，歷史紀錄也是呼叫這隻來取得
        return client.history(roomId);
    })
    .then((historyInfo) => {
        // 開始處理會被觸發的事件，註冊一定要寫在 join 執行完畢，取得資訊之後
        client.setOnBetInfoEvent((betInfo) => {
            // 更新遊戲桌面押注資訊，人頭資訊
        });
        client.setOnRoomStatusEvent((roomStatusInfo) => {
            // 更新遊戲狀態，如果是停止下注狀態就不能在下注了
        });
        client.setOnPlayersInfoEvent((playersInfo) => {
            // 有玩家進來這個遊戲房間了
        });
        client.setOnResultsEvent((resultsInfo) => {
            // 玩家贏錢了 Server 告知目前金額。
        });
        // 押注
        return client.bet();
    })
    .catch((err) => {
        // 檢查錯誤代碼
        client.close();
    });
```
