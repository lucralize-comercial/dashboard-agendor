# Dashboard Lucralize · CRM (Agendor)

Dashboard comercial interno da Lucralize Contabilidade, integrado ao Agendor CRM via proxy Railway.

**URL de produção:** [lucralize-comercial.github.io/dashboard-agendor](https://lucralize-comercial.github.io/dashboard-agendor/)

---

## Stack

| Camada | Tecnologia |
|--------|-----------|
| Frontend | HTML + CSS + JS puro (arquivo único `index.html`) |
| Gráficos | Chart.js 4.4.1 via CDN |
| Fontes | Comfortaa, Poppins, DM Mono (Google Fonts) |
| Hospedagem | GitHub Pages — repositório `lucralize-comercial/dashboard-agendor` |
| Proxy API | Railway — `agendo-proxy-production.up.railway.app` |
| Proxy código | `ronaldocassimiro/agendo-proxy` (`app.py`) |

---

## Proxy Railway

O proxy centraliza o acesso à API do Agendor e mantém um cache em memória dos negócios.

| Endpoint | Método | Descrição |
|----------|--------|-----------|
| `/` | GET | Status do proxy: negócios em cache e timestamp |
| `/deals` | GET | Retorna todos os negócios com `customFields` e produtos |
| `/refresh` | POST | Força atualização do cache |

**Cache:** atualizado automaticamente a cada 1 hora. Os negócios são buscados com `order_by: updatedAt, order_dir: desc`, 100 por página.

**Campo de data de ganho:** `endTime` (preenchido manualmente no Agendor) como principal, `wonAt` como fallback.

---

## Abas do dashboard

### 1. Visão Geral
- Métricas do período: total de negócios, valor total, taxa de conversão, ticket médio, em andamento, ganhos, perdidos
- Gráfico de evolução mensal (últimos 12 meses)
- Leads por semana e por mês por origem
- Gráfico por etapa do funil e por origem
- Motivos de perda, responsáveis e lista de negócios

### 2. Análise de Conversão
- **Funil de conversão por etapa** — select de funil (dados do cache), filtro de canal, filtro de data por início do negócio, % de avanço e quantidade que avançou entre etapas
- Cards: origens únicas, melhor conversão, maior volume, maior perda
- Tabelas: desempenho por origem e por funil (total, média/dia, ganhos, perdidos, % conversão, % perda, ticket médio, valor ganho)
- Gráficos: top origens por volume e por conversão
- Cruzamento Origem × Responsável
- Responsável por conversão e principais motivos de perda

### 3. Acompanhamento
- Cards no topo: leads parados, valor em pipeline, tempo médio ganho, tempo médio perda
- Leads parados em andamento (padrão: sem atualização há mais de **7 dias**, configurável)
- Pipeline por responsável
- Tempo médio por funil

### 4. Contratos Ganhos
- Tabela de negócios ganhos no período com produto, valor, responsável e data

### 5. 💰 Comissões
- Protegido por senha: `GestaoComercial` (expira em 2h)
- Ciclo: dia 26 do mês anterior ao dia 25 do mês atual

---

## Regras de Comissão

### Tabela padrão

| Condição Comercial | % Comissão |
|--------------------|-----------|
| Preço base | 30% |
| Indicação de Amigo | 80% |
| IRPF grátis primeiro ano | 28% |
| Isenção primeiro mês | 25% |
| 15% desconto | 22% |
| Gratuidade dois meses | 20% |
| Parceria Leo Marconi | 20% |
| Gratuidade três meses | 15% |

### Tabela Fabíola Carvalho

A comissão é calculada como `Valor × (1 − desconto) + bônus reunião`.

| Condição Comercial | Desconto | % Efetiva |
|--------------------|----------|-----------|
| Preço base | 0% | 100% |
| IRPF grátis no primeiro ano | 15% | 85% |
| Isenção do primeiro mês | 18% | 82% |
| Parceria Afiliados | 20% | 80% |
| Indicação de Amigo | 20% | 80% |
| Parceria Leo Marconi | 20% | 80% |
| Gratuidade dois meses | 40% | 60% |
| 15% de desconto no primeiro ano | 40% | 60% |
| Gratuidade de três meses | 50% | 50% |

**Bônus reunião:** R$ 30,00 por reunião agendada (marcado manualmente via checkbox no detalhamento).

### Regras por responsável

| Responsável | Regra | Ronaldo recebe |
|-------------|-------|----------------|
| Ronaldo Junior | Tabela padrão | 100% para si |
| Everton Silva | Tabela padrão | 1/3 do total |
| Fabíola Carvalho | Tabela própria + bônus reunião | 1/3 do total (comissão + reunião) |
| Brenda Medeiros | 80% indicação colaborador · 15% demais · 0% diretoria | — |
| Diogo Vieira | Igual Brenda (legado) | — |
| Matheus Augusto | Antes de 01/06/2026: exibe como Diogo Vieira e segue regra do Diogo. Após: não aparece | — |
| Luiz Santos | Diretoria, sem comissão | — |

---

## Fechamento Fabíola

Seção abaixo do detalhamento, visível quando há negócios da Fabíola no período. Colunas: Data, Negócio, Produto, Condição Comercial, Origem, Valor, Desconto, Reunião, Comissão, % Com. Ronaldo, Com. Ronaldo, Data Ganho, Reg. Ganho. Filtros por produto e condição. Botão Exportar Excel.

---

## Filtros de Canal (Funil de Etapas)

Sistema de mapeamento Origem → Canal, salvo em `localStorage`.

- **Adic/Remov canais:** gerencia a lista de canais disponíveis (campo editável + 🗑 para remover)
- **Configurar canais:** associa cada origem a um canal via select
- **Select "Todos os canais":** filtra as etapas do funil pelo canal selecionado
- O mapeamento persiste entre sessões via `localStorage` (chaves: `lucralize_canal_map`, `lucralize_canais_lista`)

---

## localStorage

| Chave | Conteúdo |
|-------|----------|
| `lucralize_bonus_reuniao` | `{ [dealId]: quantidade }` — bônus de reunião da Fabíola |
| `lucralize_canal_map` | `{ [origem]: canal }` — mapeamento origem → canal |
| `lucralize_canais_lista` | `["Meta Ads", "Orgânico", ...]` — lista de canais cadastrados |

---

## Pendências abertas

- Bônus Operacional — regras não definidas
- Saídas por Insatisfação — regras não definidas
- Comissão Legalização — regras não definidas
- Venda de Automações — regras não definidas
- Parceiros — regras não definidas
- Bônus Reunião Fabíola: integração com Agendor pendente (por ora campo manual via checkbox)
- Negócios sem `condicao_comercial` no cache aparecem como pendentes — resolvem quando o proxy atualiza

---

## Deploy

O dashboard é um arquivo único sem build. Para publicar:

1. Edite o `index.html`
2. Faça upload direto no repositório `lucralize-comercial/dashboard-agendor` via GitHub (Add file → Upload files)
3. O GitHub Pages publica automaticamente em ~1 minuto

> **Atenção:** não use copiar/colar do HTML — sempre faça upload do arquivo para evitar corrupção de caracteres especiais.
