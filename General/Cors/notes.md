## Cors

- [header 清單](#header清單)

<br/>

### header 清單

- 一般範例

```JSON
{
    "Access-Control-Allow-Origin": "https://host",
    "Access-Control-Allow-Methods": "POST, GET, OPTIONS",
    "Access-Control-Allow-Headers": "User-Agent, Authorization, Content-Length, X-CSRF-Token, Origin, Content-Type, Accept, Cache-Control",
    "Access-Control-Max-Age": "172800",
    "Access-Control-Allow-Credentials": "true"
}
```

<br/>

`Access-Control-Allow-Origin`：允許跨域的來源
`Access-Control-Allow-Methods`：允許跨域來源使用的請求方法
`Access-Control-Allow-Headers`：允許跨域來源使用的標頭
`Access-Control-Max-Age`：允許跨域來源快取 preflight 的時間
`Access-Control-Allow-Credentials`：允許跨域來源攜帶 Cookie
`Access-Control-Expose-Headers`：允許跨域來源讀取的標頭

`Access-Control-Allow-Headers` 的常用選項：

- `User-Agent` - 向伺服器表明請求的來源程式（瀏覽器、伺服器等）信息
- `Origin` - 提供發送請求的頁面的來源域名
- `Authorization` - 用於認證的頭部，如 Bearer 令牌
- `Content-Type` - 指示資源的 MIME 類型
- `Content-Length` - 用於指示響應實體主體的長度
- `X-CSRF-Token` - 用於阻止 CSRF 攻擊的夾帶 Token
- `Cache-Control` - 緩存控制
- `Accept` - 指定客戶端可以接受的響應內容類型
