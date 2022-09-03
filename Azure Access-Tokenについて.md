### 待确认

1.获取access token的timing是什么？

2.access token的生存期是多长时间？

3.如果access token过期后的处理？是否使用refresh token？ → 如果使用refresh token，在获取access token时需一并存入DB 

***

### OAuth 2.0

https://docs.microsoft.com/zh-cn/azure/active-directory/develop/v2-oauth2-auth-code-flow

#### 1. 请求授权代码

```http
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read%20api%3A%2F%2F
&state=12345
&code_challenge=YTFjNjI1OWYzMzA3MTI4ZDY2Njg5M2RkNmVjNDE5YmEyZGRhOGYyM2IzNjdmZWFhMTQ1ODg3NDcxY2Nl
&code_challenge_method=S256
```

| 参数                    | 必需/可选     | 说明                                                         |
| :---------------------- | :------------ | :----------------------------------------------------------- |
| `tenant`                | **必需**      | 请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。 有效值为 `common`、`organizations`、`consumers` 和租户标识符。 对于用户从一个租户登录到另一个租户的来宾场景，必须提供租户标识符才能让其登录到资源租户。 有关详细信息，请参阅[终结点](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/active-directory-v2-protocols#endpoints)。 |
| `client_id`             | **必填**      | [Azure 门户 – 应用注册](https://go.microsoft.com/fwlink/?linkid=2083908)体验分配给你的应用的应用程序（客户端）ID。 |
| `response_type`         | **必填**      | 必须包括授权代码流的 `code` 。 如果使用[混合流](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/v2-oauth2-auth-code-flow#request-an-id-token-as-well-or-hybrid-flow)，则还可以包括 `id_token` 或 `token`。 |
| `redirect_uri`          | **必需**      | 应用的 `redirect_uri`，你的应用可通过该应用发送和接收身份验证响应。 其必须完全符合在门户中注册的其中一个重定向 URI，否则必须是编码的 URL。 对于本机应用和移动应用，请使用一个建议的值：`https://login.microsoftonline.com/common/oauth2/nativeclient`（适用于使用嵌入式浏览器的应用）或 `http://localhost`（适用于使用系统浏览器的应用）。 |
| `scope`                 | **必需**      | 希望用户同意的[范围](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/v2-permissions-and-consent)的空格分隔列表。 对于请求的 `/authorize` 分支，此参数可以涵盖多个资源。 此值允许应用获取你要调用的多个 Web API 的同意。 |
| `response_mode`         | 建议          | 指定标识平台应如何将请求的令牌返回到应用。  支持的值：  - `query`：请求访问令牌时的默认值。 在重定向 URI 上提供代码作为查询字符串参数。 使用隐式流请求 ID 令牌时不支持该 `query` 参数。 - `fragment`：通过使用隐式流请求 ID 令牌时的默认值。 如果*只*请求一个代码，也支持。 - `form_post`：对重定向 URI 执行包含代码的 POST。 请求代码时支持。 |
| `state`                 | 建议          | 同时随令牌响应返回的请求中所包含的值。 可以是想要的任何内容的字符串。 随机生成的唯一值通常用于 [防止跨站点请求伪造攻击](https://tools.ietf.org/html/rfc6749#section-10.12)。 该值还可用于在身份验证请求发生前，对有关用户在应用中的状态信息进行编码。 例如，它可以对用户所在的页面或视图进行编码。 |
| `prompt`                | 可选          | 表示需要的用户交互类型。 有效值为 `login`、`none`、`consent` 和 `select_account`。  - `prompt=login` 强制用户在该请求上输入其凭据，从而使单一登录无效。 - `prompt=none` 则相反。 它确保不向用户显示任何交互式提示。 如果请求无法通过单一登录以无提示方式完成，则 Microsoft 标识平台将返回 `interaction_required` 错误。 - `prompt=consent` 在用户登录后触发 OAuth 同意对话框，要求用户向应用授予权限。 - `prompt=select_account` 将中断单一登录，提供帐户选择体验，列出会话或任何记住的帐户中的所有帐户，或者提供选择一起使用其他帐户的选项。 |
| `login_hint`            | 可选          | 可使用此参数预先填充用户登录页面的用户名和电子邮件地址字段。 应用在已经从前次登录提取 `login_hint`[可选声明](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/active-directory-optional-claims)后，可在重新身份验证时使用此参数。 |
| `domain_hint`           | 可选          | 如果包含，应用将跳过用户在登录页面上经历的基于电子邮件的发现过程，导致稍微更加流畅的用户体验。 例如，将其发送到其联合标识提供者。 应用可以在重新身份验证期间使用此参数，方法是从前次登录提取 `tid`。 |
| `code_challenge`        | **建议/必需** | 用于通过“用于代码交换的证明密钥”(PKCE) 来保护授权代码授予。 如果包含 `code_challenge_method`，则需要。 有关详细信息，请参阅 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。 目前建议将参数用于所有应用程序类型（公共和机密客户端），Microsoft 标识平台则要求[使用授权代码流的单页应用](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/reference-third-party-cookies-spas)使用此参数。 |
| `code_challenge_method` | **建议/必需** | 用于为 `code_challenge` 参数编码 `code_verifier` 的方法。 此方法应为 `S256`，但是如果客户端不能支持 SHA256，则该规范允许使用 `plain`。  如果已排除在外，且包含了 `code_challenge`，则假定 `code_challenge` 为纯文本。 Microsoft 标识平台支持 `plain` 和 `S256`。 有关详细信息，请参阅 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。 [使用授权代码流的单页应用](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/reference-third-party-cookies-spas)需要此参数。 |

- #### 成功的响应

  ```http
  GET http://localhost?
  code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
  &state=12345
  ```

| 参数    | 说明                                                         |
| :------ | :----------------------------------------------------------- |
| `code`  | 应用请求的 `authorization_code`。 应用可以使用授权代码请求目标资源的访问令牌。 授权代码的生存期很短。 通常，它们在约 10 分钟后过期。 |
| `state` | 如果请求中包含 `state` 参数，则响应中应显示相同的值。 应用应该验证请求和响应中的 state 值是否完全相同。 |

- #### 错误响应

  ```http
  GET http://localhost?
  error=access_denied
  &error_description=the+user+canceled+the+authentication
  ```

| 参数                | 说明                                                         |
| :------------------ | :----------------------------------------------------------- |
| `error`             | 一个错误代码字符串，可用于对错误类型进行分类并对错误做出反应。 提供此错误部分是使应用能够对错误做出相应的反应，但不会深入解释错误发生的原因。 |
| `error_description` | 帮助开发人员识别身份验证错误原因的特定错误消息。 包含有关为何发生错误的大部分有用信息。 |



#### 2. 使用证书凭据请求访问令牌

```Http
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https://graph.microsoft.com/mail.read
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=http://localhost/myapp/
&grant_type=authorization_code
&code_verifier=ThisIsntRandomButItNeedsToBe43CharactersLong
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
```

| 参数                    | 必需/可选         | 说明                                                         |
| :---------------------- | :---------------- | :----------------------------------------------------------- |
| `tenant`                | **必需**          | 请求路径中的 `{tenant}` 值可用于控制哪些用户可以登录应用程序。 有效值为 `common`、`organizations`、`consumers` 和租户标识符。 有关更多详细信息，请参阅[终结点](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/active-directory-v2-protocols#endpoints)。 |
| `client_id`             | 必需              | 在 [Azure 门户 – 应用注册](https://go.microsoft.com/fwlink/?linkid=2083908)页中分配给应用的应用程序（客户端）ID。 |
| `scope`                 | 可选              | 范围的空格分隔列表。 范围必须全部来自单个资源，以及 OIDC范围（`profile`、`openid`、`email`）。 有关详细信息，请参阅[权限、同意和范围](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/v2-permissions-and-consent)。 此参数是授权代码流的 Microsoft 扩展。 应用可通过此扩展在令牌兑换期间声明它们需要获取哪个资源的令牌。 |
| `code`                  | **必需**          | 在流的第一个阶段获取的 `authorization_code`。                |
| `redirect_uri`          | **必需**          | 用于获取 `authorization_code` 的相同 `redirect_uri` 值。     |
| `grant_type`            | 必需              | 必须是授权代码流的 `authorization_code` 。                   |
| `code_verifier`         | 建议              | 用于获取 `authorization_code` 的同一 `code_verifier`。 如果在授权码授权请求中使用 PKCE，则需要。 有关详细信息，请参阅 [PKCE RFC](https://tools.ietf.org/html/rfc7636)。 |
| `client_assertion_type` | 机密 Web 应用所需 | 必须将值设置为 `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` 才能使用证书凭据。 |
| `client_assertion`      | 机密 Web 应用所需 | 断言（一个 JSON Web 令牌 (JWT)），需使用作为凭据向应用程序注册的证书进行创建和签名。 有关如何注册证书以及断言的格式，请阅读[证书凭据](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/active-directory-certificate-credentials)的相关信息。 |

- ### 成功的响应

  ```json
  {
      "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
      "token_type": "Bearer",
      "expires_in": 3599,
      "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
      "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
      "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
  }
  ```

| 参数            | 说明                                                         |
| :-------------- | :----------------------------------------------------------- |
| `access_token`  | 请求的访问令牌。 应用可以使用此令牌对受保护的资源（如 Web API）进行身份验证。 |
| `token_type`    | 指示令牌类型值。 Azure AD 支持的唯一类型是 `Bearer`。        |
| `expires_in`    | 访问令牌的有效期（以秒为单位）。                             |
| `scope`         | `access_token` 的有效范围。 可选。 此参数是非标准的，如果省略，令牌将用于流的初始阶段中所请求的范围。 |
| `refresh_token` | OAuth 2.0 刷新令牌。 应用可以使用此令牌，在当前访问令牌过期之后获取其他访问令牌。 刷新令牌的生存期较长。 它们可以长期保留对资源的访问权限。 有关刷新访问令牌的详细信息，请参阅本文稍后的[刷新访问令牌](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/v2-oauth2-auth-code-flow#refresh-the-access-token)。 **注意：** 仅当已请求 `offline_access` 作用域时提供。 |
| `id_token`      | 一个 JSON Web 令牌。 应用可以解码此令牌的段，以请求有关登录用户的信息。 应用可以缓存并显示值，机密客户端可以使用此令牌进行授权。 有关 id_tokens 的详细信息，请参阅 [`id_token reference`](https://docs.microsoft.com/zh-cn/azure/active-directory/develop/id-tokens)。 **注意：** 仅当已请求 `openid` 作用域时提供。 |

- ### 错误响应

  ```json
  {
    "error": "invalid_scope",
    "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
    "error_codes": [
      70011
    ],
    "timestamp": "2016-01-09 02:02:12Z",
    "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
    "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
  }
  ```

| 参数                | 说明                                                         |
| :------------------ | :----------------------------------------------------------- |
| `error`             | 一个错误代码字符串，可用于对错误类型进行分类并对错误做出反应。 |
| `error_description` | 帮助开发人员识别身份验证错误原因的特定错误消息。             |
| `error_codes`       | 可帮助诊断的 STS 特定错误代码列表。                          |
| `timestamp`         | 发生错误的时间。                                             |
| `trace_id`          | 可帮助诊断的请求唯一标识符。                                 |
| `correlation_id`    | 可帮助跨组件诊断的请求唯一标识符。                           |





