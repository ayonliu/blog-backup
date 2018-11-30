---
title: 用测试账户调试Google Ads API
tags:
- google ads
---
## 申请生产经理账号和开发者令牌

申请生产经理账号（production manager account）后可以申请生产经理开发者令牌（后面称 Developer Token），Developer Token 会让 AdWords API 识别你的应用。只有经过批准的令牌才能连接到生产 AdWords 帐号的 API；待处理令牌只能连接到测试帐号。一旦 Developer Token 获得批准，就可以将同一令牌用于针对所有 AdWords 帐号的请求，即使它们未关联到与此开发者令牌相关联的经理帐号，也就是说 Developer Token 不仅可以在正式环境中使用，也可以在测试环境中使用。
1. [创建生产经理帐号](https://ads.google.com/home/tools/manager-accounts/)
2. [在生产经理帐号中请求开发者令牌（production manager developer token）]
注册 AdWords API 时，系统会自动生成一个开发者令牌。提出申请后，令牌立刻进入待审批状态。在令牌获得批准后，可以指定在生产环境中使用的目标 AdWords 帐号。
(https://developers.google.com/adwords/api/docs/guides/signup)

请求成功后在 TOOLS -> API Center 中可以看到相应的 Developer Token。
![Developer Token](https://raw.githubusercontent.com/ayonliu/exercise/master/images/product-manager-account.png)

**可以使用开发者令牌来指定任何目标 AdWords 帐号，因此，只需使用目标帐号（或其某一个经理帐号，包括测试账号）的 OAuth2 凭据来授权你的请求即可（[见下文](#测试经理帐号（test-manager-account）)）。**

在下面的流程中，对测试经理帐号发出请求时，就会使用到这里的生产经理帐号开发者令牌。

## 测试经理帐号（test manager account）
* [申请测试经理帐号](https://adwords.google.com/um/Welcome/?sf=mt)
测试经理帐号申请成功，登录后可以看到一个红色的标签 Test account，说明是测试经理帐号，如果没有在 AdWords 帐号页上看到红色的“Test account”标签，那么该帐号是一个生产帐号。
![test account](https://raw.githubusercontent.com/ayonliu/exercise/master/images/test-manager-account.png)

## 申请测试客户账号（test client account）

测试经理帐号申请成功后才能进行这一步。
1. 使用 AdWords 网页界面，在之前创建的测试经理帐号下创建测试客户帐号。当您以测试经理帐号登录 AdWords 时，您创建的所有客户帐号都将自动成为测试帐号。
![create test client account](https://raw.githubusercontent.com/ayonliu/exercise/master/images/creat-test-client-account.png)
2. 记下新测试客户帐号的客户 ID （Client Customer ID）并保存：稍后您需要将其添加到配置文件中。客户帐号的 Client Customer ID 如下所示：
![client customer ID](https://raw.githubusercontent.com/ayonliu/exercise/master/images/test-client-customer-id.png)
3. 使用 AdWords 网页界面，在测试客户帐号下创建几个测试广告系列（test campaigns），比如我这里创建一个名为“test1”的 campaign，生成的 id 为1557850521（后面会用到）。

## 设置 OAuth2 身份验证，将 OAuth2 与测试帐号配合使用

应用必须先获取授予 API 访问权限的 OAuth2 访问令牌，才可以使用 API 访问非公开数据。通过 OAuth2 进行身份验证后可让应用代表你的帐号进行各种操作。要使用 OAuth2 访问测试帐号，测试经理帐号用户必须向客户端应用授予权限。授权后，应用就可以用授权账户提供的 OAuth2 客户端 ID （Client ID）和客户端密钥（Client Secret）来访问 API。

### 生成 OAuth2 凭据（Credentials）

1. 使用经理帐号凭据登录后，打开 [Google API 控制台凭据页面](https://console.developers.google.com/apis/credentials)。
2. 从项目下拉菜单中，选择 **新建项目**，输入项目的名称，然后点击 **创建**。
3. 选择 **创建凭据**，然后选择 **OAuth 客户端 ID**。
4. 点击 **配置同意屏幕(Configure consent screen)**，提供要求的信息，然后点击 **保存** 以返回到**凭据（Credentials）**屏幕。
5. 在**应用类型**下，选择**其他**。在提供的空间中输入名称。
6. 点击**创建**。系统会显示 OAuth2 客户端 ID （Client ID）和客户端密钥（Client Secret）。复制并保存这些信息。在下一步中将它们添加到配置文件中。
![oauth2 client id & client secret](https://raw.githubusercontent.com/ayonliu/exercise/master/images/oauth2-client-id-secret.png)

### 获取 OAuth2 刷新令牌（Refresh token）

由于 OAuth2 访问权限会在限定时间后过期，因此使用 OAuth2 刷新令牌来自动更新 OAuth2 访问权限。当请求 OAuth2 刷新令牌时，请确保您以测试经理帐号用户身份登录，以获取测试经理帐号对应的 OAuth2 客户端 ID （client ID）和客户端密钥（client secret）。在针对测试经理帐号提出请求时需要使用生产经理帐号的[开发者令牌](#申请生产经理账号和开发者令牌)。

1. 调用刷新令牌脚本，这里我用[Python版](https://github.com/googleads/googleads-python-lib/blob/master/examples/adwords/authentication/generate_refresh_token.py)的。 修改 generate_refresh_token.py 文件，把其中的 DEFAULT_CLIENT_ID，DEFAULT_CLIENT_SECRET 修改为[上面](#生成-OAuth2-凭据（Credentials）)刚刚创建好的 Client ID 和 Client Secret：
```
...
# Your OAuth2 Client ID and Secret. If you do not have an ID and Secret yet,
# please go to https://console.developers.google.com and create a set.
DEFAULT_CLIENT_ID = 'Your client id here'
DEFAULT_CLIENT_SECRET = 'Your client secret here'
...
```

2. 执行：`python generate_refresh_token.py`
3. 按照提示复制生成的 url 到浏览器，在到达的页面中用[测试经理帐号](#测试经理帐号（test-manager-account）)对应的账号登录, 下一步点击“允许”，会出现下图：
![access code](https://raw.githubusercontent.com/ayonliu/exercise/master/images/access_code.png)
4. 复制下图中的 code 粘贴到步骤2执行的命令行
![refresh token](https://raw.githubusercontent.com/ayonliu/exercise/master/images/refresh_token.png)
5. 记录下上步中的 Refresh Token。

生产环境中，从测试经理帐号切换到生产经理帐号，需要用生产经理帐号相关的 Client ID 和 Client Secret 来请求生产经理帐号的刷新令牌。

## 测试 ads API 功能

上面的步骤里我们已经准备好了调用 API 需要的必要配置：Developer Token, Client Customer ID, Client ID, Client Secret, Refresh Token。下面测试一个 API 调用：
1. 选择自己熟悉语言的[客户端库](https://developers.google.com/adwords/api/docs/clientlibraries)，这里用Python。
2. 配置文件（googleads.yaml）中填写正确的配置信息：
```
# AdWordsClient configurations
adwords:
  developer_token: Developer Token
  client_customer_id: Client Customer ID
  client_id: Client ID
  client_secret: Client Secret
  refresh_token: Refresh Token
```
3. 测试 [ad groups](https://github.com/googleads/googleads-python-lib/blob/master/examples/adwords/v201806/basic_operations/add_ad_groups.py) 功能：
   * 修改 add_ad_groups.py ：
```
...
CAMPAIGN_ID = 1557850521 # 上面创建测试客户帐号后添加的 campaign
...
if __name__ == '__main__':
  # Initialize client object.
  adwords_client = adwords.AdWordsClient.LoadFromStorage('你的 googleads.yaml 存放位置')

  main(adwords_client, CAMPAIGN_ID)

...
```
   * 命令行执行：`python add_ad_groups.py`
4. 执行后到测试客户账号下查看对应的 campaign 下的 ad groups 如下所示：
![test-ad-groups](https://raw.githubusercontent.com/ayonliu/exercise/master/images/test-ad-groups.png)

## 参考
* [管理帐号](https://developers.google.com/adwords/api/docs/guides/accounts-overview#test_accounts)
* [OAuth2 身份验证](https://developers.google.com/adwords/api/docs/guides/authentication#generate_oauth2_credentials)
* [进行首次 API 调用](https://developers.google.com/adwords/api/docs/guides/first-api-call)
* [API 典型使用案例](https://developers.google.com/adwords/api/docs/guides/call-structure)