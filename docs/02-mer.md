# 2. Modelo Entidade-Relacionamento (MER)

## 2.1 Visão Geral

Este MER modela a **persistência** das classes do diagrama UML da Seção 1. Apenas classes cujos dados precisam sobreviver ao término da execução são modeladas como entidades.

**Notação**: Crow's Foot (cardinalidades visuais) com extensões UML para constraints.

**Classes NÃO persistidas** (existem apenas em memória):
- `EstadoEntrega` (subclasses) — apenas o nome do estado é persistido como VARCHAR
- `AlgoritmoMatching` — lógica de negócio, não dados
- `CalculadoraBonificacao` — idem
- `GatewayPagamento` — interface externa

---

## 2.2 Diagrama MER (Mermaid)

```mermaid
erDiagram
    %% ============================================
    %% ENTIDADES CORE
    %% ============================================
    
    CLIENTE {
        UUID id PK
        VARCHAR(100) nome
        VARCHAR(11) cpf UK
        VARCHAR(100) email UK
        VARCHAR(15) telefone
        TIMESTAMP data_cadastro
        BOOLEAN ativo
        INT pontos_fidelidade
    }
    
    ENDERECO {
        UUID id PK
        UUID cliente_id FK
        VARCHAR(200) logradouro
        VARCHAR(10) numero
        VARCHAR(100) complemento
        VARCHAR(100) bairro
        VARCHAR(100) cidade
        VARCHAR(2) estado
        VARCHAR(8) cep
        DECIMAL(10,8) latitude
        DECIMAL(11,8) longitude
        BOOLEAN principal
    }
    
    EMPRESA_PARCEIRA {
        UUID id PK
        VARCHAR(200) razao_social
        VARCHAR(14) cnpj UK
        VARCHAR(100) nome_fantasia
        UUID endereco_id FK
        BOOLEAN aceita_pedidos
        TIME horario_abertura
        TIME horario_fechamento
        TIMESTAMP data_cadastro
    }
    
    PRODUTO {
        UUID id PK
        UUID empresa_id FK
        VARCHAR(200) nome
        TEXT descricao
        DECIMAL(10,2) preco
        VARCHAR(50) categoria
        BOOLEAN disponivel
        INT quantidade_estoque
        JSON atributos_extras
        TIMESTAMP data_cadastro
    }
    
    %% ============================================
    %% CARRINHO E CUPOM
    %% ============================================
    
    CARRINHO {
        UUID id PK
        UUID cliente_id FK
        UUID empresa_id FK
        UUID cupom_id FK "nullable"
        DECIMAL(10,2) subtotal
        DECIMAL(10,2) taxa_entrega
        DECIMAL(10,2) desconto
        TIMESTAMP data_criacao
        TIMESTAMP data_atualizacao
    }
    
    ITEM_CARRINHO {
        UUID id PK
        UUID carrinho_id FK
        UUID produto_id FK
        INT quantidade
        DECIMAL(10,2) preco_unitario
        TEXT observacoes
    }
    
    CUPOM {
        VARCHAR(20) codigo PK
        VARCHAR(20) tipo_desconto "PERCENTUAL|VALOR_FIXO|FRETE_GRATIS"
        DECIMAL(10,2) valor
        TIMESTAMP validade_inicio
        TIMESTAMP validade_fim
        INT uso_maximo
        INT uso_atual
        BOOLEAN ativo
    }
    
    %% ============================================
    %% PEDIDO E PAGAMENTO
    %% ============================================
    
    PEDIDO {
        UUID id PK
        VARCHAR(20) numero_pedido UK
        UUID cliente_id FK
        UUID empresa_id FK
        UUID endereco_entrega_id FK
        DECIMAL(10,2) valor_total
        VARCHAR(30) status "AGUARDANDO_PAGAMENTO|PAGO|EM_PREPARACAO|..."
        TIMESTAMP data_criacao
        TIMESTAMP data_atualizacao
    }
    
    ITEM_PEDIDO {
        UUID id PK
        UUID pedido_id FK
        UUID produto_id FK
        INT quantidade
        DECIMAL(10,2) preco_unitario
        DECIMAL(10,2) subtotal
    }
    
    PAGAMENTO {
        UUID id PK
        UUID pedido_id FK "UNIQUE"
        VARCHAR(50) forma_pagamento "CARTAO_CREDITO|PIX|..."
        DECIMAL(10,2) valor
        VARCHAR(30) status "PENDENTE|APROVADO|RECUSADO|ESTORNADO"
        VARCHAR(100) transacao_gateway_id UK "nullable"
        JSON dados_forma_pagamento "criptografados"
        TIMESTAMP data_processamento
    }
    
    %% ============================================
    %% ENTREGA E RASTREAMENTO
    %% ============================================
    
    ENTREGADOR {
        UUID id PK
        VARCHAR(100) nome
        VARCHAR(11) cpf UK
        VARCHAR(15) telefone
        VARCHAR(10) placa
        VARCHAR(20) tipo_veiculo "MOTO|CARRO|BICICLETA|A_PE"
        BOOLEAN disponivel
        DECIMAL(10,8) latitude_atual
        DECIMAL(11,8) longitude_atual
        DECIMAL(3,2) avaliacao_media
        INT total_entregas
        TIMESTAMP data_cadastro
    }
    
    ENTREGA {
        UUID id PK
        UUID pedido_id FK "UNIQUE"
        UUID entregador_id FK
        UUID endereco_origem_id FK
        UUID endereco_destino_id FK
        DECIMAL(8,2) distancia_km
        DECIMAL(10,2) valor_frete
        VARCHAR(30) estado "AGUARDANDO_ACEITE|ACEITA|EM_TRANSITO|ENTREGUE|..."
        TIMESTAMP data_atribuicao
        TIMESTAMP data_coleta
        TIMESTAMP data_entrega
        VARCHAR(10) codigo_confirmacao "nullable"
    }
    
    PONTO_RASTREAMENTO {
        UUID id PK
        UUID entrega_id FK
        DECIMAL(10,8) latitude
        DECIMAL(11,8) longitude
        TIMESTAMP timestamp
        VARCHAR(100) descricao
    }
    
    AVALIACAO {
        UUID id PK
        UUID entrega_id FK
        VARCHAR(20) tipo "CLIENTE_AVALIA_ENTREGADOR|ENTREGADOR_AVALIA_CLIENTE"
        INT nota "CHECK (nota BETWEEN 1 AND 5)"
        TEXT comentario
        TIMESTAMP data
    }
    
    %% ============================================
    %% GAMIFICAÇÃO
    %% ============================================
    
    PERFIL_GAMIFICACAO {
        UUID id PK
        UUID entregador_id FK "UNIQUE"
        INT pontos_acumulados
        VARCHAR(20) nivel "BRONZE|PRATA|OURO|PLATINA|DIAMANTE"
        INT streak_dias
        TIMESTAMP data_ultima_entrega
    }
    
    BADGE {
        UUID id PK
        VARCHAR(50) nome UK
        TEXT descricao
        VARCHAR(200) icone_path
        VARCHAR(100) criterio "100_ENTREGAS|AVALIACAO_4_8|STREAK_7_DIAS|..."
    }
    
    ENTREGADOR_BADGE {
        UUID entregador_id FK
        UUID badge_id FK
        TIMESTAMP data_desbloqueio
    }
    
    RANKING_SEMANAL {
        UUID id PK
        INT semana
        INT ano
        TIMESTAMP data_atualizacao
    }
    
    POSICAO_RANKING {
        UUID id PK
        UUID ranking_id FK
        UUID entregador_id FK
        INT posicao
        INT total_entregas_semana
        DECIMAL(3,2) avaliacao_media_semana
        DECIMAL(10,2) tempo_medio_entrega
        DECIMAL(10,2) pontos_calculados
    }
    
    TABELA_MULTIPLICADORES {
        UUID id PK
        VARCHAR(50) tipo "HORARIO_PICO|REGIAO_DEMANDA|STREAK"
        VARCHAR(100) condicao "JSON com critérios"
        DECIMAL(4,2) multiplicador "1.00 = 100%, 1.30 = 130%"
        BOOLEAN ativo
    }
    
    %% ============================================
    %% RELACIONAMENTOS
    %% ============================================
    
    CLIENTE ||--o{ ENDERECO : "possui 1+"
    CLIENTE ||--o{ PEDIDO : "realiza 0+"
    CLIENTE ||--o| CARRINHO : "tem no máximo 1 ativo"
    
    EMPRESA_PARCEIRA ||--|| ENDERECO : "localizada em"
    EMPRESA_PARCEIRA ||--o{ PRODUTO : "oferece 0+"
    EMPRESA_PARCEIRA ||--o{ PEDIDO : "recebe 0+"
    
    CARRINHO ||--|{ ITEM_CARRINHO : "contém 1+"
    CARRINHO }o--o| CUPOM : "aplica 0 ou 1"
    ITEM_CARRINHO }o--|| PRODUTO : "referencia"
    
    PEDIDO ||--|{ ITEM_PEDIDO : "contém 1+"
    PEDIDO ||--|| PAGAMENTO : "possui"
    PEDIDO ||--o| ENTREGA : "gera 0 ou 1"
    PEDIDO }o--|| ENDERECO : "entrega em"
    ITEM_PEDIDO }o--|| PRODUTO : "referencia"
    
    ENTREGA }o--|| ENTREGADOR : "atribuída a"
    ENTREGA }o--|| ENDERECO : "origem"
    ENTREGA }o--|| ENDERECO : "destino"
    ENTREGA ||--o{ PONTO_RASTREAMENTO : "rastreada por 0+"
    ENTREGA ||--o{ AVALIACAO : "recebe 0 a 2"
    
    ENTREGADOR ||--|| PERFIL_GAMIFICACAO : "possui"
    ENTREGADOR }o--o{ BADGE : "desbloqueou 0+"
    
    RANKING_SEMANAL ||--|{ POSICAO_RANKING : "contém 1+"
    POSICAO_RANKING }o--|| ENTREGADOR : "referencia"
```

