```
dotnet new webapi -n PagamentosAPI

cd PagamentosAPI

dotnet add package Microsoft.EntityFrameworkCore.SqlServer

dotnet add package Microsoft.EntityFrameworkCore.Design

// Model (Models/Pagamento.cs)
using System;

namespace PagamentosAPI.Models
{
    public class Pagamento
    {
        public int Id { get; set; }
        public string PedidoId { get; set; }
        public decimal Valor { get; set; }
        public string Status { get; set; }
        public DateTime DataCriacao { get; set; } = DateTime.UtcNow;
    }
}

// DTO (DTOs/PagamentoDTO.cs)
namespace PagamentosAPI.DTOs
{
    public class PagamentoDTO
    {
        public string PedidoId { get; set; }
        public decimal Valor { get; set; }
    }
}

// Repository (Repositories/PagamentoRepository.cs)
using System.Collections.Generic;
using System.Linq;
using PagamentosAPI.Models;

namespace PagamentosAPI.Repositories
{
    public class PagamentoRepository
    {
        private readonly List<Pagamento> _pagamentos = new();

        public IEnumerable<Pagamento> GetAll() => _pagamentos;
        public Pagamento GetById(int id) => _pagamentos.FirstOrDefault(p => p.Id == id);
        public void Add(Pagamento pagamento) => _pagamentos.Add(pagamento);
    }
}

// Service (Services/PagamentoService.cs)
using PagamentosAPI.DTOs;
using PagamentosAPI.Models;
using PagamentosAPI.Repositories;

namespace PagamentosAPI.Services
{
    public class PagamentoService
    {
        private readonly PagamentoRepository _repository;
        
        public PagamentoService(PagamentoRepository repository)
        {
            _repository = repository;
        }

        public Pagamento CriarPagamento(PagamentoDTO dto)
        {
            var pagamento = new Pagamento
            {
                PedidoId = dto.PedidoId,
                Valor = dto.Valor,
                Status = "Pendente"
            };
            _repository.Add(pagamento);
            return pagamento;
        }
    }
}

// Controller (Controllers/PagamentoController.cs)
using Microsoft.AspNetCore.Mvc;
using PagamentosAPI.DTOs;
using PagamentosAPI.Services;

namespace PagamentosAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class PagamentoController : ControllerBase
    {
        private readonly PagamentoService _service;

        public PagamentoController(PagamentoService service)
        {
            _service = service;
        }

        [HttpPost]
        public IActionResult CriarPagamento([FromBody] PagamentoDTO dto)
        {
            var pagamento = _service.CriarPagamento(dto);
            return CreatedAtAction(nameof(CriarPagamento), new { id = pagamento.Id }, pagamento);
        }
    }
}

// Dockerfile\ 
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
WORKDIR /app
COPY . .
EXPOSE 5000
ENTRYPOINT ["dotnet", "PagamentosAPI.dll"]

// docker-compose.yml
version: '3.8'
services:
  pagamentos_api:
    build: .
    container_name: dotnet_pagamento
    restart: unless-stopped
    ports:
      - "5000:5000"
    networks:
      - api_net

networks:
  api_net:

```
