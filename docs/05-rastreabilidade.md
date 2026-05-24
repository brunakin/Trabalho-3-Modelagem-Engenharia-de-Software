# 5. Rastreabilidade

Este documento apresenta a matriz de rastreabilidade bidirecional do projeto **CityDash**, conectando os requisitos levantados no Trabalho 2 (Casos de Uso e Requisitos Não-Funcionais) com os artefatos de modelagem produzidos no Trabalho 3 (Diagrama de Classes, MER e Diagramas Comportamentais).

O objetivo é garantir que todas as fatias verticais selecionadas para modelagem profunda estejam devidamente ancoradas nas necessidades de negócio aprovadas anteriormente, e comprovar que nenhuma classe ou tabela existe "no vácuo" sem um requisito que a justifique.

## Matriz de Rastreabilidade: Requisitos ➔ Modelagem

A tabela abaixo mapeia os principais Casos de Uso (UC) das três fatias verticais selecionadas para os elementos arquiteturais modelados neste trabalho.

| ID Requisito  | Descrição do Requisito (Trabalho 2)         | Fatia Vertical | Classes Envolvidas (Diag. Classes)                  | Entidades Relacionadas (MER)    | Diagramas Comportamentais        |
| :------------ | :------------------------------------------ | :------------- | :-------------------------------------------------- | :------------------------------ | :------------------------------- |
| **UC-01**     | Solicitar Entrega (Cálculo de rota e preço) | Fatia 1        | `Cliente`, `Pedido`, `Endereco`, `CalculadoraFrete` | `CLIENTE`, `PEDIDO`, `ENDERECO` | Diagrama de Sequência (Fatia 1)  |
| **UC-03**     | Realizar Pagamento (Cartão, Pix, Boleto)    | Fatia 1        | `Pedido`, `Pagamento`, `GatewayPagamento`           | `PAGAMENTO`                     | Diagrama de Sequência (Fatia 1)  |
| **UC-06**     | Avaliar Entregador (Atendimento e Tempo)    | Fatia 2        | `Cliente`, `Entregador`, `Avaliacao`                | `AVALIACAO`                     | Diagrama de Estados (Fatia 2)    |
| **UC-08**     | Ficar Online/Offline (Disponibilidade)      | Fatia 2        | `Entregador`, `StatusDisponibilidade`               | `ENTREGADOR`                    | Diagrama de Estados (Fatia 2)    |
| **UC-09**     | Receber e Responder Oferta de Entrega       | Fatia 2        | `Entregador`, `Pedido`, `Oferta`                    | `PEDIDO_OFERTA`                 | Diagrama de Estados (Fatia 2)    |
| **UC-10**     | Atualizar Status da Entrega                 | Fatia 2        | `Pedido`, `StatusEntrega`                           | `PEDIDO`, `HISTORICO_STATUS`    | Diagrama de Estados (Fatia 2)    |
| **UC-12**     | Cadastrar Empresa e Região de Atuação       | Fatia 3        | `Empresa`, `RegiaoAtuacao`                          | `EMPRESA`, `REGIAO`             | Diagrama de Atividades (Fatia 3) |
| **UC-13**     | Gerenciar Entregadores (Editar/Desativar)   | Fatia 3        | `Empresa`, `Entregador`                             | `EMPRESA`, `ENTREGADOR`         | Diagrama de Atividades (Fatia 3) |
| **NF-CD-001** | Tempo de Resposta e Performance             | Todas          | _Aplicado nos métodos assíncronos_                  | _Índices de banco de dados_     | Todos                            |
| **NF-CD-005** | Segurança e Controle de Acesso              | Todas          | `Usuario`, `Sessao`                                 | `USUARIO`, `PERFIL`             | Diagrama de Sequência (Fatia 1)  |

## Justificativa de Divergências e Adaptações

Durante a transição da fase de requisitos para a modelagem estrutural e de dados, algumas adaptações foram necessárias:

1. **Relacionamentos N:N:** Requisitos que indicavam múltiplos itens ou múltiplas ofertas (como o UC-09, onde um pedido é ofertado a vários entregadores e um entregador recebe vários pedidos) foram resolvidos no MER através de tabelas associativas (ex: `PEDIDO_OFERTA`), que não apareciam explicitamente como classes de domínio no Diagrama de Classes.
2. **Atributos Calculados:** A classe `CalculadoraFrete` (Fatia 1) existe no Diagrama de Classes como uma representação de regra de negócio (Strategy/Service), mas não possui correspondência no MER, pois seus resultados (preço estimado) são salvos diretamente na entidade `PEDIDO`, não exigindo uma tabela própria de calculadora.
3. **Herança:** Os diferentes métodos de pagamento do UC-03 foram modelados com herança no Diagrama de Classes (`PagamentoPix`, `PagamentoCartao`), mas no MER optou-se pela estratégia de _Single Table_ (Tabela Única) com uma coluna discriminadora (`tipo_pagamento`) na tabela `PAGAMENTO` para otimizar a performance do banco de dados, atendendo ao requisito **NF-CD-001**.
