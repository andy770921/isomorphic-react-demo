# 使用者自動更換到前端的新版本

### 解決方案  
後端建立版本號，前端可用以下幾種方式，在網頁運行途中，檢查版本號有無更新。  
若版本號有更新，就更新前端頁面 ( 最簡單的方式為 reload )  
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

# React 重整頁面，正確顯示畫面 

### 純前端解法：HashRouter
- 如 ```http://example.com/#/name``` ，井字號後的文字，不會送給 server ，server 視為 ```http://example.com/```，回傳 index page
- 前端的 React-Router 會處理井字號後的文字，達到正確導向
### 前後端共同解法： Isomorphic
- 前後端都要設定 Route ，如重新整理後的網址為 ```http://example.com/name``` ，後端伺服器需要知道該如何處理這網址
- Isomorphic 指前後端共用 JavaScript 的開發方式，可在後端伺服器設定不同網址要如何回應
- 連結有 [設定分頁的程式碼 Repo](https://github.com/andy770921/isomorphic-react-demo/blob/master/functions/index.js)，擷取相關部分如下
- 使用 ```app.get('/:userId?', async (req, res) => {......});``` 設定不同網址的回應方式

```js
const app = require('express')();

app.get('/:userId?', async (req, res) => {
  res.set('Cache-Control', 'public, max-age=60, s-maxage=180');
  if (req.params.userId) {
    // client is requesting user-details page with userId
    const resp = await database.getEmployeeById(req.params.userId);
    renderApplication(req.url, res, resp);
  } else {
    // index page. load data for all employees
    const resp = await database.getAllEmployees();
    renderApplication(req.url, res, resp);
  }
});
```
- 最終在分頁重新整理後，畫面可以正確返回，如 [範例](https://isomorphic-react-demo.firebaseapp.com/georgebrown) 

# 重整頁面，保留目前商品或列表
- 將使用者操作，用 JSON 格式紀錄在 object ，stringify 後再存在 localStorage 中。
- 使用者重新整理瀏覽器時，先去 localStorage 中取上次的資料，再呈現頁面
- 若不用 localStorage ，可送 post request 回後端，由後端紀錄。使用者重新整理瀏覽器時，前端依照後端的資料呈現畫面
# React 的 SEO 優化
- server-side rendering 可以解決
- 可以使用 React 供後端 server 用的函式庫 ```ReactDOMServer```，```renderToString(element)``` 
- 將 react element 轉換成 HTML 字串，再回傳給前端，再由前端接管使用者操作。
- 連結有 [實作 renderToString](https://github.com/andy770921/isomorphic-react-demo/blob/master/functions/index.js) 的範例，後端將 [HTML](https://github.com/andy770921/isomorphic-react-demo/blob/master/functions/template.js) 送給前端
- HTML 其中一部份 ```<script src='/assets/client.bundle.js'></script>``` ，讓前端 client.bundle.js 接管



