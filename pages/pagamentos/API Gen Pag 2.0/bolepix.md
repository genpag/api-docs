# 📌 **Documentação da API Bolepix - Genpag V2**

### **📢 Aviso Importante**

Esta funcionalidade está em **Beta** e pode sofrer modificações nas próximas semanas. Todos os clientes impactados serão comunicados sobre eventuais alterações.

## **📌 Introdução**

O **Bolepix** é uma solução híbrida de pagamento que combina a praticidade do **PIX** com a segurança do **boleto bancário**. Esta documentação descreve a **integração da API Bolepix da Genpag V2**, totalmente **RESTful**.

---

### **1️⃣ Criação de Cobrança Bolepix Celcoin**

#### **🔑 Autenticação na API**

Antes de criar uma cobrança, é necessário autenticar-se na API.

- **Acesse:** [API Dev Genpag](https://api-dev.gen.com.br/graphql)
- **Realize o login** conforme descrito no documento [Autenticação e Token](https://www.notion.so/teste_chamadas_mostqi-12ffeb246d1d805faf70f3dfd321da4e?pvs=21)
- **Configure o Bearer Token** para acessar os endpoints

#### **🛠 Criando uma Cobrança**

Após a autenticação, utilize a seguinte requisição para criar uma cobrança:

```http
POST /api/v2/payments/bolepix
```

**📌 Payload:**

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

**📌 Resposta esperada:**

```json
{
  "message": "Cobrança criada com sucesso",
  "invoiceId": "5b848292-c1d2-4c48-918f-43f08e0dcd95"
}
```

---

### **2️⃣ Geração do PDF da Cobrança**

#### **📥 Obtendo o PDF da Cobrança**

```http
GET /api/v2/payments/bolepix/{invoiceId}/pdf
```

**📌 Resposta esperada:**

```json
{
  "url": "https://cdn.genpag.com.br/bolepix/5b848292-c1d2-4c48-918f-43f08e0dcd95.pdf"
}
```

---

### **3️⃣ Consulta de Cobranças**

#### **📂 Listar todas as cobranças de um Seller**

```http
GET /bolepix/invoices?sellerId={sellerId}
Authorization: Bearer {TOKEN}
```

#### **📑 Obter detalhes de uma cobrança específica**

```http
GET /bolepix/invoices/{invoiceId}
Authorization: Bearer {TOKEN}
```

**📌 Resposta esperada:**

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

### **4️⃣ Cancelamento de Cobranças**

#### **📛 Cancelando uma Cobrança**

```http
POST /api/v2/payments/bolepix/{invoiceId}/cancel
```

**📌 Payload:**

```json
{
  "reason": "Falta de saldo para pagar"
}
```

**📌 Resposta esperada:**

```json
{
  "message": "Cobrança cancelada com sucesso"
}
```

---

## **Fluxo da Jornada de Pagamento**

1. Criar uma cobrança via `POST /api/v2/payments/bolepix`
2. Gerar o PDF da cobrança via `GET /api/v2/payments/bolepix/{invoiceId}/pdf`
3. Cliente efetua pagamento via QR Code ou boleto
4. Status atualizado via webhooks (`charge-in`, `charge-cancelled`)

#### **🌍 Webhooks Disponíveis**

| Evento             | Descrição                   |
| ------------------ | --------------------------- |
| `charge-create`    | Cobrança criada com sucesso |
| `charge-in`        | Cobrança paga pelo cliente  |
| `charge-cancelled` | Cobrança cancelada          |

---

## **Conclusão**

Esta documentação segue **as melhores práticas de clareza, estruturação e exemplos bem formatados** para facilitar a integração com a API Bolepix. Para dúvidas ou suporte, consulte os canais oficiais da Genpag. 🚀
