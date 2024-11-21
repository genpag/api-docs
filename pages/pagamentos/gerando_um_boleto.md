---
title: Gerando um Boleto
keywords: cobranças, pagamentos, split de pagamentos, boleto
last_updated: July 3, 2024
tags: [cobranças, pagamentos, split, boleto]
summary: "Aprenda a gerar boletos para cobrar seus clientes"
sidebar: mydoc_sidebar
permalink: gerando-um-boleto.html
folder: pagamentos
---
## Introdução

Depois de ter cadastrado um pedido, o próximo passo é registrar um ou mais pagamentos. Neste guia vamos ver como gerar boletos para pagamentos:
* Analisando os dados obrigatórios e opcionais;
* Como enviar a requisição;
* Verificando os dados de retorno para acessar o boleto;
* Como são processados os diferentes estados do pagamento com boleto.

## Dados necessários

| Campo                     | Descrição |
| ------------------------- |:--------------|
| order_id                  | ID do pedido gerado                                                    |
| session_id                | Um identificador da sessão do usuário da sua plataforma                |
| method                    | Método escolhido para pagamento, (use o valor BANK_SLIP)               |
| bank_slip                 | Detalhes do Boelto                                                     |
| bank_slip.expiration_date | Data de vencimento do boleto, precisa ser uma data no futuro.          |
| bank_slip.instructions    | (Opcional) Instruções extras para serem exibidas no boleto             |
| customer_id               | (Opcional) ID de um cliente já cadastrado                              |
| customer                  | (Opcional) Dados de um novo cliente caso ainda não exista um associado |
| external_id               | (Opcional) Identificador deste pagamento no seu sistema                |

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
        "method": "BANK_SLIP",
        "bank_slip": {
            "expiration_date": "2022-04-01",
            "instructions": "Não receber após o vencimento",
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

O retorno vem dentro de um objeto **data**. Para pagamentos com boleto o mais importante neste momento é permitir que o cliente acesse o boleto. O boleto gerado pode ser baixado através das URLs dos campos **bank_slip.print_link_pdf** ou **bank_slip.print_link_html**

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
        "bank_slip": {
            "acquirer_number": "0000001896800",
            "bank": "0033",
            "bank_slip_id": "d8f5112a-e126-4331-ba7b-ea205f7dae40",
            "barcode": "03391890700000840009235515900000000000040101",
            "expiration_date": 2022-02-25,
            "id": "47eca2fc-f6c0-46fb-8845-bcae75ab1236",
            "instructions": "Não receber após o vencimento",
            "issue_date": 2022-02-18,
            "print_link_html": "https://api.getnet.com.br/v1/payments/boleto/620fbba563d8780010c2a8f5/pdf",
            "print_link_pdf": "https://api.getnet.com.br/v1/payments/boleto/620fbba563d8780010c2a8f5/pdf",
            "typeful_line": "03399.23559 15900.000017 89681.401017 0 00000000084000"
        },
        "credit": null,
        "external_id": "id_so_seu_sistema",
        "inserted_at": "2022-03-01T22:22:36.878Z",
        "updated_at": "2002-03-01T22:22:36.878Z"
    }
}
```

## Mudanças de status
Enquanto não for identificado a compensação do boleto o pagamento permanecerá com o status pendente **(PENDING)**. Após a compensação do pagamento o status irá mudar para pago **(PAID)**. Boletos pagos não podem ser cancelados, caso deseje cancelar o boleto você deve cancelar o pagamento antes da sua compensação, caso seja confirmado o cancelamento o status irá mudar para cancelado **(CANCELLED)**.

Todas as mudanças de status ou atualizações sobre um pagamento ou pedido são enviadas para webhooks cadastrados, não deixe de consultar nosso guia de webhooks se quiser receber notificações de atualizações no seu sistema.


{% include links.html %}