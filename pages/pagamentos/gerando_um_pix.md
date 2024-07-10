---
title: Gerando um PIX
keywords: cobranças, pagamentos, split de pagamentos, pix
last_updated: July 3, 2016
tags: [cobranças, pagamentos, split, pix]
summary: "Aprenda a gerar um PIX de cobrança através da API da Genpag"
sidebar: mydoc_sidebar
permalink: gerando-um-pix.html
folder: pagamentos
---
## Introdução

Depois de ter cadastrado um pedido, o próximo passo é registrar um ou mais pagamentos. Neste guia vamos ver como gerar PIX de cobrança:
* Analisando os dados obrigatórios e opcionais;
* Como enviar a requisição;
* Verificando os dados de retorno para acessar o PIX copia e cola;
* Como são processados os diferentes estados do pagamento com PIX.

## Dados necessários

| Campo                | Descrição |
| -------------------- |:--------------|
| order_id             | ID do pedido gerado                                                    |
| session_id           | Um identificador da sessão do usuário da sua plataforma                |
| method               | Método escolhido para pagamento. (Use o valor PIX)                     |
| pix                  | Detalhes do PIX                                                        |
| pix.expiration       | Data e hora do vencimento do PIX, precisa ser uma data no futuro.      |
| customer_id          | (Opcional) ID de um cliente já cadastrado                              |
| customer             | (Opcional) Dados de um novo cliente caso ainda não exista um associado |
| external_id          | (Opcional) Identificador deste pagamento no seu sistema                |

Os dados do cliente (customer) são opcionais se o customer_id estiver presente ou se já houver um cliente associado ao pedido.
No id externo (external_id) você pode informar um id que identifica este pagamento no seu sistema para localizá-lo novamente no retorno dos dados ou em notificações de eventos.

{% include warning.html content="A informação de sessão (session_id) é utilizada por nossos sistemas antifraude. Para ambiente sandbox utilize um valor qualquer, para ambiente de produção ele deve ser gerado seguindo as instruções do guia Capturando dados da Sessão." %}

## Requisição

Envie os dados em uma requisição POST para o endpoint de pagamentos conforme o formato abaixo, os dados opcionais não precisam ser enviados:

[Referência da API: Pagamentos](https://docs.gen.com.br/#afd46332-94bb-45f1-8e99-be8e9fe19b41)
```bash
curl -X POST "https://api-sandbox.genpag.com.br/api/sellers/:sellerId/payments" \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer (access token)' \
--data '{
    "payment": {
        "session_id": "123",
        "order_id": "6c48f2e3-722a-8198-fffc-73b458502665",
        "method": "PIX",
        "pix": {
            "expiration": "2022-04-01T00:05:00"
        },
        "customer_id": "e035c42f-1ad9-4edd-db73-ebc0a2b227cc",
        "customer": {
           "name": "Jhon Trevor",
           "email": "customer@email.com",
           "cpf": "01234567891",
           "phone": "+5562992929292",
           "birth_date": "1965-08-10",
           "cnpj": "12345678000166",
           "address": {
              "line1": "Rua tal",
              "line2": "S/N",
              "line3": "QD 2, LT 1, Ed. Sala 100"
              "neighborhood": "Centro",
              "city": "São Paulo",
              "state": "SP",
              "country_code": "BR",
              "postal_code": "01153-000"
          }
        },
        "external_id": "id_do_seu_sistema"
    }
}'
```
## Retorno

O retorno vem dentro de um objeto **data**. Para pagamentos por PIX é importante permitir que o cliente acesse o código do PIX no campo **pix.brcode**.

Você pode disponibilizar via copia e cola ou construir um QRCode baseado neste código, ele será acessível por qualquer instituição financeira.

```json
{
    "data": {
        "id": "12ac0bcb-397b-776d-a621-fce33ec302e9",
        "session_id": "123",
        "order_id": "6c48f2e3-722a-8198-fffc-73b458502665",
        "method": "PIX",
        "seller_id": "60b2eac6-bf01-58ee-f488-67d43c6d2d0f",
        "marketplace_id": "7a65dc09-e1ec-93f2-fb01-9acb4e9f1fa2",
        "customer_id": "e035c42f-1ad9-4edd-db73-ebc0a2b227cc",
        "subscription_id": null,
        "subtotal_cents": 10000,
        "discount_cents": 0,
        "addition_cents": 0,
        "shipping_cents": 0,
        "total_cents": 10000,
        "status": "PENDING",
        "device": {},
        "pix": {
           "brcode": "00020101021226910014br.gov.bcb.pix2569api.developer.btgpactual.com/v1/p/v2/bf1652570dd04572a94f2abd785bd4ab5204000053039865802BR5925GABRIEL SANTORI6014Belo Horizonte61083032005062070503***6304AE14",
           "expiration": "2022-03-02T23:59:59Z"
        },
        "credit": null,
        "external_id": "id_so_seu_sistema",
        "inserted_at": "2022-03-01T22:22:36.878Z",
        "updated_at": "2002-03-01T22:22:36.878Z"
    }
}
```

## Revertendo um PIX
Após a confirmação do PIX e mudança de status do pagamento para pago (PAID) é possível realizar uma devolução total ou parcial do valor recebido.

Você pode realizar a devolução através da chamada a seguir:
```bash
curl -X POST "https://api-sandbox.genpag.com.br/api/sellers/:sellerId/pix/refund/:paymentId" \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer (access token)' \
--data '{
   "amount": 2290
}'
```

O valor informado no campo amount deve estar entre 0 e o valor total do pagamento. Você irá utilizar na URL os ids da sua conta e do pagamento PIX gerado no retorno da última requisição.


## Mudanças de status
Enquanto não for identificado o envio do PIX, o pagamento permanecerá com o status pendente **(PENDING)**. Após a identificação do pagamento o status irá mudar para pago **(PAID)**. No caso do PIX o pagamento pode ser cancelado **(CANCELLED)** antes da identificação do pagamento ou devolvido **(REVERSED)** até a data limite estabelecida pelas regras do Banco Central do Brasil.

Todas as mudanças de status ou atualizações sobre um pagamento ou pedido são enviadas para webhooks cadastrados, não deixe de consultar nosso guia de webhooks se quiser receber notificações de atualizações no seu sistema.




{% include links.html %}