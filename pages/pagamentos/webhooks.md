---
title: Webhooks e Eventos
keywords: cobranças, pagamentos, webhooks, eventos
last_updated: July 3, 2016
tags: [cobranças, pagamentos, webhooks, eventos]
summary: "Veja como ouvir eventos de pagamentos para atualizar seu sistema."
sidebar: mydoc_sidebar
permalink: webhooks-e-eventos.html
folder: pagamentos
---

## Introdução
Neste guia você vai ver como cadastrar um webhook para receber notificações no seu sistema.
Você pode escolher os eventos dos quais deseja ser notificado, como mudanças de status em pedidos e pagamentos, e ao ocorrerem, enviamos uma requisição para uma URL cadastrada por você.

Neste guia você verá:
* Cadastrando um webhook;
* Conferindo os eventos ocorridos;
* Realizando re-tentativas de notificações.

## Cadastrando um webhook
Acesse sua conta em ambiente de [sandbox](https://business-dev.gen.com.br) ou de [produção](https://business.gen.com.br), vá em no seu perfil e clique em **Perfil e preferências > Notificações > + Criar Webhook** e informe os seguintes dados:
* **URL**: A url do seu sistema para onde devemos enviar a notificação. Será feita uma requisição POST neste endereço;
* **Autorização**: Um token de autenticação. Enviaremos esse token no cabeçalho **Authorization** e você pode usar esse token para validar quem está enviando a notificação;
* **Eventos**: A lista dos eventos que você deseja receber.

### Cadastrando pela API
Para cadastrar um novo webhook pela API, realize uma chamada POST no endpoint de webhooks no formato abaixo, utilize a chave de acesso da sua conta:

[Referência da API: Webhooks](https://docs.gen.com.br/#1072ee00-9e83-4460-bf9f-6bc3ee52e1d6)
```bash
curl -X GET 'https://api-sandbox.genpag.com.br/api/webhooks' \
        -H 'Content-Type: application/json' \
        -H 'Authorization: Bearer <access_token>'
        --data '{
          "webhook": {
              "events": ["PAYMENT.*","ORDER.*"],
              "target": "https://meusistema.com.br/webhooks",
              "authorization": "umachavequalquer"
           }
        }'
```

Na lista de eventos você pode cadastrar um evento específico PAYMENT.CREATED ou todos os eventos de uma categoria o padrão PAYMENT.*

## Conferindo os eventos ocorridos
Para listar os eventos ocorridos, vá em **Perfil e preferências > Notificações** na aba Eventos. Você vai ver a lista de eventos já ocorridos.

### Consultando pela API
Você também pode consultar os eventos já ocorridos realizando uma chamada GET na API de eventos utilizando a chave de acesso da sua conta:

[Referência da API: Eventos](https://docs.gen.com.br/#d0f1e393-0063-4f7f-9593-2b2f4cc53229)
```bash
curl -X GET 'https://api-sandbox.genpag.com.br/api/events' \
        -H 'Content-Type: application/json' \
        -H 'Authorization: Bearer <access_token>'
```

## Realizando re-tentativas de notificações

Ao clicar em um evento na lista você pode visualizar o histórico de notificações onde é possível conferir o status da resposta. Quando recebemos uma resposta 2XX do seu sistema confirmamos a entrega na notificação, qualquer resposta diferente tratamos como erro e realizamos até 20 tentativas de entrega.

Você também pode re-enviar uma notificação manualmente clicando no ícone que fica à direita do status:


### Consultando pela API

Para consultar as notificações enviadas utilizando nossa API, realize uma chamada GET na API de notificações utilizando a chave de acesso da sua conta:

[Referência da API: Notificações](https://docs.gen.com.br/#f77acab1-bd7b-457c-b729-44889f84c627)
```bash
curl -X GET 'https://api-sandbox.genpag.com.br/api/notifications' \
        -H 'Content-Type: application/json' \
        -H 'Authorization: Bearer <access_token>'
```



