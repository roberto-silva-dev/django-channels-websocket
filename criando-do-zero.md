# Django Channels - Projeto de Chat ao Vivo

Este projeto é uma aplicação Django simples que utiliza **Django Channels** para implementar um chat em tempo real via WebSockets.

## Requisitos

- Python 3.8+
- Redis (broker para o Django Channels)
- Daphne (servidor ASGI)

## Instalação e Configuração

### 1. Crie e ative o ambiente virtual (opcional)
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

### 2. Instale as dependências
```bash
pip install django channels channels_redis daphne
```

### 3. Crie o projeto e app
```bash
django-admin startproject core .
django-admin startapp chat
```

### 4. Configure o `settings.py`
Adicione ao `INSTALLED_APPS`:
```python
INSTALLED_APPS = [
    ...
    'channels',
    'chat',
]
```

Adicione as configurações do Channels:
```python
ASGI_APPLICATION = 'core.asgi.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('127.0.0.1', 6379)],
        },
    },
}
```

### 5. Configure o `asgi.py`
```python
import os
import django
from channels.routing import get_default_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "core.settings")
django.setup()
application = get_default_application()
```

### 6. Crie o `routing.py` no projeto
```python
from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
import chat.routing

application = ProtocolTypeRouter({
    "websocket": AuthMiddlewareStack(
        URLRouter(
            chat.routing.websocket_urlpatterns
        )
    ),
})
```

### 7. Crie o `consumers.py` no app `chat`
```python
import json
from channels.generic.websocket import AsyncWebsocketConsumer

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope["url_route"]["kwargs"]["room_name"]
        self.room_group_name = f"chat_{self.room_name}"

        await self.channel_layer.group_add(self.room_group_name, self.channel_name)
        await self.accept()

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(self.room_group_name, self.channel_name)

    async def receive(self, text_data):
        data = json.loads(text_data)
        message = data["message"]
        username = data["username"]

        await self.channel_layer.group_send(
            self.room_group_name,
            {
                "type": "chat_message",
                "message": message,
                "username": username,
            }
        )

    async def chat_message(self, event):
        await self.send(text_data=json.dumps({
            "message": event["message"],
            "username": event["username"],
        }))
```

### 8. Configure `chat/routing.py`
```python
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r"ws/chat/(?P<room_name>\w+)/$", consumers.ChatConsumer.as_asgi()),
]
```

### 9. Crie a view e URL em `views.py` e `urls.py`
```python
# chat/views.py
from django.shortcuts import render

def room(request, room_name):
    return render(request, "chat/room.html", {"room_name": room_name})
```

```python
# chat/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("chat/<str:room_name>/", views.room, name="room"),
]
```

Inclua no `core/urls.py`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('chat.urls')),
]
```

### 10. Template HTML (`chat/templates/chat/room.html`)
```html
<!DOCTYPE html>
<html>
<head>
    <title>Chat</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        #chat-log { border: 1px solid #ccc; height: 300px; overflow-y: scroll; padding: 10px; margin-bottom: 10px; background: #f9f9f9; }
        #chat-controls { display: flex; gap: 10px; }
        input[type="text"] { padding: 8px; font-size: 14px; flex: 1; }
        button { padding: 8px 16px; font-size: 14px; }
    </style>
</head>
<body>
    <h2>Chat Room: {{ room_name }}</h2>
    <div id="chat-log"></div>
    <div id="chat-controls">
        <input id="username" type="text" placeholder="Seu nome">
        <input id="chat-message-input" type="text" placeholder="Digite sua mensagem">
        <button id="chat-message-submit">Enviar</button>
    </div>
    <script>
        const roomName = "{{ room_name }}";
        const chatSocket = new WebSocket("ws://" + window.location.host + "/ws/chat/" + roomName + "/");

        chatSocket.onmessage = function(e) {
            const data = JSON.parse(e.data);
            const log = document.querySelector("#chat-log");
            const message = document.createElement("div");
            message.textContent = `${data.username}: ${data.message}`;
            log.appendChild(message);
            log.scrollTop = log.scrollHeight;
        };

        document.querySelector("#chat-message-submit").onclick = function() {
            const messageInput = document.querySelector("#chat-message-input");
            const usernameInput = document.querySelector("#username");
            const message = messageInput.value;
            const username = usernameInput.value || "Anônimo";
            if (message) {
                chatSocket.send(JSON.stringify({ "message": message, "username": username }));
                messageInput.value = "";
            }
        };
    </script>
</body>
</html>
```

### 11. Executar com Daphne
```bash
daphne core.asgi:application
```

---

Arquivo gerado automaticamente para documentação e testes com Django Channels.  
