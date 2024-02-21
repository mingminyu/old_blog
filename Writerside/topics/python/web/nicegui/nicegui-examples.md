# 应用示例

<show-structure depth="2"/>

## 1. Chat-App

NiceGUI 提供了一个简单的 [ChatApp 的示例](https://github.com/zauberzeug/nicegui/tree/main/examples/chat_app)，通过 `ui.chat_message` 组件可以达到 ChatGPT 应用的对话效果。

### 1.1 源码分析

以下具体应用的源码，通过 `ui.chat_message` 组件来构建出一个对话界面。

```Python
from uuid import uuid4
from datetime import datetime
from typing import List, Tuple
from nicegui import Client, ui

messages: List[Tuple[str, str, str, str]] = []


@ui.refreshable
def chat_messages(own_id: str) -> None:
    for user_id, avatar, text, stamp in messages:
        ui.chat_message(
            text=text, 
            stamp=stamp, 
            avatar=avatar, 
            sent=own_id == user_id
            )
    
    ui.run_javascript('window.scrollTo(0, document.body.scrollHeight)')


@ui.page('/')
async def main(client: Client):
    def send() -> None:
        stamp = datetime.utcnow().strftime('%X')
        messages.append((user_id, avatar, text.value, stamp))
        text.value = ''
        chat_messages.refresh()

    user_id = str(uuid4())
    avatar = f'https://robohash.org/{user_id}?bgset=bg2'

    anchor_style = r'a:link, a:visited {color: inherit !important; text-decoration: none; font-weight: 500}'
    ui.add_head_html(f'<style>{anchor_style}</style>')
    with ui.footer().classes('bg-white'), ui.column().classes('w-full max-w-3xl mx-auto my-6'):
        with ui.row().classes('w-full no-wrap items-center'):
            with ui.avatar().on('click', lambda: ui.open(main)):
                ui.image(avatar)
            
            text = ui.input(placeholder='message').on('keydown.enter', send) \
                .props('rounded outlined input-class=mx-3').classes('flex-grow')
        
        ui.markdown('simple chat app built with [NiceGUI](https://nicegui.io)') \
            .classes('text-xs self-end mr-8 m-[-1em] text-primary')
    
    # chat_messages(...) uses run_javascript which is only possible after connecting
    await client.connected()
    
    with ui.column().classes('w-full max-w-2xl mx-auto items-stretch'):
        chat_messages(user_id)

ui.run()
```
{ collapsible="true" default-state="collapsed" collapsed-title="NiceGUI ChatApp 示例代码"}


> 值得注意的是，`ui.chat_message` 组件库只能输出纯文本，无法显示 Markdown 形式的内容。
{style="warning"}

### 1.2 界面预览

![ChatApp](%myimgs%/nicegui-chatapp.png?raw=true)

## 2. 表格插槽案例

> [表格插槽案例](https://www.bilibili.com/video/BV1Gx4y1Z79V) 来自 B 站 《数据大宇宙》的发布视频，这里研究一下代码。

### 2.1 源码

```Python
import ng_init
import pandas as pd
from nicegui import ui
from itertools import cycle

ng_init.styles()

def create_chip_color(df: pd.DataFrame) -> None:
    """给不同类别 chip 添加不同颜色"""
    _chip_colors = cycle()
    color_map = {
        name: color
        for name, color in zip(df["产品类别"].unique(), _chip_colors)
    }
    return df.assign(_chip_color=lambda x: df["产品类别"].map(color_map))


df = pd.read_excel("销售数据.xlsx")
df = create_chip_color(df)
table = ui.table.from_pandas(df, pagination=10)
table._props["visible-columns"] = "_chip_color"  # 设置 _chip_color 列不展示


table.add_slot(
    "body-cell-产品类型",
    r'''
        <q-td :props="props">
            <q-chip clickable text-color="white" icon="bookmark"
                :color="props.row['_chip_color']"
                @click="$parent.$emit('clickChip', props.row)"
                >
                {{ props.value }}
            </q-chip>
        </q-td>
    '''
)

def click_chip(e):
    ui.notify(e)


region_options = df["区域"].unique()
table.add_slot(
    "body-cell-区域",
    f'''
        <q-td :props="props">
            <q-select outlined v-model="props.row[props.col.field]"
                :options={str(region_options)}
                @update:model-value="() => $parent.$emit('zoneSelect', props.row, props.rowIndex)"
            />
        </q-td>
    '''
)

def region_select(e):
    ui.notify(f"选择区域为: {e}")
    

def on_knob_value_change(e):
    show_knob = e.value
    
    if show_knob:
        table.add_slot(
            "body-cell-预计销售成本",
            r'''
                <q-td :props="props">
                    <q-knob readonly show-value 
                        size="60px" color="primary" track-color="grey-3"
                        class="text-primary q-ma-md"
                        v-model="props.value"
                        :thickness="0.22"
                    ><q-knob>
                </q-td>
            '''
        )
    else:
        table.slots.pop("body-cell-预计销售成本")
    
    table.update()
    

table.on("zoneSelect", zone_select)
table.on("clickChip", click_chip)
ui.checkbox("图标数值", value=False, on_change=on_knob_value_change)
```




<seealso>
<category ref="ref_github">
<a href="https://github.com/zauberzeug/nicegui/tree/main/examples/chat_app">ChatApp</a>
</category>
</seealso>



