# Django Channels Chat - Instruções Rápidas

Este projeto é uma aplicação de **chat em tempo real** feita com **Django** e **Django Channels**, utilizando WebSockets. Ideal para entender a base de comunicação em tempo real com Django.

## Clonando o Repositório

```bash
git clone https://github.com/roberto-silva-dev/django-channels-websocket.git
```

## Criando Ambiente Virtual (opcional)

```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

## Instalando as Dependências

```bash
pip install -r requirements.txt
```

> Certifique-se de ter o Redis rodando localmente na porta padrão (6379).

## Rodando o Servidor

```bash
daphne -b 127.0.0.1 -p 8000 core.asgi:application
#OU
python -m daphne -b 127.0.0.1 -p 8000 core.asgi:application
#OU
daphne core.asgi:application
#OU
python -m daphne core.asgi:application
```

## Testando o Chat

Abra no navegador:
```
http://127.0.0.1:8000/chat/sala1/
```

Abra outra aba com a mesma URL e envie mensagens entre as abas para simular uma conversa ao vivo.

---

Feito com Django Channels + WebSockets.
