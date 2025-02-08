# URLs, Métodos HTTP e Queries na Comunicação entre APIs

## **1. Estrutura da URL**
A URL (Uniform Resource Locator) é composta por diferentes partes que ajudam a identificar recursos em uma aplicação web. Exemplo:

```
http://localhost:8000/api/pedidos?status=pendente&page=2&sort=asc&limit=10&fields=id,nome,valor&search=camisa&range=100-500&date_from=2024-01-01&date_to=2024-12-31
```

### **Componentes da URL:**
- **Protocolo:** `http://` → Define o esquema de comunicação.
- **Host:** `localhost` → Endereço do servidor.
- **Porta:** `8000` → Porta onde a aplicação está rodando.
- **Caminho:** `/api/pedidos` → Rota que identifica um recurso.
- **Query String:** Conjunto de parâmetros usados para personalizar a requisição.

## **2. Métodos de Comunicação HTTP**
Os métodos HTTP definem a ação realizada sobre um recurso da API:

| Método  | Descrição |
|---------|------------|
| **GET** | Buscar dados do servidor sem modificar nada. |
| **POST** | Criar um novo recurso no servidor. |
| **PUT** | Atualizar um recurso existente. |
| **PATCH** | Atualizar parcialmente um recurso. |
| **DELETE** | Remover um recurso do servidor. |

## **3. Estrutura da URI e Query Strings**
A **URI (Uniform Resource Identifier)** identifica um recurso de forma única. As query strings são usadas para passar informações adicionais.

### **Parâmetros de Query Comuns**
- `status=pago` → Filtra por status.
- `page=2` → Define a página de resultados.
- `limit=10` → Número máximo de itens retornados.
- `sort=asc` ou `sort=desc` → Ordenação crescente ou decrescente.
- `fields=id,nome,valor` → Especifica quais campos retornar.
- `search=camisa` → Realiza uma busca por um termo específico.
- `range=100-500` → Filtra resultados dentro de um intervalo numérico.
- `date_from=2024-01-01&date_to=2024-12-31` → Filtra por intervalo de datas.

### **Exemplo de requisição GET com múltiplos filtros**
```
GET http://localhost:8000/api/pedidos?status=pendente&page=2&sort=asc&limit=10&search=camisa
```

## **4. Comunicação entre Laravel e .NET Core**
Laravel enviará requisições para a API de pagamentos desenvolvida em .NET Core 9.

### **Fluxo de comunicação**
1. Laravel registra um pedido na base de dados.
2. Laravel envia uma requisição para a API .NET para processar o pagamento.
3. A API .NET retorna uma resposta confirmando ou negando o pagamento.

### **Exemplo de Requisição Laravel para a API .NET**

#### **Criar um pagamento no Laravel**
```php
use Illuminate\Support\Facades\Http;

$response = Http::post('http://localhost:5000/api/pagamento', [
    'pedidoId' => 123,
    'valor' => 250.00,
    'metodo' => 'cartao_credito'
]);
```

#### **Resposta esperada da API .NET**
```json
{
    "id": 1,
    "pedidoId": "123",
    "valor": 250.00,
    "metodo": "cartao_credito",
    "status": "Aprovado",
    "dataCriacao": "2025-02-06T12:00:00Z"
}
```

### **Endpoints Criados**

#### **Laravel API (Pedidos)**
- `GET /api/pedidos` → Retorna todos os pedidos.
- `POST /api/pedidos` → Cria um novo pedido.
- `PUT /api/pedidos/{id}` → Atualiza um pedido existente.
- `DELETE /api/pedidos/{id}` → Remove um pedido.

#### **.NET Core API (Pagamentos)**
- `GET /api/pagamento/{id}` → Retorna um pagamento específico.
- `POST /api/pagamento` → Processa um pagamento.
- `PUT /api/pagamento/{id}` → Atualiza o status do pagamento.
- `DELETE /api/pagamento/{id}` → Cancela um pagamento.

## **5. Tratamento de Erros e Respostas HTTP**
As APIs devem retornar respostas HTTP apropriadas para cada situação:

| Código | Significado |
|--------|------------|
| **200 OK** | Requisição bem-sucedida. |
| **201 Created** | Recurso criado com sucesso. |
| **400 Bad Request** | Dados inválidos enviados. |
| **401 Unauthorized** | Acesso não autorizado. |
| **404 Not Found** | Recurso não encontrado. |
| **500 Internal Server Error** | Erro inesperado no servidor. |

Exemplo de resposta de erro JSON:
```json
{
    "error": "Pedido não encontrado",
    "code": 404
}
```

---
Com essa estrutura detalhada, garantimos uma comunicação eficaz entre as APIs Laravel e .NET Core, utilizando métodos HTTP corretamente, explorando todos os tipos de parâmetros de query disponíveis e tratando respostas de maneira adequada.

