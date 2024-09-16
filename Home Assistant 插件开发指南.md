Home Assistant OAuth2认证插件开发指南

Feb 2 2024

# 介绍

开源的智能家居管理系统，适合喜欢DIY的用户。

- 插件丰富
- 私有化部署、本地保存数据
- 基于python、开源

demo: https://demo.home-assistant.io

# integration

也就是插件。https://www.home-assistant.io/integrations/。在Home Assistant里面叫做integration

开发者文档：https://developers.home-assistant.io/

integration开发文档：https://developers.home-assistant.io/docs/development_index

# 环境配置

## 开发环境配置

开发环境搭建看文档即可，推荐使用基于Docker的VSCode的DevContainer。无需手动配置，容器会自动配置好所有的环境。

不过Docker在windows上运行需要配置WSL（Windows Subsystem for Linux）。相关教程可参考https://learn.microsoft.com/en-us/windows/wsl/install。而Windows Docker的安装可参考https://docs.docker.com/desktop/install/windows-install/.   

此外，WSL还区分1和2两个版本，以上提到的都是新版WSL2。而新版WSL要求新的Windows系统。虽然Windows 10部分系统也已经支持了WSL2，但出于方便起见，尽量使用Windows 11开发。

此外，以上提到的都是本地的Docker环境，VSCode的DevContainer也支持远程主机上的Docker，详见https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers。[^system]

按照官方文档，fork github仓库并打开，VSCode会自动打开项目并且安装所有插件，并配置Docker中的容器。之后在VSCode最近中可以找到名为core的项目，之后开发只需要打开这个项目就可以自动加载环境。前提是保证Docker运行。

![image-20240129103042515](./assets/image-20240129103042515.png)

## 网络环境配置

由于自动配置需要访问 github 以及 docker hub，请尽量使用代理。好在 docker desktop 中可以很方便的配置。[^proxy] 其他需要注意的是，DevContainer是WSL虚拟机中的虚拟机。如果需要访问宿主机上的网络，最好用外网转发。

![image-20240129103528418](./assets/image-20240129103528418.png)

另外，可以使用```curl cip.cc```命令来检查环境的当前外网IP[^cip]。

```shell
vscode ➜ /workspaces/core (dev) $ curl cip.cc
IP      : 170.187.198.109
地址    : 美国  美国

数据二  : 美国

数据三  : 美国伊利诺伊

URL     : http://www.cip.cc/170.187.198.109
```

[^system]:当然，使用MacOS或者Linux桌面系统开发也是可行的，不再赘述。
[^proxy]:如果此方法不可行，建议使用环境变量```ALL_PROXY```配置代理。```set ALL_PROXY=http://127.0.0.1:7890```；或者使用虚拟网卡代理等方法。
[^cip]: cip.cc是个很有用的检查外网IP的网站。

# 运行以及DEBUG

按F5即可运行Home Assistant实例并进入Debug。其中左上角展示当前的变量，右下角OUTPUT的DEBUG CONSOLE可以执行变量的表达式。由于python中的一切都是对象，所以所有变量的都有一些特殊属性，这部分可以忽略。[^shortcut]

如果不需要DEBUG，打开命令面板，选择Tasks: Run Task` -> `Run Home Assistant Core也可运行项目。

![image-20240129105441355](./assets/image-20240129105441355.png)

首次运行需要设置用户名及密码，如果忘记密码了，就只能删除容器重新配置。

实例地址：http://localhost:8123

[^shortcut]:与Jetbrains IDE的DEBUG快捷键不同，F10是执行下一步，F11是进入方法，F5是继续程序。

# 如何下手

互联网上开发Home Assistantintegration的资料很少，只有官方文档以及部分论坛的帖子可供参考，而且都是英文的。所以**最重要的参考资料是其他integration的代码**，而且这些代码都在同一个仓库里，非常方便检阅。

