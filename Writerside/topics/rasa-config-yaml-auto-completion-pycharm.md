# 配置 YAML 文件自动补全

<show-structure depth="3"/>

使用过 Rasa Open Source 框架的同学应该都很清楚，我们需要在多个 YAML 文件中配置训练数据、规则以及故事等，但是 PyCharm 并不能像 IDEA 构建 Spring Boot 工程时那样能够提供参数自动补全的功能，万幸的是我们可以在 PyCharm 中可以自定义一些模板，从而达到预期的效果。

## 1. 定义 Json Schema

### 1.1 config.yml

### 1.2 domain.yml

### 1.3 endpoints.yml

### 1.4 credentials.yml

### 1.5 nlu.yml

### 1.6 rules.yml

### 1.7 stories.yml

我们先看下 Rasa 中的 story 大概的数据结构，再来定义一个对应的 JSON Schema 文件。

```yaml
version: "3.1"

stories:
- story:
  steps:
  - intent: intent_1
  - action: utter_1
```
