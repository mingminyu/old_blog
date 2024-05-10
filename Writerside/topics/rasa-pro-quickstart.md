# 快速入门

<show-structure depth="3"/>

> 对于 Python 3.7 以下的版本，Rasa Pro 对应的库为 rasa-plus；而对于 Python 3.8 以上的版本，Rasa Pro 对应的库就是 rasa-pro。 
> 
{style="warning"}

<format color="Red">注意</format>，Rasa Pro 实际上是收费的版本，但是 Rasa 社区提供了开发版，可以通过个人邮箱进行申请与获取秘钥。

## 1. 安装

如果你能正常访问 europe-west3-python.pkg.dev 的话(国内用户可能需要梯子)，那么按照官方推荐的做法在 pip.conf 文件中先添加 Rasa Pro 的安装源，然后使用 `pip install rasa-pro` 进行安装。

```Ini
[global]
extra-index-url = https://europe-west3-python.pkg.dev/rasa-releases/rasa-plus-py/simple/
```

一些梯子是按照流量进行计费的，直接在 pip.conf 中配置安装源的话，可能导致流量大大超出预期(因为过程中会安装 TensorFlow 这些大空间的库)。但实际上除了 [rasa-pro](https://europe-west3-python.pkg.dev/rasa-releases/rasa-pro-python/rasa-pro/rasa_pro-3.8.5-py3-none-any.whl) 库是从 europe-west3-python.pkg.dev 进行安装的，其他的库都是可以直接从阿里云的 PyPI 源上进行安装，所以我们通过手动下载 rasa-pro 的 whl 文件，然后在不挂梯子的情况下使用阿里云进行安装。

```Bash
pip install ./rasa_pro-3.8.5-py3-none-any.whl
```

过程中 pip 为了解决版本依赖冲突，可能会下载很多库进行尝试，耗时比较长，这里整理了一份针对 rasa-pro==3.8.5 版本的依赖库，可以先安装这些依赖库，再安装 rasa-pro 的 whl 文件。


```Bash
pip install Babel==2.15.0 MarkupSafe==2.1.5 PyJWT==2.8.0 SQLAlchemy==2.0.30 absl-py==2.0.0 aio-pika==8.2.3 aiofiles==23.2.1 aiogram==2.15 aiohttp==3.9.5 aiohttp-retry==2.8.3 aiormq==6.4.2 aiosignal==1.3.1 anyio==3.7.1 apscheduler==3.10.4 argon2-cffi==23.1.0 argon2-cffi-bindings==21.2.0 astunparse==1.6.3 async-timeout==4.0.3 attrs==23.1.0 azure-core==1.30.1 azure-storage-blob==12.16.0 backoff==2.2.1 bidict==0.23.1 blis==0.7.11 boto3==1.34.99 botocore==1.34.99 cachetools==5.3.3 catalogue==2.0.10 cffi==1.16.0 click==8.1.7 cloudpathlib==0.16.0 cloudpickle==3.0.0 colorclass==2.2.2 coloredlogs==15.0.1 colorhash==2.0.0 confection==0.1.4 confluent-kafka==2.3.0 contourpy==1.2.1 cryptography==42.0.7 cvg-python-sdk==0.5.1 cycler==0.12.1 cymem==2.0.8 dask==2022.10.2 dataclasses-json==0.6.5 deprecated==1.2.14 deprecation==2.1.0 dnspython==2.6.1 docopt==0.6.2 environs==9.5.0 faiss-cpu==1.8.0 faker==19.13.0 fbmessenger==6.0.0 filelock==3.14.0 fire==0.6.0 flatbuffers==24.3.25 fonttools==4.51.0 frozenlist==1.4.1 fsspec==2024.3.1 future==1.0.0 gast==0.4.0 google-api-core==2.8.0 google-auth==2.29.0 google-auth-oauthlib==1.0.0 google-cloud-core==2.4.1 google-cloud-storage==2.14.0 google-crc32c==1.5.0 google-pasta==0.2.0 google-resumable-media==2.7.0 googleapis-common-protos==1.56.1 greenlet==3.0.3 grpcio==1.60.0 grpcio-tools==1.60.0 h5py==3.11.0 httptools==0.6.1 humanfriendly==10.0 hvac==1.2.1 importlib-metadata==6.11.0 importlib-resources==6.4.0 isodate==0.6.1 jax==0.4.26 jinja2==3.1.4 jmespath==1.0.1 joblib==1.2.0 jsonpatch==1.33 jsonpickle==3.0.4 jsonpointer==2.4 jsonschema==4.20.0 jsonschema-specifications==2023.12.1 jwcrypto==1.5.6 keras==2.12.0 kiwisolver==1.4.5 langchain==0.0.329 langcodes==3.4.0 langsmith==0.0.92 language-data==1.2.0 libclang==18.1.1 locket==1.0.0 marisa-trie==1.1.1 markdown==3.6 markdown-it-py==3.0.0 marshmallow==3.21.2 matplotlib==3.7.5 mattermostwrapper==2.2 mdurl==0.1.2 minio==7.2.7 ml-dtypes==0.4.0 multidict==5.2.0 murmurhash==1.0.10 mypy-extensions==1.0.0 networkx==3.1 numpy==1.23.5 oauthlib==3.2.2 openai==0.28.1 opentelemetry-api==1.15.0 opentelemetry-exporter-jaeger==1.15.0 opentelemetry-exporter-jaeger-proto-grpc==1.15.0 opentelemetry-exporter-jaeger-thrift==1.15.0 opentelemetry-exporter-otlp==1.15.0 opentelemetry-exporter-otlp-proto-grpc==1.15.0 opentelemetry-exporter-otlp-proto-http==1.15.0 opentelemetry-proto==1.15.0 opentelemetry-sdk==1.15.0 opentelemetry-semantic-conventions==0.36b0 opt-einsum==3.3.0 packaging==20.9 pamqp==3.2.1 pandas==2.2.2 partd==1.4.2 pep440-version-utils==0.3.0 phonenumbers==8.13.36 pillow==10.3.0 pluggy==1.5.0 ply==3.11 preshed==3.0.9 presidio-analyzer==2.2.33 presidio-anonymizer==2.2.33 prompt-toolkit==3.0.28 protobuf==4.23.3 psutil==5.9.8 psycopg2-binary==2.9.9 pyarrow==16.0.0 pyasn1==0.6.0 pyasn1-modules==0.4.0 pycountry==22.3.5 pycparser==2.22 pycryptodome==3.20.0 pydot==1.4.2 pygments==2.18.0 pyhcl==0.4.5 pykwalify==1.8.0 pymilvus==2.4.1 pymongo==4.6.3 pyparsing==3.1.2 pypred==0.4.0 python-crfsuite==0.9.10 python-dateutil==2.8.2 python-dotenv==1.0.1 python-engineio==4.9.0 python-keycloak==3.12.0 python-socketio==5.11.2 pytz==2022.7.1 pyyaml==6.0.1 questionary==1.10.0 randomname==0.2.1 rasa-pro==3.8.5 rasa-sdk==3.8.0 redis==5.0.4 referencing==0.35.1 regex==2022.10.31 requests-file==2.0.0 requests-oauthlib==2.0.0 requests-toolbelt==1.0.0 rich==13.7.1 rocketchat_API==1.30.0 rpds-py==0.18.1 rsa==4.9 ruamel.yaml==0.17.21 ruamel.yaml.clib==0.2.8 s3transfer==0.10.1 sanic==21.12.2 sanic-cors==2.0.1 sanic-jwt==1.8.0 sanic-routing==0.7.2 scikit-learn==1.1.3 scipy==1.10.1 sentry-sdk==1.14.0 simple-websocket==1.0.0 six==1.16.0 sklearn-crfsuite==0.3.6 slack-sdk==3.27.1 smart-open==6.4.0 spacy==3.7.4 spacy-legacy==3.0.12 spacy-loggers==1.0.5 srsly==2.4.8 structlog==23.1.0 structlog-sentry==2.1.0 tabulate==0.9.0 tarsafe==0.0.5 tenacity==8.2.3 tensorboard==2.12.3 tensorboard-data-server==0.7.2 tensorflow==2.12.0 tensorflow-estimator==2.12.0 tensorflow-io-gcs-filesystem==0.32.0 tensorflow-text==2.12.0 tensorflow_hub==0.13.0 termcolor==2.4.0 terminaltables==3.1.10 thinc==8.2.3 threadpoolctl==3.5.0 thrift==0.20.0 tiktoken==0.4.0 tldextract==5.1.2 toolz==0.12.1 tqdm==4.66.4 twilio==8.4.0 typer==0.9.4 typing-inspect==0.9.0 typing-utils==0.1.0 tzdata==2024.1 tzlocal==5.2 ujson==5.9.0 urllib3==1.26.18 uvloop==0.19.0 wasabi==1.1.2 wcwidth==0.2.13 weasel==0.3.4 webexteamssdk==1.6.1 websockets==10.4 werkzeug==3.0.3 wrapt==1.14.1 wsproto==1.2.0 yarl==1.9.4 zipp==3.18.1
```

## 2. CALM


## 3. 第一个示例应用

接下来，我们将搭建一个通过机器人进行转账的示例应用，这个机器人应该具备高鲁棒性，能够









<seealso>
<category ref="ref_docs">
    <a href="https://rasa.com/docs/rasa-pro/installation/python/installation">Installation for local development</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>
