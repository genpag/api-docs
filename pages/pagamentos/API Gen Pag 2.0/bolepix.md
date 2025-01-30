# Documentação de Integração - API Bolepix Genpag

## Introdução

O **Bolepix** é um método de pagamento híbrido que combina as vantagens do **boleto bancário** com a rapidez e praticidade do **PIX**. Com essa solução, os usuários podem pagar suas cobranças tanto via **QR Code PIX** quanto por **código de barras do boleto**.

Esta documentação descreve a **integração da API Bolepix da Genpag**, abordando os seguintes processos essenciais:

1. **Criação de cobrança Bolepix Celcoin**
2. **Geração do PDF da cobrança**
3. **Consulta de cobranças**
4. **Cancelamento de cobranças**
5. **Retorno de cobrança (webhook)**

## 1. Teste de Criação de Cobrança Bolepix Celcoin

### Autenticação na API

Antes de criar uma cobrança, é necessário autenticar-se na API da Genpag.

- **Acesse:** [API Dev Genpag](https://api-dev.gen.com.br/graphql)
- **Realize o login** conforme descrito no documento [Autenticação e Token](https://www.notion.so/teste_chamadas_mostqi-12ffeb246d1d805faf70f3dfd321da4e?pvs=21)
- **Configure o Bearer Token** para acessar os endpoints

### Criando uma Cobrança Bolepix

Após a autenticação, utilize a seguinte mutation GraphQL para criar uma cobrança:

```graphql
mutation {
  createInvoiceBolepix(
    params: {
      order_id: "a7b92991-c0a7-45e9-93ad-47e80bee7028"
      expirationAfterPayment: 1
      dueDate: "2024-12-17"
      amount: 5.0
      debtor: {
        number: "123"
        neighborhood: "Alphaville Residencial Um"
        name: "Erick Augusto Farias"
        document: "42318970858"
        city: "Barueri"
        publicArea: "Alameda Holanda"
        state: "SP"
        postalCode: "06474320"
      }
      instructions: {
        fine: 2.0
        interest: 2.0
        discount: {
          amount: 1.0
          modality: "fixed"
          limitDate: "2024-12-17T00:00:00.0000000"
        }
      }
    }
  ) {
    message
  }
}
```

## 2. Teste de Geração do PDF da Cobrança Bolepix Celcoin

### Obtendo o PDF da Cobrança

Após gerar a cobrança, utilize a seguinte query para obter o PDF:

```graphql
query {
  invoicePDFBolepix(
    sellerId: "a7b92991-c0a7-45e9-93ad-47e80bee7028"
    invoiceId: "5b848292-c1d2-4c48-918f-43f08e0dcd95"
  ) {
    url
  }
}
```

O **invoiceId** gerado deve ser utilizado para consultar a cobrança e acessar o PDF.

## 3. Teste de Consulta de Cobrança Bolepix Celcoin

### Listando Cobranças

Para listar todas as cobranças de um `sellerId`, utilize:

```graphql
query {
  listInvoiceBolepix(sellerId: "algum_seller_id") {
    id
    sellerId
    invoiceId
    insertedAt
    updatedAt
  }
}
```

### Consulta Detalhada de uma Cobrança

```graphql
query {
  bolepixById(
    sellerId: "algum_seller_id"
    invoiceId: "alguma_cobranca_bolepix"
  ) {
    id
    sellerId
    invoiceId
    insertedAt
    updatedAt
  }
}
```

## 4. Teste de Cancelamento de Cobrança Bolepix Celcoin

### Cancelando uma Cobrança

```graphql
mutation {
  cancelBolepixInvoice(
    invoiceId: "algum_invoice_id"
    reason: "Falta de saldo para pagar"
  ) {
    message
  }
}
```

Caso seja bem-sucedido, o retorno será:

```json
{
  "message": "success"
}
```

## 5. Teste de Retorno de Cobrança Bolepix (webhook)

### Verificando o Status no Banco

Após criar ou cancelar uma cobrança, consulte a base `gen_pag_dev_elixir_db`:

```sql
SELECT *
FROM public.payments
WHERE invoice_id = 'algum_invoice_id';
```

## Como Funciona a Jornada de Pagamento

- Criar uma cobrança utilizando `createInvoiceBolepix`
- Gerar o PDF da cobrança com `invoicePDFBolepix`
- Efetuar o pagamento utilizando o QR Code ou código de barras gerado

### Webhooks e Atualizações de Status

Os seguintes eventos de Webhook estão disponíveis:

| Evento             | Descrição                             |
| ------------------ | ------------------------------------- |
| `charge-create`    | Cobrança gerada                       |
| `charge-in`        | Pagamento confirmado e split de taxas |
| `charge-cancelled` | Cobrança cancelada                    |
