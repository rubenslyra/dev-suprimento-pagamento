Para implementar uma paginação com controle de início, limites e navegação entre páginas via parâmetros/query, podemos utilizar os recursos nativos do Laravel. A ideia é permitir que o cliente da API (por exemplo, um frontend ou Postman) defina quantos itens por página deseja (`limit`), em qual página quer começar (`page`), e outros parâmetros opcionais, como ordenação.

Aqui está como você pode fazer isso:

---

### **Implementação com Parâmetros de Query**

#### **1. Paginação com `page` e `per_page`**
O Laravel já suporta paginação com os parâmetros `page` e `per_page` por padrão. Podemos aproveitar isso para permitir que o cliente defina quantos itens por página deseja (`per_page`) e em qual página quer começar (`page`).

```php
public function index(): AnonymousResourceCollection
{
    // Define o número de itens por página (padrão: 15)
    $perPage = request()->query('per_page', 15);

    // Define a página atual (padrão: 1)
    $page = request()->query('page', 1);

    // Busca os posts com paginação
    $posts = Post::latest()->paginate($perPage, ['*'], 'page', $page);

    return PostResource::collection($posts);
}
```

**Exemplos de uso:**
- `GET /posts?per_page=10&page=2` → Retorna 10 posts por página, começando na página 2.
- `GET /posts?per_page=5` → Retorna 5 posts por página, começando na página 1.

---

#### **2. Adicionar Ordenação Personalizada**
Podemos permitir que o cliente defina a ordenação dos resultados usando parâmetros como `sort_by` (campo para ordenar) e `sort_dir` (direção da ordenação: `asc` ou `desc`).

```php
public function index(): AnonymousResourceCollection
{
    // Define o número de itens por página (padrão: 15)
    $perPage = request()->query('per_page', 15);

    // Define a página atual (padrão: 1)
    $page = request()->query('page', 1);

    // Define o campo de ordenação (padrão: created_at)
    $sortField = request()->query('sort_by', 'created_at');

    // Define a direção da ordenação (padrão: desc)
    $sortDirection = request()->query('sort_dir', 'desc');

    // Valida a direção da ordenação
    if (!in_array($sortDirection, ['asc', 'desc'])) {
        $sortDirection = 'desc'; // Valor padrão se inválido
    }

    // Busca os posts com ordenação e paginação
    $posts = Post::orderBy($sortField, $sortDirection)
                 ->paginate($perPage, ['*'], 'page', $page);

    return PostResource::collection($posts);
}
```

**Exemplos de uso:**
- `GET /posts?sort_by=title&sort_dir=asc` → Ordena por título em ordem ascendente.
- `GET /posts?sort_by=created_at&sort_dir=desc&per_page=10&page=2` → Ordena por data de criação (mais recentes primeiro), 10 itens por página, página 2.

---

#### **3. Adicionar Filtros Opcionais**
Podemos adicionar filtros opcionais, como buscar posts por título ou conteúdo.

```php
public function index(): AnonymousResourceCollection
{
    // Define o número de itens por página (padrão: 15)
    $perPage = request()->query('per_page', 15);

    // Define a página atual (padrão: 1)
    $page = request()->query('page', 1);

    // Define o campo de ordenação (padrão: created_at)
    $sortField = request()->query('sort_by', 'created_at');

    // Define a direção da ordenação (padrão: desc)
    $sortDirection = request()->query('sort_dir', 'desc');

    // Valida a direção da ordenação
    if (!in_array($sortDirection, ['asc', 'desc'])) {
        $sortDirection = 'desc'; // Valor padrão se inválido
    }

    // Inicia a query
    $query = Post::query();

    // Filtro por título (opcional)
    if ($title = request()->query('title')) {
        $query->where('title', 'like', "%{$title}%");
    }

    // Filtro por conteúdo (opcional)
    if ($content = request()->query('content')) {
        $query->where('content', 'like', "%{$content}%");
    }

    // Aplica ordenação e paginação
    $posts = $query->orderBy($sortField, $sortDirection)
                   ->paginate($perPage, ['*'], 'page', $page);

    return PostResource::collection($posts);
}
```

**Exemplos de uso:**
- `GET /posts?title=Laravel` → Filtra posts com "Laravel" no título.
- `GET /posts?content=API&sort_by=created_at&sort_dir=desc&per_page=5` → Filtra posts com "API" no conteúdo, ordena por data de criação (mais recentes primeiro), 5 itens por página.

---

#### **4. Retornar Metadados Úteis**
Para facilitar a navegação, podemos retornar metadados úteis na resposta, como o total de itens, número de páginas, etc.

```php
public function index(): AnonymousResourceCollection
{
    // Define o número de itens por página (padrão: 15)
    $perPage = request()->query('per_page', 15);

    // Define a página atual (padrão: 1)
    $page = request()->query('page', 1);

    // Define o campo de ordenação (padrão: created_at)
    $sortField = request()->query('sort_by', 'created_at');

    // Define a direção da ordenação (padrão: desc)
    $sortDirection = request()->query('sort_dir', 'desc');

    // Valida a direção da ordenação
    if (!in_array($sortDirection, ['asc', 'desc'])) {
        $sortDirection = 'desc'; // Valor padrão se inválido
    }

    // Inicia a query
    $query = Post::query();

    // Filtro por título (opcional)
    if ($title = request()->query('title')) {
        $query->where('title', 'like', "%{$title}%");
    }

    // Filtro por conteúdo (opcional)
    if ($content = request()->query('content')) {
        $query->where('content', 'like', "%{$content}%");
    }

    // Aplica ordenação e paginação
    $posts = $query->orderBy($sortField, $sortDirection)
                   ->paginate($perPage, ['*'], 'page', $page);

    // Retorna os posts com metadados adicionais
    return PostResource::collection($posts)->additional([
        'meta' => [
            'total' => $posts->total(),
            'per_page' => $posts->perPage(),
            'current_page' => $posts->currentPage(),
            'last_page' => $posts->lastPage(),
        ],
    ]);
}
```

**Exemplo de resposta:**
```json
{
    "data": [
        {
            "id": 1,
            "title": "Primeiro Post",
            "content": "Conteúdo do primeiro post...",
            "created_at": "2023-10-01T12:00:00Z"
        }
    ],
    "meta": {
        "total": 100,
        "per_page": 15,
        "current_page": 1,
        "last_page": 7
    }
}
```

---

### **Resumo dos Parâmetros de Query Suportados**
| Parâmetro     | Descrição                                      | Valor Padrão |
|---------------|------------------------------------------------|--------------|
| `per_page`    | Número de itens por página.                    | 15           |
| `page`        | Página atual.                                  | 1            |
| `sort_by`     | Campo para ordenação (ex: `title`, `created_at`). | `created_at` |
| `sort_dir`    | Direção da ordenação (`asc` ou `desc`).        | `desc`       |
| `title`       | Filtra posts por título (busca parcial).       | -            |
| `content`     | Filtra posts por conteúdo (busca parcial).     | -            |

---

### **Exemplo Completo de Uso**
```plaintext
GET /posts?title=Laravel&sort_by=created_at&sort_dir=desc&per_page=10&page=2
```

**Resultado:**
- Filtra posts com "Laravel" no título.
- Ordena por data de criação (mais recentes primeiro).
- Retorna 10 itens por página, começando na página 2.

---

Essa implementação é flexível, profissional e pronta para uso em APIs reais! 🚀