---

## 2.3 Decisões de Mapeamento OO → Relacional

### 2.3.1 Herança: `FormaPagamento` → Tabela Única com Discriminador

No diagrama de classes, `FormaPagamento` é abstrata com subclasses `CartaoCredito` e `PIX`.

**Estratégia escolhida**: **Single Table Inheritance** (tabela única com discriminador).

- Tabela: `PAGAMENTO`
- Coluna discriminadora: `forma_pagamento` (VARCHAR)
- Dados específicos: `dados_forma_pagamento` (JSON criptografado)

**Justificativa**: 
- Poucos tipos de pagamento (2-3)
- Queries frequentes juntam pedido + pagamento — JOIN único é mais eficiente
- Adicionar novo tipo (Boleto) requer apenas novo valor no enum, sem migração de schema

**Alternativa descartada**: Class Table Inheritance (tabela por subclasse) — geraria `CARTAO_CREDITO`, `PIX`, com FKs complexas e JOINs em todas as queries.

---

### 2.3.2 Padrão State: `EstadoEntrega` → Coluna VARCHAR

Estados (`AguardandoAceite`, `EmTransito`, etc.) **não são entidades** — são valores que mudam.

**Mapeamento**:
- Coluna: `ENTREGA.estado` (VARCHAR(30))
- Valores possíveis: controlados por constraint ou enum no banco