此外，另外一个重要部分是，功能的具体实现要使用发布在pypi的python依赖包，参考[Python依赖](https://developers.home-assistant.io/docs/api_lib_index)。

# 创建integration

## 脚手架配置

使用以下命令可以创建框架代码，并可直接用交互方式配置。

```shell
python3 -m script.scaffold integration

# 跟上条命令唯一的区别是：会不经过询问直接生成 OAUTH2 的框架代码
python3 -m script.scaffold config_flow_oauth2
```

参考[配置流程](https://developers.home-assistant.io/docs/creating_integration_manifest/)：

```shell
vscode ➜ /workspaces/core (dev) $ python3 -m script.scaffold integration

What is the domain?
# domain 是指integration的唯一标识
> my_integration

What is the name of your integration?
> My Integration

What is your GitHub handle?   
# github handle 是指 github 用户名，注意前面必须加@
> @xxxxx

What PyPI package and version do you depend on? Leave blank for none.
# 一般的integration都是需要额外开发一套 PYTHON 依赖包，用来给integration调用。
> my_pypi_package==0.0.1

How will your integration gather data?
# 数据源是什么，通常分成本地数据和云数据。

Valid values are assumed_state, calculated, cloud_polling, cloud_push, local_polling, local_push

More info @ https://developers.home-assistant.io/docs/creating_integration_manifest#iot-class

> cloud_polling

Does Home Assistant need the user to authenticate to control the device/service? (yes/no) [yes]
> yes

Is the device/service discoverable on the local network? (yes/no) [no]
> no

Is this a helper integration? (yes/no) [no]
# integration类型见 https://developers.home-assistant.io/docs/creating_integration_manifest#integration-type
> no

Can the user authenticate the device using OAuth2? (yes/no) [no]
> yes
```

integration类型默认情况下都是HUB，可以包含多设备、实体、服务等等，方便拓展。

此外，该自动脚本会生成测试代码等等其他的配置。

## 手动配置

命令创建流程还会生成一个manifest.json配置文件，用于integration的配置，该文件也可以手动修改。参考[配置含义](https://developers.home-assistant.io/docs/creating_integration_manifest/)。

注意python依赖是其中的requirements，而不是dependecies。config_flow则表示用户加载integration的时候使用GUI配置还是YAML配置。

```json
{
  "domain": "my_integration",
  "name": "My Integration",
  "codeowners": [
    "@xxxxxx"
  ],
  "config_flow": true,
  "dependencies": [
    "application_credentials"
  ],
  "documentation": "https://www.home-assistant.io/integrations/my_integration",
  "homekit": {},
  "iot_class": "cloud_polling",
  "requirements": ["my_pypi_package==0.0.1"],
  "ssdp": [],
  "zeroconf": []
}
```

另外，integration代码中还包括一个strings.json文件，该配置用于配置程序运行过程中的各种提示语。可以结合下文的async_get_description_placeholders方法使用。

# 认证

如果不需要OAuth2认证，则在```config_flow```中写自定义的认证逻辑即可。以下是OAuth2授权码模式的的认证实现说明。

## 介绍

本地部署的Home Assistant支持OAuth2的授权码模式，只要根据脚手架代码实现，就可以完整实现授权流程。其中的重定向URI由Home Assistant官方网站提供。这部分可以参考integration```electric_kiwi```开发。

默认情况下，OAuth2所需的客户端凭证需要用户手动填写，然后进行后续流程；也可以如同```electric_kiwi```integration一样在integration内直接提供客户端凭证，这种情况下用户体验自然更好。

根据官方文档[应用凭据](https://developers.home-assistant.io/docs/core/platform/application_credentials)，Home Assistant的oauth2实际上分成两种模式，本地模式和云模式。

1. 本地模式：用户输入客户端凭据。
2. 云模式：客户端凭据保存在付费服务 Home Assistant cloud 中的。

一般情况下，使用只使用本地模式。显然这样做的用户体验不佳，这也是Home Assistant用来推销云服务的手段。不过下文会提到如何绕过这一限制，实现无缝连接。

## 代码

oauth2代码框架会生成以下三个文件，下面分别解释每个文件的作用。

### api.py

```python
"""API for My Integrationloud bound to Home Assistant OAuth."""
from asyncio import run_coroutine_threadsafe

from aiohttp import ClientSession
import my_pypi_package

from homeassistant.core import HomeAssistant
from homeassistant.helpers import config_entry_oauth2_flow

# TODO the following two API examples are based on our suggested best practices
# for libraries using OAuth2 with requests or aiohttp. Delete the one you won't use.
# For more info see the docs at https://developers.home-assistant.io/docs/api_lib_auth/#oauth2.


class ConfigEntryAuth(my_pypi_package.AbstractAuth):
    """Provide My Integrationloud authentication tied to an OAuth2 based config entry."""

    def __init__(
        self,
        Home Assistantss: HomeAssistant,
        oauth_session: config_entry_oauth2_flow.OAuth2Session,
    ) -> None:
        """Initialize My Integrationloud Auth."""
        self.hass = hass
        self.session = oauth_session
        super().__init__(self.session.token)

    def refresh_tokens(self) -> str:
        """Refresh and return new My Integrationloud tokens using Home Assistant OAuth2 session."""
        run_coroutine_threadsafe(
            self.session.async_ensure_token_valid(), self.hass.loop
        ).result()

        return self.session.token["access_token"]


class AsyncConfigEntryAuth(my_pypi_package.AbstractAuth):
    """Provide My Integrationloud authentication tied to an OAuth2 based config entry."""

    def __init__(
        self,
        websession: ClientSession,
        oauth_session: config_entry_oauth2_flow.OAuth2Session,
    ) -> None:
        """Initialize My Integrationloud auth."""
        super().__init__(websession)
        self._oauth_session = oauth_session

    async def async_get_access_token(self) -> str:
        """Return a valid access token."""
        if not self._oauth_session.valid_token:
            await self._oauth_session.async_ensure_token_valid()

        return self._oauth_session.token["access_token"]

```

其中API中封装了已经被oauth2授权的HTTP请求，其中存在两个类：ConfigEntryAuth和AsyncConfigEntryAuth。这两个类只用其中一个即可，区别在于第一个使用的是同步的requests http依赖，第二个使用的是异步的aiohttp http依赖。

而这两个类继承自my_pypi_package.AbstractAuth。这里的my_pypi_package也就是上述提到了额外的pytho依赖包。而此处的 AbstractAuth 则实现了有授权request方法，相关[示例](https://developers.home-assistant.io/docs/api_lib_auth)。通过修改该request方法，可以自定义请求中的参数。

```async_get_access_token```是个抽象方法，就是获取ACCESS_TOKEN的意思。因为在OAUTH2认证中，ACCESS_TOKEN会不断的过期，需要不断的获取（刷新）。

```python
from abc import ABC, abstractmethod


class AbstractAuth(ABC):
    """Abstract class to make authenticated requests."""

    def __init__(self, websession: ClientSession, host: str):
        """Initialize the auth."""
        self.websession = websession
        self.host = host

    @abstractmethod
    async def async_get_access_token(self) -> str:
        """Return a valid access token."""

    async def request(self, method, url, **kwargs) -> ClientResponse:
        """Make a request."""
        headers = kwargs.get("headers")

        if headers is None:
            headers = {}
        else:
            headers = dict(headers)

        access_token = await self.async_get_access_token()
        headers["authorization"] = f"Bearer {access_token}"

        return await self.websession.request(
            method, f"{self.host}/{url}", **kwargs, headers=headers,
        )
```

### application_credentials

```python
"""application_credentials platform the My Integrationloud integration."""

from homeassistant.components.application_credentials import AuthorizationServer
from homeassistant.core import HomeAssistant

# TODO Update with your own urls
OAUTH2_AUTHORIZE = "https://www.example.com/auth/authorize"
OAUTH2_TOKEN = "https://www.example.com/auth/token"


async def async_get_authorization_server(hass: HomeAssistant) -> AuthorizationServer:
    """Return authorization server."""
    return AuthorizationServer(
        authorize_url=OAUTH2_AUTHORIZE,
        token_url=OAUTH2_TOKEN,
    )

```

除了文件中已有的一个方法async_get_authorization_server，该方法返回OAUTH2授权服务器的信息。

根据[应用凭据](https://developers.home-assistant.io/docs/core/platform/application_credentials)，还可以增加async_get_auth_implementation方法，用于返回实现OAUTH2认证的类。

async_get_description_placeholders方法用于返回描述中的占位符，在示例中给出的是引导用户获取客户端凭证的URL。

一个完整的例子如下：

```python
"""application_credentials platform the My Integrationloud integration."""

from homeassistant.components.application_credentials import (
    AuthorizationServer,
    ClientCredential,
)
from homeassistant.core import HomeAssistant
from homeassistant.helpers import config_entry_oauth2_flow

from .const import OAUTH2_AUTHORIZE, OAUTH2_TOKEN
from .oauth2 import MyIntegrationLocalOAuth2Implementation


async def async_get_auth_implementation(
    hass: HomeAssistant, auth_domain: str, credential: ClientCredential
) -> config_entry_oauth2_flow.AbstractOAuth2Implementation:
    """Return auth implementation."""
    return MyIntegrationLocalOAuth2Implementation(
        hass,
        auth_domain,
        credential,
        authorization_server=await async_get_authorization_server(hass),
    )


async def async_get_authorization_server(hass: HomeAssistant) -> AuthorizationServer:
    """Return authorization server."""
    return AuthorizationServer(
        authorize_url=OAUTH2_AUTHORIZE,
        token_url=OAUTH2_TOKEN,
    )


async def async_get_description_placeholders(hass: HomeAssistant) -> dict[str, str]:
    """Return description placeholders for the credentials dialog."""
    return {"more_info_url": "https://www.home-assistant.io/integrations/solaxcloud/"}

```

### config_flow

```python
"""Config flow for My Integrationloud."""
import logging

from homeassistant.helpers import config_entry_oauth2_flow

from .const import DOMAIN


class OAuth2FlowHandler(
    config_entry_oauth2_flow.AbstractOAuth2FlowHandler, domain=DOMAIN
):
    """Config flow to handle My Integrationloud OAuth2 authentication."""

    DOMAIN = DOMAIN

    @property
    def logger(self) -> logging.Logger:
        """Return logger."""
        return logging.getLogger(__name__)
```

config_flow实际上是整个integration第一次加载的配置流，如果不使用oauth2，那么也可以使用该类进行用户引导配置。通过继承不同的预设类，就可以实现不同的integration配置流程。实际上通过继承AbstractOAuth2FlowHandler就已经完成了oauth2的所有配置，Home Assistant已经实现了全部的oauth2流程。

不过考虑到认证失败的情况，官方文档还是提供了几个再次认证的方法需要实现。参考[重认证](https://developers.home-assistant.io/docs/config_entries_config_flow_handler#reauthentication)。例子如下：

```python

class OAuth2FlowHandler(
    config_entry_oauth2_flow.AbstractOAuth2FlowHandler, domain=DOMAIN
):
    """Config flow to handle OAuth2 authentication."""

    reauth_entry: ConfigEntry | None = None

    DOMAIN = DOMAIN

    @property
    def logger(self) -> logging.Logger:
        """Return logger."""
        return logging.getLogger(__name__)

    async def async_step_reauth(self, user_input=None):
        """Perform reauth upon an API authentication error."""
        self.reauth_entry = self.hass.config_entries.async_get_entry(
            self.context["entry_id"]
        )
        return await self.async_step_reauth_confirm()

    async def async_step_reauth_confirm(self, user_input=None):
        """Dialog that informs the user that reauth is required."""
        if user_input is None:
            return self.async_show_form(
                step_id="reauth_confirm",
                data_schema=vol.Schema({}),
            )
        return await self.async_step_user()

    async def async_oauth_create_entry(self, data: dict) -> dict:
        """Create an oauth config entry or update existing entry for reauth."""
        if self._reauth_entry:
            self.hass.config_entries.async_update_entry(self.reauth_entry, data=data)
            await self.hass.config_entries.async_reload(self.reauth_entry.entry_id)
            return self.async_abort(reason="reauth_successful")
        return await super().async_oauth_create_entry(data)
```

## 本地OAUTH2实现

虽然Home Assistant已经提供了一套OAuth2的实现，但是通过继承可以构造自定义的实现类。

例子：oauth2.py

```python
"""OAuth2 implementations for Toon."""
from __future__ import annotations

import base64
from typing import Any, cast

from homeassistant.components.application_credentials import (
    AuthImplementation,
    AuthorizationServer,
    ClientCredential,
)
from homeassistant.core import HomeAssistant
from homeassistant.helpers.aiohttp_client import async_get_clientsession

from .const import SCOPE_VALUES


class MyIntegrationLocalOAuth2Implementation(AuthImplementation):
    """Local OAuth2 implementation for My Integration."""

    def __init__(
        self,
        hass: HomeAssistant,
        domain: str,
        client_credential: ClientCredential,
        authorization_server: AuthorizationServer,
    ) -> None:
        """Set up Electric Kiwi oauth."""
        super().__init__(
            hass=hass,
            auth_domain=domain,
            credential=client_credential,
            authorization_server=authorization_server,
        )

        self._name = client_credential.name

    @property
    def extra_authorize_data(self) -> dict[str, Any]:
        """Extra data that needs to be appended to the authorize url."""
        return {"scope": SCOPE_VALUES}

    async def async_resolve_external_data(self, external_data: Any) -> dict:
        """Initialize local My Integration auth implementation."""
        data = {
            "grant_type": "authorization_code",
            "code": external_data["code"],
            "redirect_uri": external_data["state"]["redirect_uri"],
        }

        return await self._token_request(data)

    async def _async_refresh_token(self, token: dict) -> dict:
        """Refresh tokens."""
        data = {
            "grant_type": "refresh_token",
            "refresh_token": token["refresh_token"],
        }

        new_token = await self._token_request(data)
        return {**token, **new_token}

    async def _token_request(self, data: dict) -> dict:
        """Make a token request."""
        session = async_get_clientsession(self.hass)
        client_str = f"{self.client_id}:{self.client_secret}"
        client_string_bytes = client_str.encode("ascii")

        base64_bytes = base64.b64encode(client_string_bytes)
        base64_client = base64_bytes.decode("ascii")
        headers = {"Authorization": f"Basic {base64_client}"}
        resp = await session.post(self.token_url, data=data, headers=headers)
        resp.raise_for_status()
        resp_json = cast(dict, await resp.json())
        return resp_json
```

其中的方法的解释：

1. async_resolve_external_data：实际上是oauth2流程中的客户端向授权服务器申请access_token和refresh_token。
2. _token_request：发起一个带有access_token的请求。与原版不同的地方在于，该方法的请求带有authorization。

## 绕过客户端凭证填写

实际上Home Assistant只要保存过一遍客户端凭据，下次加载集成就会直接跳转登陆URL。而客户端凭据可以在该页面管理：http://localhost:8123/config/application_credentials。这意味着，Home Assistant本身就支持客户端凭据的储存，只是需要找到办法直接存。

以下展示分析路径，不想看分析可直接看代码实现。

### 集成配置入口

加载一个集成的入口都是在config_flow中：当用户选择一个集成，首先会执行config_flow中自定义的config_flow类中的```async_step_user```方法，该方法代表了用户配置。

实际上如果不使用 Home Assistant的OAuth2配置，config_flow文件的写法是需要继承```homeassistant.config_entries.ConfigFlow```类，并实现```async_step_user```方法。参考[Config Flow](https://developers.home-assistant.io/docs/config_entries_config_flow_handler)。

### OAuth2 Config Flow分析

自定义的```Oauth2FlowHandler```继承自```config_entry_oauth2_flow.AbstractOAuth2FlowHandler```类。方法调用顺序如下：

1. ```AbstractOAuth2FlowHandler.async_step_user```
2. ```AbstractOAuth2FlowHandler.async_step_pick_implementation```
3. ```config_entry_oauth2_flow.async_get_implementations```
4. 省略

上述方法的implentation也就是指```AbstractOAuth2Implementation```的实现。

给出```async_step_pick_implementation```的代码：

```python
async def async_step_pick_implementation(
        self, user_input: dict | None = None
    ) -> FlowResult:
        """Handle a flow start."""
        implementations = await async_get_implementations(self.hass, self.DOMAIN)

        if user_input is not None:
            self.flow_impl = implementations[user_input["implementation"]]
            return await self.async_step_auth()

        if not implementations:
            if self.DOMAIN in await async_get_application_credentials(self.hass):
                return self.async_abort(reason="missing_credentials")
            return self.async_abort(reason="missing_configuration")

        req = http.current_request.get()
        if len(implementations) == 1 and req is not None:
            # Pick first implementation if we have only one, but only
            # if this is triggered by a user interaction (request).
            self.flow_impl = list(implementations.values())[0]
            return await self.async_step_auth()

        return self.async_show_form(
            step_id="pick_implementation",
            data_schema=vol.Schema(
                {
                    vol.Required(
                        "implementation", default=list(implementations)[0]
                    ): vol.In({key: impl.name for key, impl in implementations.items()})
                }
            ),
        )
```

通过对不同xbox以及electric_kiwi及solax_cloud集成Debug可知，如果上述方法中的变量implementations是空的，那么就会进入到用户手动填写客户端凭据的流程。联系到application_credentials文件中的```async_get_auth_implementation```方法，可知用户配置所需要的参数就是client_id和client_secret。

而implementations取决于```async_get_implementations```方法，以下给出代码：

```python
async def async_get_implementations(
    hass: HomeAssistant, domain: str
) -> dict[str, AbstractOAuth2Implementation]:
    """Return OAuth2 implementations for specified domain."""
    registered = cast(
        dict[str, AbstractOAuth2Implementation],
        hass.data.setdefault(DATA_IMPLEMENTATIONS, {}).get(domain, {}),
    )

    if DATA_PROVIDERS not in hass.data:
        return registered

    registered = dict(registered)
    for get_impl in list(hass.data[DATA_PROVIDERS].values()):
        for impl in await get_impl(hass, domain):
            registered[impl.domain] = impl

    return registered
```

显而易见，其中```hass.data[DATA_PROVIDERS]```保存了 OAuth2的实现类，**返回结果是用一个名为```registered```的字典保存的**。经过Debug发现，```list(hass.data[DATA_PROVIDERS].values())```对应了前面提及的两种认证模式：本地和云。

上述XBOX和ELECTRIC_KIWI都是云模式的实现，其实现类是：```homeassistant.components.cloud.account_link.CloudOAuth2Implementation```。不过如果提前保存的客户端凭据，这里也会有本地实现的。

非常巧合的是，```async_get_implementations```方法上面一个方法正好就是```async_register_implementation```。通过调用该方法，应该就能实现手动把```AbstractOAuth2Implementation```实现加入到```hass.data[DATA_PROVIDERS]```中。

### 代码实现

回看上述方法调用链可以发现，只要在```async_step_pick_implementation```方法之前调用```async_register_implementation```方法即可，而实现这一点只需要在自定义的```Oauth2FlowHandler```重载```async_step_user```方法，代码如下：

```python
    async def async_step_user(
        self, user_input: dict[str, Any] | None = None
    ) -> FlowResult:
        """Handle a flow start."""
        implementation = get_auth_implementation(self.hass, DOMAIN)
        config_entry_oauth2_flow.async_register_implementation(
            self.hass, DOMAIN, implementation
        )
        return await self.async_step_pick_implementation(user_input)
```

同时由于重启之后不会经过用户配置流程，所以```__init__```文件函数```async_setup_entry```其中获取OAuth2实现的方法失效，也需要手动替换。

原：

```python
async def async_setup_entry(hass: HomeAssistant, entry: ConfigEntry) -> bool:
    """Set up xbox from a config entry."""
    implementation = (
        await config_entry_oauth2_flow.async_get_config_entry_implementation(
            hass, entry
        )
    )
    session = config_entry_oauth2_flow.OAuth2Session(hass, entry, implementation)
    auth = api.AsyncConfigEntryAuth(
        aiohttp_client.async_get_clientsession(hass), session
    )
    ...
```

改成：

```python
async def async_setup_entry(hass: HomeAssistant, entry: ConfigEntry) -> bool:
    """Set up xbox from a config entry."""
    implementation = get_auth_implementation(self.hass, DOMAIN)
    session = config_entry_oauth2_flow.OAuth2Session(hass, entry, implementation)
    auth = api.AsyncConfigEntryAuth(
        aiohttp_client.async_get_clientsession(hass), session
    )
    ...
```

其中```get_auth_implementation```函数定义如下：

```python 
def get_client_credential():
    """Return client credential."""
    return ClientCredential(
        client_id=OAUTH2_CLIENT_ID,
        client_secret=OAUTH2_CLIENT_SECRET,
        name=OAUTH2_CLIENT_NAME,
    )


def get_authorization_server() -> AuthorizationServer:
    """Return authorization server."""
    return AuthorizationServer(
        authorize_url=OAUTH2_AUTHORIZE,
        token_url=OAUTH2_TOKEN,
    )


def get_auth_implementation(
    hass: HomeAssistant, domain: str
) -> config_entry_oauth2_flow.AbstractOAuth2Implementation:
    """Return auth implementation."""
    return MyIntegrationLocalOAuth2Implementation(
        hass,
        domain,
        credential=get_client_credential(),
        authorization_server=get_authorization_server(),
    )
```

# 初始化

脚手架配置还生成了```__init__.py```文件，其执行顺序是每次重启以及Integration用户配置完成之后。该文件有两个作用，：

1. 说明该integration有哪些entity。
2. 往对应的entry里插入必要的信息，以供所有entity使用。

```python
"""The My Integration integration."""
from __future__ import annotations

from homeassistant.config_entries import ConfigEntry
from homeassistant.const import Platform
from homeassistant.core import HomeAssistant
from homeassistant.helpers import aiohttp_client, config_entry_oauth2_flow

from . import api
from .const import DOMAIN

# TODO List the platforms that you want to support.
# For your initial PR, limit it to 1 platform.
PLATFORMS: list[Platform] = [Platform.LIGHT]


async def async_setup_entry(hass: HomeAssistant, entry: ConfigEntry) -> bool:
    """Set up My Integration from a config entry."""
    implementation = (
        await config_entry_oauth2_flow.async_get_config_entry_implementation(
            hass, entry
        )
    )

    session = config_entry_oauth2_flow.OAuth2Session(hass, entry, implementation)

    # If using a requests-based API lib
    hass.data.setdefault(DOMAIN, {})[entry.entry_id] = api.ConfigEntryAuth(
        hass, session
    )

    # If using an aiohttp-based API lib
    hass.data.setdefault(DOMAIN, {})[entry.entry_id] = api.AsyncConfigEntryAuth(
        aiohttp_client.async_get_clientsession(hass), session
    )

    await hass.config_entries.async_forward_entry_setups(entry, PLATFORMS)

    return True


async def async_unload_entry(hass: HomeAssistant, entry: ConfigEntry) -> bool:
    """Unload a config entry."""
    if unload_ok := await hass.config_entries.async_unload_platforms(entry, PLATFORMS):
        hass.data[DOMAIN].pop(entry.entry_id)

    return unload_ok

```

**一个integration有哪几种entity，是定义在该文件中的PLATFORMS变量中**。不过每一个具体的entity定义在自己的代码文件中。例如sensor就定义在sensor.py中。关于entity的概念与使用见后文。

## Config Entry

实际上config entry只是一个全局共享的字典。它的key就是domain，而值通常是可以实现entity功能的api。在有认证框架中的integration中，该api还有一个带token的ClientSession。

基于这样的理解可以这样描述async_setup_entry方法：首先获取本地的 OAuth2实现，然后拿到OAuth2Session，构造API，并存入config_entry中。

# Entity

## 概念：Entity & Device

1. entity是Home Assistant中最小的控制或计量单位。它表示一个设备的某一个指标，或者对设备的某项控制。
2. device是entity的集合， 对应现实中设备与指标或者控制的关系。 

entity 有很多种，几乎囊括了现实设备上的所有指标和控制。具体可见：https://developers.home-assistant.io/docs/core/entity。常用的有sensor、select、switch、button。

sensor用于定时抓取并展示数据，有各种单位制和含义可选；select switch button分别是对设备的控制。

同时，定义了entity，也就定义了device。不同entity定义之间使用了相同的device信息也就实现了entity和device关系的绑定。

## 编写规则

写一个entity的基本思想是：编写类EntityClass继承某个类型的entity，并定义device_info，单位制等等。同时定义方法：async_setup_entry，用于自定义创建entity的规则。

以下是一个async_setup_entry的例子。

```python 
async def async_setup_entry(
    hass: HomeAssistant,
    entry: ConfigEntry,
    async_add_entities: AddEntitiesCallback,
) -> None:
    """Create entities."""
	......
    devices: list[EntityClass] = ...
    async_add_entities(devices)
```

其中ConfigEntry中可以拿到api。

以下是一个EntityClass的例子：

```python
class Inverter(SensorEntity):
    """Class for a sensor."""

    _attr_should_poll = False

    def __init__(
        self,
        manufacturer,
        uid,
        serial,
        version,
        key,
        unit,
        state_class=None,
        device_class=None,
    ):
        """Initialize an inverter sensor."""
        self._attr_unique_id = uid
        self._attr_name = f"{manufacturer} {serial} {key}"
        self._attr_native_unit_of_measurement = unit
        self._attr_state_class = state_class
        self._attr_device_class = device_class
        self._attr_device_info = DeviceInfo(
            identifiers={(DOMAIN, serial)},
            manufacturer=MANUFACTURER,
            name=f"{manufacturer} {serial}",
            sw_version=version,
        )
        self.key = key
        self.value = None

    @property
    def native_value(self):
        """State of this inverter attribute."""
        return self.value
```

## sensor

传感器的概念十分简单，只是定时抓取并展示数据的entity。https://developers.home-assistant.io/docs/core/entity/sensor。

一般而言，每个sensor类都有一个update或者 async_update方法，用于更新数据，不过假如一组sensor的数据来自同一个接口，这样每个sensor单独更新自己的数据自然非常浪费，比较好的做法就是记录下，所有的sensor，然后调用Home Assistant 提供的定时任务去抓取数据。例如：

```python
"""Support for Solax inverter via local API."""
from __future__ import annotations

import asyncio
from datetime import timedelta
import logging

from homeassistant.components.sensor import (
    SensorDeviceClass,
    SensorEntity,
    SensorStateClass,
)
from homeassistant.config_entries import ConfigEntry
from homeassistant.const import UnitOfEnergy
from homeassistant.core import HomeAssistant
from homeassistant.exceptions import PlatformNotReady
from homeassistant.helpers.entity_platform import AddEntitiesCallback
from homeassistant.helpers.event import async_track_time_interval

from .const import DOMAIN, POLL_INTERVAL_SECONDS
from .inverter import InverterEntity
from .solax_cloud import SolaxCloudApi, SolaxCloudError

_LOGGER = logging.getLogger(__name__)
SCAN_INTERVAL = timedelta(seconds=POLL_INTERVAL_SECONDS)


async def async_setup_entry(
    hass: HomeAssistant,
    entry: ConfigEntry,
    async_add_entities: AddEntitiesCallback,
) -> None:
    """Create entities."""
    api: SolaxCloudApi = hass.data[DOMAIN][entry.entry_id]
    dictionary: dict[str, float] = await api.total_yield()
    devices: list[InverterYield] = [
        InverterYield(serial_number, total_yield)
        for serial_number, total_yield in dictionary.items()
    ]
    updater: InverterDataUpdater = InverterDataUpdater(devices, api)
    async_track_time_interval(hass, updater.update, SCAN_INTERVAL)
    async_add_entities(devices)


class InverterYield(SensorEntity, InverterEntity):
    """Class for a sensor."""

    _attr_should_poll = False

    def __init__(self, serial_number: str, value: float) -> None:
        """Initialize an inverter sensor."""
        super().__init__()
        self._attr_unique_id = f"{serial_number}-yield"
        self._attr_name = f"{serial_number}-yield"
        self._attr_native_unit_of_measurement = UnitOfEnergy.KILO_WATT_HOUR
        self._attr_state_class = SensorStateClass.TOTAL
        self._attr_device_class = SensorDeviceClass.ENERGY
        self._attr_device_info = self.generate_device_info(serial_number)
        self._attr_icon = "mdi:solar-power"
        self.serial_number = serial_number
        self.value = value

    @property
    def native_value(self) -> float:
        """State of this inverter attribute."""
        return self.value


class InverterDataUpdater:
    """Update inverter data periodically."""

    def __init__(self, sensors: list[InverterYield], api: SolaxCloudApi) -> None:
        """Initialize."""
        self.sensors = sensors
        self.api = api
        self.ready = asyncio.Event()

    async def update(self, now=None) -> None:
        """Fetch new state data for the sensor.

        This is the only method that should fetch new data for Home Assistant.
        """
        try:
            dct: dict[str, float] = await self.api.total_yield()
            self.ready.set()
        except SolaxCloudError as err:
            if now is not None:
                self.ready.clear()
                return
            raise PlatformNotReady from err
        for sensor in self.sensors:
            if sensor.serial_number in dct:
                sensor.value = dct[sensor.serial_number]
                sensor.async_schedule_update_ha_state()

```

# Python

## Asynchronization

### 关键字：await & async 

python的两个异步的关键字。

async放在方法定义前，表示该方法是异步方法。而调用异步方法时，必须加上await关键字，才能得到方法的返回值。

同步方法及函数里不能出现await关键字。

### 异步网络请求：Aiohttp

Aiohttp 是 python用于HTTP请求的框架，是原生异步的。这一点跟requests有着本质的不同，在本项目中更多使用aiohttp。

基本用法是：生成一个 ClientSession，然后通过session发起请求。返回的结果是ClientResponse。然后可根据结果判断HTTP状态，调用其json()方法即可得到body。注意以上的方法都是异步方法，调用时需要加上await关键字。

## Library

这部分涉及一下python依赖包的开发与发布

## 代码规范

1. 注释
2. 日志
3. 导入排序
4. f-string优先
5. 类型标注

[官方文档上关于代码规范的详细说明](https://developers.home-assistant.io/docs/development_guidelines/)

