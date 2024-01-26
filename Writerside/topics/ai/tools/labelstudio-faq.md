# 常见问题

<show-structure for="chapter,procedure" depth="2"/>

## 1. 跨域问题

Label Studio 不建议加载本地文件，通常情况下我们是通过给本地文件开启一个 HTTP 服务，然后在模板里面配置变量的值为具体 url。

### 1.1 图像文件

如果你加载的是图像或者文本文件的话，那么只需要开启 Python 内置的 HTTP 服务:

```Bash
python -m http.server 8000
```

然后我们在模板中配置一个变量 `image`，指定其值为 `http://127.0.0.1:8000/images/test.png` 就好。

### 1.2 音频文件

但是如果你需要获取的是音频文件的话，那么使用内置的 HTTP 服务就不可行，这里需要通过 WEB 框架构建一个简单的 HTTP GET 接口，这里我们选择 FastAPI 来构建对应的接口:

```Python
from fastapi import FastAPI
from fastapi.responses import FileResponse
from fastapi.middleware.cors import CORSMiddleware


app = FastAPI()
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/recordings/{wav_path}")
async def get_wav_file(wav_path: str):
    # 这里 audio_path 须为绝对路径
    audio_path = f"/Users/yumingmin/llm_labelstudio/data/recordings/{wav_path}"
    return FileResponse(file_path, media_type="audio/wav")


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

使用如下命令开启 FastAPI 服务:

```Bash
python server.py
```

接下来我们只需要在模板中配置一下 `audio` 变量的值为 `http://127.0.0.1:8000/audio/test.wav` 就可以了。

### 1.3 反向代理

还有一种特殊的情况，Label Studio 服务是架设在 NGINX 反向代理上的，比如本地 Label Studio 服务为 127.0.0.1:8080（实际物理 IP 为 1.2.3.4），对应反向代理域名为 label-studio.ai.com。

我们需要首先理解的是，如果按照上面的情况去配置是不行的，因为配置好 `audio` 的值后，它不是服务器直接对本地发起请求，而是先由浏览器向目标服务器发起请求，但是浏览器所访问的 8080 的服务实际上 NGINX 通过域名转发过来的，本身我们自己的浏览器是不能直接访问 `1.2.3.4:8080` 的服务，由于网络不通的原因，是不可能拿到具体音频的文件流数据的。


### 1.4 静态文件

因为 Label Studio Web 服务自己本身就可以访问一下示例的静态文件，如果我们不希望另加接口的话，那么可以直接将自己的数据文件放在 Label Studio Web 服务的静态文件夹下。

在 Label Studio 安装包路径下新建 core/recordings 文件夹，将所有音频放进去，那么我们就可以在模板中直接使用 `http://127.0.0.1:8000/static/recordings/test.wav` 来访问（使用 1.2.3.4 或者 localhost 都可以）。

```Text
PYTHONENV/lib/python3.10/site-packages/label_studio/core/static_build
```

> 当然浏览器与服务网络不同的情况下，本身服务是通过 NGINX 反向代理开启的，那么需要使用**域名**代替 `IP:端口`，比如使用 `http://label-studio.ai.com/static/recordings/test.wav` 来获取音频文件。
