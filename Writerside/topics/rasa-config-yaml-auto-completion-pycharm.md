# 配置 YAML 文件自动补全

<show-structure depth="3"/>

使用过 Rasa Open Source 框架的同学应该都很清楚，我们需要在多个 YAML 文件中配置训练数据、规则以及故事等，但是 PyCharm 并不能像 IDEA 构建 Spring Boot 工程时那样能够提供参数自动补全的功能，万幸的是我们可以在 PyCharm 中可以自定义一些模板，从而达到预期的效果。

## 1. 定义 Json Schema

Rasa 项目中 YAML 文件有点多，主要包括 config.yml、domain.yml、endpoints.yml、credentials.yml、rules.yml 以及 stories.yml 文件，针对每个文件我们需要定义一个 JSON Schema，确保打开对应文件时能达到自动补全的效果。

### 1.1 config.yml

关于 config.yml 文件对应的 JSON Schema 文件可以访问 [config.json](https://github.com/mingminyu/python-config-file-auto-complete/rasa-open-source/config.json)。


### 1.2 domain.yml

关于 domain.yml 文件对应的 JSON Schema 文件可以访问 [domain.json](https://github.com/mingminyu/python-config-file-auto-complete/rasa-open-source/domain.json)。

### 1.3 endpoints.yml

关于 endpoints.yml 文件对应的 JSON Schema 文件可以访问 [endpoints.json](https://github.com/mingminyu/python-config-file-auto-complete/rasa-open-source/endpoints.json)。

### 1.4 credentials.yml

关于 credentials.yml 文件对应的 JSON Schema 文件可以访问 [credentials.json](https://github.com/mingminyu/python-config-file-auto-complete/rasa-open-source/credentials.json)。

### 1.5 nlu.yml

关于 nlu.yml 文件对应的 JSON Schema 文件可以访问 [nlu.json](https://github.com/mingminyu/python-config-file-auto-complete/rasa-open-source/nlu.json)。

### 1.6 rules.yml

关于 rules.yml 文件对应的 JSON Schema 文件可以访问 [rules.json](https://github.com/mingminyu/python-config-file-auto-complete/rasa-open-source/rules.json)。

### 1.7 stories.yml

关于 stories.yml 文件对应的 JSON Schema 文件可以访问 [stories.json](https://github.com/mingminyu/python-config-file-auto-complete/rasa-open-source/stories.json)。