Classes State em memória reconstroem comportamento a partir do valor persistido.

---

### 2.3.3 Relacionamento N:N: `Entregador ↔ Badge` → Tabela Associativa

No diagrama de classes: `PerfilGamificacao *-- Badge` (composição). Na prática, um badge pode ser desbloqueado por múltiplos entregadores.

**Tabela associativa**: `ENTREGADOR_BADGE`
- Chave composta: `(entregador_id, badge_id)`
- Atributo adicional: `data_desbloqueio` (quando foi conquistado)

---

### 2.3.4 Atributos Calculados: Não Persistidos

Métodos do diagrama de classes que **não viram colunas**:

| Método (UML) | Motivo de não persistir |
|--------------|-------------------------|
| `Carrinho.calcularTotal()` | Derivado de `subtotal + taxa_entrega - desconto` |
| `Entregador.calcularBonificacao()` | Calculado em tempo real com multiplicadores da tabela |
| `Entrega.calcularTempoRestante()` | Derivado de `distancia / velocidade` |
| `Badge.verificarDesbloqueio()` | Lógica de negócio, não dado |

**Exceção**: `ENTREGADOR.avaliacao_media` e `total_entregas` são **cache desnormalizado** para performance. Atualizados via trigger ou job batch.

---

### 2.3.5 Value Objects: `Endereco` e `Localizacao`

