# 领域

<show-structure depth="3"/>

<show-structure depth="3"/>

领域定义了对话机器人的全部操作，它指定对话机器人的**意图、实体、槽、响应、表单和动作**，同时它还定义了对话会话的配置。


如下是一个取自 [concertbot](https://github.com/RasaHQ/rasa/tree/main/examples/concertbot) 完整的领域实例：

```yaml
version: "3.1"

intents:
  - affirm
  - deny
  - greet
  - thankyou
  - goodbye
  - search_concerts
  - search_venues
  - compare_reviews
  - bot_challenge
  - nlu_fallback
  - how_to_get_started

entities:
  - name

slots:
  concerts:
    type: list
    influence_conversation: false
    mappings:
    - type: custom
  venues:
    type: list
    influence_conversation: false
    mappings:
    - type: custom
  likes_music:
    type: bool
    influence_conversation: true
    mappings:
    - type: custom

responses:
  utter_greet:
    - text: "Hey there!"
  utter_goodbye:
    - text: "Goodbye :("
  utter_default:
    - text: "Sorry, I didn't get that, can you rephrase?"
  utter_youarewelcome:
    - text: "You're very welcome."
  utter_iamabot:
    - text: "I am a bot, powered by Rasa."
  utter_get_started:
    - text: "I can help you find concerts and venues. Do you like music?"
  utter_awesome:
    - text: "Awesome! You can ask me things like \"Find me some concerts\" or \"What's a good venue\""

actions:
  - action_search_concerts
  - action_search_venues
  - action_show_concert_reviews
  - action_show_venue_reviews
  - action_set_music_preference

session_config:
  session_expiration_time: 60  # value in minutes
  carry_over_slots_to_new_session: true
```
{collapsible="true" default-state="expanded"}


## 1. 多个领域文件

领域可以定义为单个 YAML 文件，**也可以拆分为目录中的多个文**件。当拆分为多个文件时，领域内容可被读取并自动合并在一起。你可以通过运行如下命令在命令行界面中训练具有拆分领域文件的模型：

```Bash
rasa train --domain path_to_domain_directory
```

## 2. 意图

领域文件中的 `intents` 键列出了 NLU 数据和对话训练数据中使用的所有意图。

### 2.1 忽略某些意图的实体

要忽略某些意图的所有实体，你可以将 `use_entities: []` 参数添加到领域文件中的意图，如下所示：

```yaml
intents:
- greet:
  use_entities: []
```

要忽略某些实体或显式地仅考虑某些实体，你可以使用如下语法：

```yaml
intents:
- greet:
    use_entities:
      - name
      - first_name
- farewell:
    ignore_entities:
      - location
      - age
      - last_name
```

对于任何单一意图，只能是 `use_entities` 或 `ignore_entities`。 这些意图的**被排除实体将不被特征化**，因此不会影响下一个动作预测。当你有一个不关心被提取的实体的意图时，这很有用。如果你在**没有** `use_entities` 或 `ignore_entities` 参数的情况下列出意图，实体将被正常特征化。当然，也可以通过将实体本身的 `influence_conversation` 标识设置为 `false` 来忽略所有意图的实体。

> 如果你希望这些实体不通过槽影响动作预测，请为具有相同名称的槽设置 `influence_conversation: false`。
>
{style="note"}

