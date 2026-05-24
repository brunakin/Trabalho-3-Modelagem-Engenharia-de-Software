# 0. Seleção de Escopo

## 0.1 Contextualização do Sistema CityDash

O **CityDash** é uma plataforma de entregas multi-categoria que opera em três camadas de atores:

| Subsistema | Atores Principais |
|------------|------------------|
| **Pedidos & Catálogo** | Cliente, Empresa Parceira |
| **Logística & Entrega** | Entregador, Cliente, Sistema de Rastreamento |
| **Pagamentos & Gamificação** | Cliente, Entregador, Gateway Externo, Sistema de Bonificação |

O sistema se diferencia por combinar funcionalidades tradicionais de delivery (pedido, pagamento, rastreamento) com elementos inovadores como gamificação para entregadores, algoritmo inteligente de matching, e programa de fidelidade integrado.

Dado que modelar todo o sistema seria superficial, este trabalho foca em **3 fatias verticais** que atravessam componentes críticos e expõem a complexidade técnica e de negócio do CityDash.

---

## 0.2 Critério de Seleção das Fatias

As três fatias foram escolhidas para cobrir obrigatoriamente:

**Pelo menos 1 fatia Must Have** da priorização MoSCoW  
**Pelo menos 1 fatia envolvendo múltiplos subsistemas/atores**  
**Pelo menos 1 fatia com regras de negócio não-triviais**

---

## 0.3 Fatias Selecionadas

### Fatia 1 — Cliente realiza pedido com pagamento e alocação automática de entregador

**Histórias de Usuário Cobertas:**
- **US-PED-001**: "Como cliente, quero adicionar itens de uma empresa parceira ao carrinho para comprar de forma organizada"
- **US-PED-005**: "Como cliente, quero aplicar cupons de desconto antes de finalizar o pedido"
- **US-PAG-001**: "Como cliente, quero pagar com cartão de crédito/débito de forma segura via gateway integrado"
- **US-LOG-001**: "Como sistema, quero alocar automaticamente um entregador disponível baseado em proximidade, avaliação e tempo de resposta"

**Por que é representativa:**

- É **Must Have absoluto** — sem fluxo de pedido e pagamento, a plataforma não existe.
- Atravessa **três subsistemas**: Pedidos (validar carrinho e cupons), Pagamentos (autorizar transação com gateway externo), Logística (disparar algoritmo de matching para encontrar entregador).
- Envolve **integração crítica com gateway externo** (Stripe/Mercado Pago) — exige modelagem de comunicação assíncrona, webhooks e tratamento de falhas.
- Tem **múltiplos caminhos de erro**: cupom inválido, item indisponível entre adicionar e finalizar, cartão recusado, nenhum entregador disponível na área.

**O que esperamos aprender:**

- Como modelar um **fluxo distribuído orquestrado** onde a falha em um passo (ex.: pagamento recusado) deve reverter ações anteriores (liberar entregador alocado).
- Como representar **algoritmo de matching** em diagrama de sequência — decisão que envolve consultar disponibilidade de múltiplos entregadores, calcular distância, priorizar por rating e tempo de resposta.
- Como expressar **transações compensatórias** em caso de erro parcial (ex.: pagamento aprovado mas nenhum entregador aceita — precisa estornar).

---

### Fatia 2 — Ciclo de vida da entrega com rastreamento em tempo real

**Histórias de Usuário Cobertas:**
- **US-LOG-003**: "Como entregador, quero visualizar detalhes do pedido atribuído (endereços, itens, valor) antes de aceitar"
- **US-LOG-004**: "Como entregador, quero confirmar que coletei o pedido na empresa parceira"
- **US-LOG-006**: "Como cliente, quero acompanhar a localização do entregador em tempo real no mapa"
- **US-LOG-009**: "Como cliente, quero confirmar o recebimento do pedido e avaliar a experiência da entrega"
- **US-LOG-010**: "Como entregador, quero avaliar a experiência com o cliente (educação, disponibilidade)"

**Por que é representativa:**

- Envolve **três atores distintos em interação direta**: Cliente (acompanha), Entregador (executa), Empresa Parceira (libera pedido).
- Tem **máquina de estados clara com 8 estados**: `Aguardando Aceite → Aceita → A Caminho da Coleta → Coletando → Em Trânsito → Chegando → Entregue → Avaliada`.
- Cada transição tem **evento disparador, condições de guarda e ações automáticas** (ex.: ao entrar em "Em Trânsito", notificar cliente via push; ao permanecer em "Aguardando Aceite" por 5min, realocar para outro entregador).
- Tem **regras temporais críticas**: timeout de aceitação, tempo máximo de coleta, janela de entrega prometida ao cliente.

**O que esperamos aprender:**

- Como modelar **ciclo de vida com estados temporais** usando diagrama de estados com eventos de timeout (`after(5min)`).
- Diferença entre **estado da entrega** (o que modelamos aqui) e **estado do pedido** (que pode estar "Pago" enquanto entrega está "Aguardando Aceite").
- Como representar **eventos externos assíncronos** (atualização de GPS do entregador) que disparam transições.

---

### Fatia 3 — Sistema de gamificação e bonificação para entregadores

