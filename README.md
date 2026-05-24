# CityDash — Trabalho de Modelagem

## Engenharia de Software

> **Plataforma de entregas multi-categoria com foco em agilidade, transparência e experiência do usuário**

---

## 📋 Sobre o Projeto

O **CityDash** é uma plataforma de entregas que conecta clientes, empresas parceiras (restaurantes, mercados, farmácias, pet shops, etc.) e entregadores independentes. A plataforma se diferencia por:

- **Rastreamento em tempo real** com precisão superior
- **Múltiplas categorias** de produtos e serviços
- **Algoritmo inteligente** de matching entregador-pedido
- **Programa de fidelidade** integrado para clientes

---

## 🎯 Objetivo deste Trabalho

Este repositório contém a **modelagem em profundidade** de 3 fatias verticais representativas do sistema CityDash, conforme especificação do Trabalho 3 de Engenharia de Software.

**Pré-requisitos entregues:**
- ✅ Documento de Visão do Produto (Trabalho 1)
- ✅ Documento de Requisitos (Trabalho 2 — histórias de usuário, MoSCoW, requisitos não funcionais)

---

## 📂 Estrutura do Repositório

```
README.md                              — este arquivo
docs/
  00-selecao-de-escopo.md              — Justificativa das 3 fatias selecionadas
  01-diagrama-de-classes.md            — Diagrama de classes UML
  02-mer.md                            — Modelo Entidade-Relacionamento
  03-comportamental-fatia1.md          — Diagramas comportamentais — Fatia 1
  03-comportamental-fatia2.md          — Diagramas comportamentais — Fatia 2
  03-comportamental-fatia3.md          — Diagramas comportamentais — Fatia 3
  04-casos-de-teste.md                 — 6 casos de teste (2 por fatia)
  05-rastreabilidade.md                — Tabela de rastreabilidade completa
references.md                          — Referências bibliográficas (ABNT)
```

---

## 🔍 Fatias Verticais Selecionadas

### Fatia 1 — Cliente realiza pedido com pagamento e alocação automática de entregador
**Must Have** | Múltiplos subsistemas | Regras de negócio complexas

Cobre o fluxo completo desde a seleção de produtos até a confirmação do pedido, incluindo integração com gateway de pagamento e algoritmo de matching para atribuição automática de entregador.

### Fatia 2 — Ciclo de vida da entrega com rastreamento em tempo real
**Must Have** | Múltiplos atores | Estados com transições temporais

Modela a jornada completa da entrega: desde a aceitação pelo entregador, coleta na empresa, trajeto monitorado, até a confirmação de entrega e avaliação mútua.

### Fatia 3 — Sistema de gamificação e bonificação para entregadores
**Should Have** | Regras de negócio não-triviais | Cálculos dinâmicos

Implementa lógica de pontuação, conquistas (badges), rankings e cálculo de bonificações variáveis baseadas em performance, horários e demanda.

---

## 🛠 Ferramentas Utilizadas

- **Mermaid** — diagramas UML versionáveis em texto
- **Markdown** (CommonMark/GFM) — documentação
- **GitHub** — versionamento e entrega

---

## 📚 Convenções

- **Padrão de casos de teste**: IEEE-830
- **Citações**: ABNT
- **Nomenclatura de IDs**: `TC-FATIAX-YY` para casos de teste, `US-SUBSISTEMA-NNN` para histórias de usuário

---

## 👥 Equipe

Bruna Kinjo Luiz Pinto

Kayke Ahrens Biscegli

---

**Navegue para [`docs/00-selecao-de-escopo.md`](docs/00-selecao-de-escopo.md) para começar a leitura do trabalho.**
