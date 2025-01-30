# Documentação da API Bolepix - Genpag (v2)

## Introdução

A **API Bolepix da Genpag** permite a criação, consulta e cancelamento de cobranças via **PIX e boleto** de forma unificada. A nova **versão 2 (v2)** da API será baseada em **REST**.

Esta documentação descreve a estrutura e uso da API Bolepix, cobrindo os seguintes tópicos:

1. **Criação de cobrança Bolepix Celcoin**
2. **Geração do PDF da cobrança**
3. **Consulta de cobranças**
4. **Cancelamento de cobranças**
5. **Retorno de cobrança**

## **Aviso de Beta** 🚀

⚠️ **Esta funcionalidade está em Beta** e pode sofrer modificações nas próximas semanas. Todos os clientes impactados serão comunicados em caso de alterações.

---

## **1. Teste de Criação de Cobrança Bolepix Celcoin**

### **Autenticação na API**

Antes de criar uma cobrança, é necessário autenticar-se na API.

- **Acesse:** [API Dev Genpag](https://api-dev.gen.com.br/graphql)
- **Realize o login** conforme descrito no documento [Autenticação e Token](https://www.notion.so/teste_chamadas_mostqi-12ffeb246d1d805faf70f3dfd321da4e?pvs=21)
- **Configure o Bearer Token** para acessar os endpoints

### **Criando uma Cobrança Bolepix**

Após a autenticação, utilize a seguinte requisição para criar uma cobrança:

#### **Endpoint REST**

```http
POST /api/v2/payments/bolepix
```

#### **Exemplo de Request (JSON)**

```json
{
  "order_id": "a7b92991-c0a7-45e9-93ad-47e80bee7028",
  "expirationAfterPayment": 1,
  "dueDate": "2024-12-17",
  "amount": 5.0,
  "debtor": {
    "number": "123",
    "neighborhood": "Alphaville Residencial Um",
    "name": "Erick Augusto Farias",
    "document": "42318970858",
    "city": "Barueri",
    "publicArea": "Alameda Holanda",
    "state": "SP",
    "postalCode": "06474320"
  },
  "instructions": {
    "fine": 2.0,
    "interest": 2.0,
    "discount": {
      "amount": 1.0,
      "modality": "fixed",
      "limitDate": "2024-12-17T00:00:00.0000000"
    }
  }
}
```

#### **Exemplo de Resposta (JSON)**

```json
{
  "message": "Cobrança criada com sucesso",
  "invoiceId": "5b848292-c1d2-4c48-918f-43f08e0dcd95"
}
```

---

## **2. Teste de Geração do PDF da Cobrança Bolepix Celcoin**

### **Obtendo o PDF da Cobrança**

#### **Endpoint REST**

```http
GET /api/v2/payments/bolepix/{invoiceId}/pdf
```

#### **Exemplo de Resposta (JSON)**

```json
{
  "url": "https://cdn.genpag.com.br/bolepix/5b848292-c1d2-4c48-918f-43f08e0dcd95.pdf"
}
```

---

## **3. Teste de Consulta de Cobrança Bolepix Celcoin**

### **Listando Cobranças**

#### **Endpoint REST**

```http
GET /api/v2/payments/bolepix?sellerId={sellerId}
```

#### **Exemplo de Resposta (JSON)**

```json
[
  {
    "id": "1",
    "sellerId": "123",
    "invoiceId": "ABC123",
    "status": "PENDING",
    "insertedAt": "2024-01-15T10:00:00Z"
  }
]
```

---

## **4. Teste de Cancelamento de Cobrança Bolepix Celcoin**

### **Cancelando uma Cobrança**

#### **Endpoint REST**

```http
POST /api/v2/payments/bolepix/{invoiceId}/cancel
```

#### **Exemplo de Request (JSON)**

```json
{
  "reason": "Falta de saldo para pagar"
}
```

#### **Exemplo de Resposta (JSON)**

```json
{
  "message": "Cobrança cancelada com sucesso"
}
```

---

## **5. Teste de Retorno de Cobrança Bolepix**

### **Verificando o Status da Cobrança**

#### **Query SQL para validação**

```sql
SELECT * FROM payments WHERE invoice_id = 'ABC123';
```

---

## **Fluxo da Jornada de Pagamento**

1. Criar uma cobrança via `POST /api/v2/payments/bolepix`
2. Gerar o PDF da cobrança via `GET /api/v2/payments/bolepix/{invoiceId}/pdf`
3. Cliente efetua pagamento via QR Code ou boleto
4. Status atualizado via webhooks (`charge-in`, `charge-cancelled`)

### **Webhooks e Atualizações de Status**

| Evento             | Descrição            |
| ------------------ | -------------------- |
| `charge-create`    | Cobrança gerada      |
| `charge-in`        | Pagamento confirmado |
| `charge-cancelled` | Cobrança cancelada   |

---

## **Conclusão**

Esta documentação segue **as melhores práticas de clareza, estruturação e exemplos bem formatados** para facilitar a integração com a API Bolepix. Para dúvidas ou suporte, consulte os canais oficiais da Genpag. 🚀