`Endereco` poderia ser **embedded** (atributos dentro de CLIENTE/EMPRESA). Escolhemos **entidade separada** porque:
- Cliente tem **múltiplos endereços** (casa, trabalho)
- Queries de geolocalização (encontrar entregadores próximos) são frequentes — índice geoespacial em tabela dedicada é mais eficiente

`Localizacao` (lat/long) é **embedded** em:
- `ENDERECO` (coordenadas do endereço)
- `ENTREGADOR` (localização atual)
- `PONTO_RASTREAMENTO` (histórico de posições)

---

### 2.3.6 JSON para Flexibilidade: `atributos_extras` e `dados_forma_pagamento`

**`PRODUTO.atributos_extras`**: Produtos de categorias diferentes têm atributos específicos:
- Alimentos: validade, informações nutricionais
- Medicamentos: princípio ativo, receita necessária
- Livros: ISBN, autor

Modelar como colunas (`validade DATE`, `isbn VARCHAR`) geraria tabela esparsa. JSON permite flexibilidade sem migrations constantes.

**`PAGAMENTO.dados_forma_pagamento`**: Armazena de forma **criptografada**:
- Cartão: últimos 4 dígitos, bandeira, token do gateway
- PIX: chave, tipo de chave

---

## 2.4 Índices Essenciais (não mostrados no diagrama, mas críticos)

```sql
-- Performance de busca por geolocalização
CREATE INDEX idx_entregador_location ON ENTREGADOR USING GIST (
    ll_to_earth(latitude_atual, longitude_atual)
);

-- Query mais frequente: "pedidos de um cliente"
CREATE INDEX idx_pedido_cliente ON PEDIDO(cliente_id, data_criacao DESC);

-- Rastreamento em tempo real
CREATE INDEX idx_ponto_rastreamento_entrega ON PONTO_RASTREAMENTO(entrega_id, timestamp DESC);

-- Validação de cupom
CREATE INDEX idx_cupom_codigo_ativo ON CUPOM(codigo) WHERE ativo = true;

-- Ranking de entregadores
CREATE INDEX idx_perfil_pontos ON PERFIL_GAMIFICACAO(pontos_acumulados DESC);
```

---

## 2.5 Constraints de Negócio

### 2.5.1 Integridade Referencial com Deleção

```sql
-- Pedido deletado → itens devem ser deletados (CASCADE)
ALTER TABLE ITEM_PEDIDO 
ADD CONSTRAINT fk_item_pedido 
FOREIGN KEY (pedido_id) REFERENCES PEDIDO(id) ON DELETE CASCADE;

-- Entrega deletada → rastreamento deve ser deletado (CASCADE)
ALTER TABLE PONTO_RASTREAMENTO
ADD CONSTRAINT fk_rastreamento_entrega
FOREIGN KEY (entrega_id) REFERENCES ENTREGA(id) ON DELETE CASCADE;

-- Cliente deletado → endereços devem ser mantidos por LGPD (SET NULL e auditoria)
-- Implementação via soft delete (coluna 'ativo')
```

