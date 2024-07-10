---
title: Gerando pedidos
keywords: pedidos, pagamentos, split de pagamentos, crédito, pix, boleto
last_updated: July 3, 2016
tags: [pedidos, pagamentos, split, crédito, pix, boleto]
summary: "Veja como gerar pedidos e escolher os métodos de pagamento que irá aceitar"
sidebar: mydoc_sidebar
permalink: gerando-pedidos.html
folder: pagamentos
---
## Introdução

Neste guia vamos ver como cadastrar uma fatura através de nossa API. Antes de realizar pagamentos é necessário cadastrar uma fatura. No cadastro da fatura você pode:
* Informar os itens que estão sendo comercializados;
* Escolher e restringir os métodos de pagamento a serem aceitos (opcional);
* Escolher o número máximo de parcelas (opcional);
* Cadastrar o cliente que irá efetuar o pagamento (opcional).
* Cadastrar taxas para serem dividas entre outros estabelecimentos cadastrados na Gen (Somente para split de pagamentos)

## Criando a fatura

Você deve cadastrar uma fatura na plataforma da Gen informando os dados dos itens omercializados. Os dados do cliente são opcionais, você pode informar ou deixar que ele insira durante o pagamento.

Envie uma requisição POST na API de pedidos com os dados no formato a seguir:

[Referência da API: Pedidos](https://docs.gen.com.br/#4617b1ac-d942-4645-bc0c-760e790c0c13)

``` bash
curl -X POST 'https://api-sandbox.genpag.com.br/api/sellers/:sellerId/orders' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer (access token)' \
    --data '{
        "order": {
            "items": [
                {
                    "description": "Produto 1",
                    "quantity": 2,
                    "unit_price_cents": 2290
                }
            ],
            "customer": {
                "name": "Jhon Trevor",
                "email": "customer@email.com",
                "cpf": "01234567891",
                "phone": "+5562992929292",
                "birth_date": "1965-08-10",
                "cnpj": "12345678000166",
                "address": {
                  "line1": "Rua tal",
                  "line2": "0",
                  "line3": "QD 2, LT 1, Ed. Sala 100",
                  "neighborhood": "Centro",
                  "city": "São Paulo",
                  "state": "SP",
                  "country_code": "BR",
                  "postal_code": "01153-000"
                }
            },
            "external_id": "id-sua-plataforma-123"
        }
    }'
```
* **items**: Um array contendo os itens vendidos
    * **description**: Descrição do item
    * **unit_price_cents**: Valor do item em centavos
    * **quantity**: Quantidade de itens
* **customer_id**: O ID de um cliente já cadastrado na plataforma. Você pode enviar somente este campo, ou então os dados do cliente no objeto customer
* **customer**
    * **address**: Endereço do cliente
        * **line1**: Rua
        * **line2**: Número ou 0 se não houver número no endereço
        * **line3**: Complemento
        * **city**: Cidade
        * **state**: Estado, sigla
        * **neighborhood**: Bairro
        * **postal_code**: CEP
        * **country_code**: Código do país
    * **cpf**: CPF do cliente
    * **cnpj**: CNPJ do cliente se for pessoa jurídica
    * **phone**: Telefone do cliente
    * **email**: E-mail do cliente
* **external_id**: O ID de identificação da fatura na sua plataforma. Essa informação é opcional


## Restringindo formas de pagamento

As formas de pagamento disponíveis na plataforma são:
* Cartão de crédito;
* Boleto;
* PIX.
Por padrão, todos os métodos ativados são aceitos para uma fatura recém criada, mas você pode restringir quais formas de pagamento podem ser utilizadas de forma global em sua conta através da nossa plataforma web ou em um pedido específico utilizando o campo **methods**.

Os valores disponíveis são:

* **CREDIT**: Cartão de crédito
* **BANK_SLIP**: Boleto
* **PIX**: Pix de cobrança

```bash
curl -X POST 'https://api-sandbox.genpag.com.br/api/sellers/:sellerId/orders' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer (access token)' \
    --data '{
        "order": {
            "items": [
                {
                    "description": "Produto 1",
                    "quantity": 2,
                    "unit_price_cents": 2290
                }
            ],
            "methods": ["BANK_SLIP", "PIX"]
        }
    }'
```

## Restringindo número máximo de parcelas

É possível parcelar os pagamentos no cartão de crédito em até 12x. Voce pode restringir o número máximo de parcelas para determinado pedido utilizando o campo **max_installments**. Essa restrição será aplicada em todas as nossas interfaces de checkout, seja por API, link de pagamento ou aplicativo. Confira o exemplo a seguir:

```bash
curl -X POST 'https://api-sandbox.genpag.com.br/api/sellers/:sellerId/orders' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer (access token)' \
    --data '{
        "order": {
            "items": [
                {
                    "description": "Produto 1",
                    "quantity": 2,
                    "unit_price_cents": 2290
                }
            ],
            "max_installments": 3 // Restringe o parcelamento em até 3x
        }
    }'
```

## Integrando com nosso checkout

Se você quiser uma integração rápida, disponibilizamos uma página de checkout pronta, basta redirecionar o usuário para nosso checkout que ao final do fluxo redirecionamos ele de volta para seu sistema.
Após criar o pedido, utilize o padrão de URL a seguir para redirecionar o usuário:

// Ambiente de homologação
// https://checkout.dev.gen.com.br/external/order/:id?redirectTo=<sua_url>
"https://checkout.dev.gen.com.br/external/order/:id?redirectTo=https://example.com"

// Ambiente de produção
// https://checkout.gen.com.br/external/order/:id?redirectTo=<sua_url>
"https://checkout.gen.com.br/external/order/:id?redirectTo=https://example.com"


No campo :id utilize o ID do pedido gerado anteriormente e o usuário final irá acessar a interface de checkout.


## Pŕoximos passos

Após gerar uma fatura, o próximo passo é registrar um ou mais pagamentos para completar o valor total da fatura. Vocẽ pode fazer isso usando nosso checkout, você só vai precisar do ID da fatura gerada, ou através de nossa API. Caso queira customizar toda a experiência do usuário veja os pŕoximos passos:
* Aprenda a gerar boletos para faturas criadas. Ver Tutorial;
* Aprenda a gerar PIX de cobrança. [Ver Tutorial]({% link pages/pagamentos/gerando_um_pix.md %});
* Veja como realizar o pagamento de uma fatura utilizando cartão de crédito;
* Cadasre webhooks para receber notificações de eventos da nossa API no seu sistema.

{% include links.html %}