**Histórias de Usuário Cobertas:**
- **US-GAM-001**: "Como entregador, quero visualizar meu ranking semanal/mensal comparado a outros entregadores da minha região"
- **US-GAM-003**: "Como entregador, quero desbloquear badges por conquistas (100 entregas, avaliação 4.8+, streak de 7 dias consecutivos)"
- **US-GAM-005**: "Como sistema, quero calcular bonificação variável do entregador baseada em performance, horários de pico e metas atingidas"
- **US-BON-002**: "Como entregador, quero receber bonificação extra por entregas realizadas em horários de alta demanda"

**Por que é representativa:**

- Tem **regras de negócio complexas e compostas**:
  - Bonificação base = % do valor da entrega
  - Multiplicadores: horário de pico (+30%), região de alta demanda (+20%), streak ativo (+10%)
  - Badges desbloqueados por marcos: 10/50/100/500/1000 entregas, avaliação média ≥ 4.5, tempo médio de entrega abaixo da meta
  - Ranking calculado semanalmente com peso diferenciado (60% entregas, 30% avaliação, 10% tempo médio)

- Envolve **cálculos dinâmicos em tempo real** (bonificação de cada entrega) e **processamento batch periódico** (atualização de rankings e desbloqueio de badges).

- É **Should Have** no MoSCoW — não é crítico para MVP, mas é diferencial competitivo importante. Escolhida porque expõe complexidade de domínio que as outras fatias não capturam.

- Modela **polimorfismo aplicado**: diferentes estratégias de cálculo de bonificação (por categoria de entregador: Bronze/Prata/Ouro) e diferentes tipos de badges (por meta, por tempo, por avaliação).

**O que esperamos aprender:**

- Como modelar **regras parametrizáveis** sem hard-code — tabela de multiplicadores configurável por horário/região.
- Como representar **hierarquia de estratégias** (padrão Strategy para cálculo de bonificação).
- A relação entre **diagrama de classes** (estrutura do sistema de pontos/badges) e **diagrama de atividades** (fluxo de cálculo e atualização).

---

## 0.4 Cobertura dos Critérios Obrigatórios

| Critério | Fatia 1 | Fatia 2 | Fatia 3 |
|----------|---------|---------|---------|
| **Must Have do MoSCoW** | Sim | Sim | Poderia Ter |
| **Múltiplos subsistemas/atores** | 3 subsistemas | 3 atores | Foco em 1 subsistema |
| **Regras de negócio não-triviais** | Matching + compensação | Estados + timeouts | Cálculo composto + estratégias |

**Todos os três critérios estão cobertos.** Cada fatia traz uma dimensão complementar do sistema CityDash.

---

## 0.5 O que Fica de Fora (e Por Quê)

As seguintes histórias do Trabalho 2 **não serão modeladas** neste trabalho:

### Cadastro e Autenticação
**Histórias**: US-CAD-001 a US-CAD-004 (cadastro de cliente, empresa, entregador; login social; recuperação de senha)

**Justificativa**: Fluxos padrão de autenticação com OAuth2/JWT. Não agregam aprendizado de modelagem de domínio específico. Trataremos autenticação como pré-condição dos fluxos modelados.

### Programa de Fidelidade do Cliente
**Histórias**: US-FID-001 a US-FID-003 (acumular pontos, resgatar benefícios, histórico de pontos)

**Justificativa**: Similar em estrutura à gamificação de entregadores (Fatia 3). Modelar ambos seria redundante — o padrão de pontos/níveis/recompensas é análogo. Implícito na rastreabilidade que cliente seguirá estrutura similar.

### Chat Integrado Cliente-Entregador
**Histórias**: US-COM-001, US-COM-002 (troca de mensagens, notificações)

**Justificativa**: Funcionalidade importante para produção, mas predominantemente infraestrutura (WebSocket/Firebase). Modelagem se resumiria a CRUD de mensagens. Pouco valor didático.

### Painel Administrativo da Empresa Parceira
**Histórias**: US-EMP-005 a US-EMP-010 (gerenciar cardápio, horários, relatórios de vendas)

**Justificativa**: Conjunto extenso de telas operacionais, mas predominantemente CRUD. Modelar todas ocuparia espaço sem agregar aprendizado. O essencial (empresa disponibiliza produtos) está implícito na Fatia 1.

### Sistema de Recomendação
**Histórias**: US-REC-001, US-REC-002 (sugestões personalizadas baseadas em histórico e preferências)

**Justificativa**: Algoritmo de ML (collaborative filtering, content-based) tratado como "caixa preta". A modelagem de domínio não captura a essência do algoritmo, que depende de ciência de dados, não de engenharia de software tradicional.

---

## 0.6 Resumo da Escolha

Privilegiamos **fatias que atravessam camadas** (Fatia 1), **fatias com estados complexos** (Fatia 2) e **fatias com lógica de negócio parametrizável** (Fatia 3). 

A escolha consciente entre "o que modelar vs. deixar implícito" é exercício de **engenharia de trade-offs** — não de completude exaustiva.

---

**Próximo:** [`docs/01-diagrama-de-classes.md`](01-diagrama-de-classes.md)
