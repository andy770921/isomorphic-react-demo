# 確保所有使用者自動更換到前端的新版本

### 問題描述
- 後端修改了 API 欄位, 且後端跟前端為不同專案 / 不同 Server
- 前端為了對應新欄位修改了程式碼, 當 Release 前端新版時, 部份使用者已經開啟了網頁
- 如何確保所有使用者自動更換到前端的新版本
### 解決方案  
後端建立版本號，前端可用以下幾種方式，在網頁運行途中，檢查版本號有無更新。若版本號有更新，就更新前端頁面 ( 最簡單的方式為 reload )
- Polling: 前端設定時間，定期發送 request 給後端，get 到版本號。若版本號更新，則更新頁面
```js
/* Client - subscribing to the version number */
subscribe: (callback) => {
    const pollVersion = () => {
        $.ajax({
            method: 'GET',
            url: 'http://localhost:8080/version', 
            success: (data) => {
                callback(data) // process the data
            },
            complete: () => {
                pollVersion();
            },
            timeout: 30000
        })
    }
    pollVersion();
}
```
- WebSockets: 網頁一但連上，使用 WebSockets 保持瀏覽器和伺服器的連線。若版本號更新，則更新頁面

# React 相關細節補充說明
## React 各頁面重新整理瀏覽器，畫面可以正確返回
### 純前端解法: React-Router 使用 HashRouter
- 如 ```http://example.com/#/name``` ，井字號後的文字，不會送給 server ，server 仍視為是```http://example.com/```，回傳 index page
- 前端的 React-Router 會處理井字號後的文字，達到正確導向到特定頁面
### 前後端共同解法: 使用 Isomorphic
- 前後端都要設定 Route ，如重新整理後的網址為 ```http://example.com/name``` ，後端伺服器需要知道該如何處理這網址
- Isomorphic 指前後端共用 JavaScript 的開發方式，可在後端伺服器設定不同網址要如何回應，如這個[範例程式](https://isomorphic-react-demo.firebaseapp.com/georgebrown)，在 ```functions/index.js```，使用 ```app.get('/:userId?', async (req, res) => {......});``` 設定不同網址的回應方式

```js
const app = require('express')();
const React = require('react');
const ReactDOMServer = require('react-dom/server');
const ServerApp = React.createFactory(require('./build/server.bundle.js').default);
const template = require('./template');
const database = require('./firebase-database');

const renderApplication = (url, res, initialState) => {
  const html = ReactDOMServer.renderToString(ServerApp({url: url, context: {}, initialState}));
  const templatedHtml = template({body: html, initialState: JSON.stringify(initialState)});
  res.send(templatedHtml);
};

app.get('/:userId?', async (req, res) => {
  res.set('Cache-Control', 'public, max-age=60, s-maxage=180');
  if (req.params.userId) {
    // client is requesting user-details page with userId
    // load the data for that employee and its direct reports
    const resp = await database.getEmployeeById(req.params.userId);
    renderApplication(req.url, res, resp);
  } else {
    // index page. load data for all employees
    const resp = await database.getAllEmployees();
    renderApplication(req.url, res, resp);
  }
});
```
- 最終在分頁重新整理後，畫面可以正確返回，如: https://isomorphic-react-demo.firebaseapp.com/georgebrown

## 重新整理瀏覽器，仍保留目前商品或列表頁面
- 將使用者操作，使用 JSON 格式紀錄在 object ，再存在 localStorage 中。使用者重新整理瀏覽器時，先去 localStorage 中取上次的資料，再呈現頁面
- 將使用者操作，送 post request 回後端，由後端紀錄。使用者重新整理瀏覽器時，前端依照後端的資料呈現畫面
## SEO 的需求
- server-side rendering 可以解決
- 實作方式 : 可以使用 React 供後端 server 用的函式庫 ```ReactDOMServer```，``` renderToString(element)``` 可以將 react element 轉換成 HTML 字串，在回傳給前端，再由前端接管使用者操作。可供搜尋引擎爬蟲 

