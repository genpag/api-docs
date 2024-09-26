---
title: Cobrando no Crédito
keywords: cobranças, pagamentos, split de pagamentos, crédito
last_updated: July 3, 2024
tags: [cobranças, pagamentos, split, crédito]
summary: Neste guia você vai aprender a realizar cobranças no cartão de crédito.
sidebar: mydoc_sidebar
permalink: cobrando-no-credito.html
folder: pagamentos
---
## Introdução

Neste guia vamos ver como cobrar um cartão de crédito:
* Tokenizando os dados do cartão;
* Analisando os dados obrigatórios e opcionais;
* Como enviar a requisição;
* Dados de retorno;
* Mudanças de status;
* Captura de Pagamentos.

## Tokenizando os dados do cartão

Antes de registrar um pagamento com cartão de crédito é necessário tokenizar o número do cartão. Envie a requisição de tokenização diretamente do dispositivo do usuário, no navegador, app no celular ou computador para manter a conformidade e segurança dos dados.

Para essa chamada, no cabeçalho Authorization, informe o token de cliente **(client token)** ao invés do access token. Você receberá um token do cartão para utilizar no lugar do número do cartão nas chamadas seguintes.

[Referência da API: Tokenizar Cartão](https://docs.gen.com.br/#6f4845d0-17ce-4071-b904-6a16b6ad1486)
```bash
curl -X POST "https://api-sandbox.genpag.com.br/api/tokenize" \
-H 'Content-Type: application/json' \
-H 'Authorization: (client token)' \
--data '{
    "card_number": "4444333322221111"
}'
```

### Resposta
```bash
{
  "data": {
    "number_token": "92069efd-b8ef-1a28-f15e-113646215dc6"
  }
}
```

## Analisando os dados obrigatórios e opcionais

| Campo                               | Descrição                                                                        |
| ----------------------------------- |:---------------------------------------------------------------------------------|
| order_id                            | ID do pedido gerado                                                              |
| session_id                          | Um identificador da sessão do usuário da sua plataforma                          |
| method                              | Método escolhido para pagamento. (Use o valor PIX)                               |
| credit                              | Detalhes da cobrança no crédito                                                  |
| credit.installments                 | Número de parcelas, padrão 1                                                     |
| credit.delay_capture                | Utilizar captura tardia, padrão false.                                           |
| credit.save_card                    | Salvar dados do cartão para outras compras.                                      |
| credit.credit_card_id               | ID de um cartão salvo                                                            |
| credit.credit_card                  | Dados de um novo cartão (Opcional se credit_card_id estiver presente)            |
| credit.credit_card.token            | Token do número do cartão                                                        |
| credit.credit_card.expiration_month | Mês de expiração do cartão (ex: 12)                                              |
| credit.credit_card.expiration_year  | Ano de expiração do cartão (ex: 22)                                              |
| customer_id                         | (Opcional) ID de um cliente já cadastrado                                        |
| customer                            | (Opcional) Dados de um novo cliente caso ainda não exista um associado ao pedido |
| external_id                         | (Opcional) Identificador deste pagamento no seu sistema                          |

Os dados do cliente **(customer)** são opcionais se o **customer_id** estiver presente ou se já houver um cliente associado ao pedido.
No id externo **(external_id)** você pode informar um id que identifica este pagamento no seu sistema para localizá-lo novamente no retorno dos dados ou em notificações de eventos.

O campo **delay_capture** é utilizado para gerar uma pré-autorização no cartão de crédito do cliente, se você enviar true deverá realizar a captura da transação posteriormente ou ela será automaticamente estornada. A pré-autorização já garante que o cartão tem limites, é válido e já realiza a reserva de valor, portanto você pode utilizar para fluxos de pagamentos em que seja necessário aguardar alguma confirmação antes de capturar a cobrança.

O campo **save_card** serve para marcar se o cliente optou por salvar o cartão para uso futuro, neste caso ele irá gerar um cartão associado ao cadastro do cliente que pode ser utilizado em compras futuras ou assinaturas. No campo **credit_card_id** você pode informar o id de um cartão de crédito salvo pelo cliente **(customer)** ou informar o token de um cartão novo no objeto **credit_card**.


{% include warning.html content="A informação de sessão **(session_id)** é utilizada por nossos sistemas antifraude. Para ambiente sandbox utilize um valor qualquer, para ambiente de produção ele deve ser gerado seguindo as instruções do guia Capturando dados da Sessão." %}

### Cartões de teste

Para realizar chamadas de teste, utilize um dos cartões abaixo utilizando como data de expiração qualquer data no futuro e como código de segurança qualquer valor de 3 dígitos.


| Campo             | Tipo de Teste        | Resultado do Teste |
| ----------------- |:---------------------|:---- |
| 5155901222280001  | Transação Autorizada | Transação Aprovada |
| 4012001037141112  | Transação Autorizada | Transação Aprovada |
| 5155901222270002  | Transação Não Autorizada | Cartão Inválido |
| 5155901222260003  | Transação Não Autorizada | Cartão Vencido |
| 5155901222250004  | Transação Não Autorizada | Estabelecimento Inválido |
| 5155901222240005  | Transação Não Autorizada | Saldo Insuficiente |
| 5155901222230006  | Transação Não Autorizada | Autorização Recusada |
| 5155901222220007  | Transação Não Autorizada | Transacao Não Processada |
| 5155901222210008  | Transação Não Autorizada | Excede o Limite de Retiradas |

## Como enviar a requisição

Envie os dados em uma requisição POST para o endpoint de pagamentos conforme o formato abaixo:

[Referência da API: Pagamentos](https://docs.gen.com.br/#afd46332-94bb-45f1-8e99-be8e9fe19b41)
```bash
curl -X POST "https://api-sandbox.genpag.com.br/api/sellers/:sellerId/payments" \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer (access token)' \
--data '{
    "payment": {
        "session_id": "123",
        "order_id": "6c48f2e3-722a-8198-fffc-73b458502665",
        "method": "CREDIT_CARD",
        "credit": {
            "installments": 2,
            "delay_capture": false,
            "save_card": false,
            "credit_card_id": "e035c42f-1ad9-4edd-db73-ebc0a2b227cc",
            "credit_card": {
                "token": "token-do-cartao",
                "holder_name": "Nome do portador",
                "expiration_month": 12,
                "expiration_year": 24
            }
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
## Dados de Retorno

Os dados da transação são retornados em um objeto **data**. Para pagamentos no crédito é importante você usar o id do pagamento nas demais operações, como captura de pre-autorizações ou cancelamentos.

```json
{
    "data": {
        "id": "12ac0bcb-397b-776d-a621-fce33ec302e9",
        "session_id": "123",
        "order_id": "6c48f2e3-722a-8198-fffc-73b458502665",
        "method": "CREDIT",
        "seller_id": "60b2eac6-bf01-58ee-f488-67d43c6d2d0f",
        "marketplace_id": "7a65dc09-e1ec-93f2-fb01-9acb4e9f1fa2",
        "customer_id": "e035c42f-1ad9-4edd-db73-ebc0a2b227cc",
        "subscription_id": null,
        "subtotal_cents": 10000,
        "discount_cents": 0,
        "addition_cents": 0,
        "shipping_cents": 0,
        "total_cents": 10000,
        "status": "PAID",
        "device": {},
        "credit": {
          "installments": 3,
          "delay_capture": false,
          "save_card": false,
          "credit_card_id": "6c48f2e3-722a-8198-fffc-73b458502665",
          "credit_card": {
             "token": "123abc",
             "expiration_month": "12",
             "expiration_year": "22"
          }
        },
        "external_id": "id_so_seu_sistema",
        "inserted_at": "2022-03-01T22:22:36.878Z",
        "updated_at": "2002-03-01T22:22:36.878Z"
    }
}
```

## Mudanças de status

Para pagamentos sem atraso na captura, **(delay_capture false)**, ao ser aprovada o status será pago **(PAID)**. Em pagamentos com captura tardia **(delay_capture true)** a transação ficará com status pré-autorizado **(PRE_AUTHORIZED)**. Somente após a captura ela irá para o status pago **(PAID)**.

Ao cancelar uma pré-autorização ou uma transação o pagamento irá para o status cancelado **(CANCELLED)**.

Todas as mudanças de status ou atualizações sobre um pagamento ou pedido são enviadas para webhooks cadastrados.

## Captura de Pagamentos
Para pagamentos no cartão de crédito com opção de captura tardia **(delay_capture true)** você vai precisar fazer outra chamada para capturar a transação. Para isso utilize o ID do pagamento gerado anteriormente e execute a seguinte chamada POST:

[Referência API: Captura de Pagamento](https://docs.gen.com.br/#fe6efcc6-2012-4847-ad20-2030b92a2578)

```bash
curl -X POST "https://api-sandbox.genpag.com.br/api/sellers/:sellerId/payments/:paymentId/capture"
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer (access token)'
--data '{}'
```

Utilize o ID da sua conta **(sellerId)** e do id do pagamento **(paymentId)** no caminho da URL.


{% include links.html %}