### 2.5.2 Regras de Domínio

```sql
-- Nota de avaliação entre 1 e 5
ALTER TABLE AVALIACAO 
ADD CONSTRAINT chk_nota CHECK (nota BETWEEN 1 AND 5);

-- Cupom não pode ter uso_atual > uso_maximo
ALTER TABLE CUPOM
ADD CONSTRAINT chk_cupom_uso CHECK (uso_atual <= uso_maximo);

-- Entrega só pode ter 1 avaliação de cada tipo
CREATE UNIQUE INDEX idx_avaliacao_unica 
ON AVALIACAO(entrega_id, tipo);

-- Status de pedido válido
ALTER TABLE PEDIDO
ADD CONSTRAINT chk_status_pedido 
CHECK (status IN ('AGUARDANDO_PAGAMENTO', 'PAGO', 'EM_PREPARACAO', 
                  'PRONTO_COLETA', 'EM_ENTREGA', 'ENTREGUE', 'CANCELADO'));
```

---

## 2.6 Divergências Justificadas em Relação ao Diagrama de Classes

| Aspecto | Diagrama de Classes | MER | Justificativa |
|---------|--------------------|----|---------------|
| **FormaPagamento** | Hierarquia (abstract + subclasses) | Tabela única `PAGAMENTO` com discriminador | Single Table Inheritance — melhor performance em queries |
| **EstadoEntrega** | Classes separadas (State pattern) | VARCHAR em `ENTREGA.estado` | Estados são valores, não entidades. Comportamento fica na camada de aplicação |
| **Cupom ↔ Carrinho** | Associação simples | FK nullable em `CARRINHO` | Cardinalidade 0..1 — cupom é opcional, FK é mais simples que tabela associativa |
| **Atributos calculados** | Métodos (ex: `calcularTotal()`) | Não existem no MER | Derivados, não persistidos. Cache seletivo (ex: `avaliacao_media`) é exceção documentada |
| **Localizacao** | Value Object separado | Embedded (lat/long em várias tabelas) | Sem identidade própria — duplicar colunas é mais eficiente que JOIN |
| **Badge** | Composição em `PerfilGamificacao` | Tabela associativa `ENTREGADOR_BADGE` | Na prática é N:N (mesma badge para vários entregadores) |

---

## 2.7 Estimativa de Volume (para dimensionamento futuro)

| Entidade | Linhas/dia (estimado) | Crescimento/ano | Observação |
|----------|----------------------|----------------|------------|
| PEDIDO | 10.000 | 3.6M | Particionamento por mês recomendado |
| PONTO_RASTREAMENTO | 500.000 | 180M | GPS a cada 30seg × entregas ativas. Archiving após 90 dias |
| AVALIACAO | 15.000 | 5.4M | ~75% das entregas são avaliadas |
| ENTREGADOR_BADGE | 50 | 18K | Crescimento lento (badges desbloqueados) |

---

## 2.8 Normalização

O modelo está em **3FN (Terceira Forma Normal)**:

- **1FN**: Todos os atributos são atômicos (sem arrays, exceto JSON justificado)
- **2FN**: Todos os atributos não-chave dependem da chave completa
- **3FN**: Nenhum atributo não-chave depende transitivamente de outro não-chave

**Desnormalizações intencionais**:
- `ENTREGADOR.avaliacao_media` e `total_entregas` — cache para evitar COUNT/AVG em toda query
- `ITEM_PEDIDO.preco_unitario` duplica `PRODUTO.preco` no momento da compra (preço histórico)

Ambas justificadas por **performance e auditoria** (preço não pode mudar retroativamente após venda).

---

**Próximo:** [`docs/03-comportamental-fatia1.md`](03-comportamental-fatia1.md)
