# Comunicação entre APIs Laravel 11 e .NET Core 9 com Docker

#### **Descrição**  
Este projeto demonstra a integração entre uma API em Laravel 11 e outra em .NET Core 9, rodando em containers Docker e se comunicando dentro de uma rede interna. A API Laravel gerencia pedidos, enquanto a API .NET processa pagamentos. O foco é estruturar URLs corretamente, usar métodos HTTP adequados, trabalhar com query strings e garantir comunicação eficiente entre os serviços.  

#### **Pré-requisitos**  
Para rodar o projeto, é necessário ter instalado:  
- **Docker & Docker Compose**  
- **PHP 8.3 + Composer** (para Laravel)  
- **.NET SDK 9** (para a API .NET)  
- **Node.js** (se precisar rodar o frontend futuramente)  

#### **Recursos do projeto**  
- Arquitetura em camadas (Controller, Service, Repository).  
- Tratamento de erros adequado.  
- Comunicação entre APIs via HTTP.  
- Exemplos de requisições e manipulação de query strings.  
- Estrutura pronta para expansão.  

#### **Execução do projeto**  

1. Clone o repositório:  
   ```sh
   git clone https://github.com/seu-usuario/seu-repositorio.git  
   cd seu-repositorio
   ```

2. Suba os containers com Docker Compose:  
   ```sh
   docker-compose up -d  
   ```

3. Acesse os serviços:  
   - Laravel API: [http://localhost:8000/api/pedidos](http://localhost:8000/api/pedidos)  
   - .NET API: [http://localhost:5000/api/pagamento](http://localhost:5000/api/pagamento)  

4. Teste as requisições via cURL ou Postman:  
   ```sh
   curl -X GET "http://localhost:8000/api/pedidos?status=pendente"  
   ```

#### **Referências**  
- [Documentação do Laravel](https://laravel.com/docs)  
- [Documentação do .NET](https://learn.microsoft.com/en-us/dotnet/)  
- [Docker Docs](https://docs.docker.com/)  

