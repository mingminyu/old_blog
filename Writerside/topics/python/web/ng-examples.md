# 应用示例

<show-structure depth="2"/>

## 1. Chat-App

NiceGUI 提供了一个简单的 [ChatApp 的示例](https://github.com/zauberzeug/nicegui/tree/main/examples/chat_app)，通过 `ui.chat_message` 组件可以达到 ChatGPT 应用的对话效果。

### 1.1 源码分析

以下具体应用的源码，值得注意的是，`ui.chat_message` 组件库只能输出纯文本，无法显示 Markdown 形式的内容:

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

### 1.2 界面预览

![ChatApp](%myimgs%/nicegui-chatapp.png?raw=true)



<seealso>
<category ref="ref_github">
<a href="https://github.com/zauberzeug/nicegui/tree/main/examples/chat_app">ChatApp</a>
</category>
</seealso>



