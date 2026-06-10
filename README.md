# Dashboard Lucralize · Vendas

Dashboard comercial da Lucralize integrado ao CRM Agendor. Arquivo único HTML/CSS/JS, hospedado via GitHub Pages.

---

## Acesso

| Ambiente | URL |
|---|---|
| Dashboard | https://lucralize-comercial.github.io/dashboard-agendor/ |
| Proxy Railway | https://agendo-proxy-production.up.railway.app |

---

## Arquitetura

```
GitHub Pages          Railway (Proxy Python)       Agendor API
index.html     →      agendo-proxy (app.py)   →    api.agendor.com.br/v3
               ←      /deals, /tasks          ←
```

O dashboard não chama a API do Agendor diretamente — todas as requisições passam pelo proxy Railway, que faz o cache dos dados e serve ao frontend.

### Repositórios

| Repositório | Conteúdo |
|---|---|
| `lucralize-comercial/dashboard-agendor` | Este arquivo (`index.html`) |
| `lucralize-comercial/agendo-proxy` | Proxy Python (`app.py`) |

---

## Proxy — Endpoints

| Método | Endpoint | Descrição |
|---|---|---|
| `GET` | `/` | Status do proxy e metadados do cache |
| `GET` | `/deals` | Cache de negócios do Agendor |
| `POST` | `/refresh` | Dispara atualização do cache de deals |
| `POST` | `/refresh-tasks` | Atualiza só o cache de tasks (sem refazer deals) |
| `GET` | `/tasks` | Cache de tarefas (últimos 90 dias) |
| `GET` | `/history-cache` | Histórico de movimentação de etapas (Funil Comercial, últimos 30 dias) |

**Observação:** O endpoint `/deals/{id}/history` da API v3 do Agendor retorna 404 — histórico de movimentação de etapas não está disponível na API pública. Os `events` no history-cache ficam sempre vazios. Consulta aberta ao suporte do Agendor sobre endpoint não documentado.

O cache de tasks é populado automaticamente ~10s após o fetch de deals terminar, e também roda a cada 2h de forma independente.

---

## Estrutura do Dashboard

### Abas

| Aba | Descrição |
|---|---|
| **Visão Geral** | Métricas consolidadas, evolução mensal, gráficos por etapa e origem |
| **Análise de Conversão** | Funil de etapas por canal, desempenho por origem e funil, cruzamento Origem × Responsável |
| **Acompanhamento** | Leads parados, pipeline por responsável, tempo médio |
| **Contratos Ganhos** | Listagem de negócios ganhos no período |
| **📋 Atividades** | Produtividade do time por tipo de atividade e status |
| **💰 Comissões** | Cálculo de comissões por responsável — acesso por senha |

### Filtro Global

A barra de filtros no topo é compartilhada por todas as abas:

- **Funil** — seleção múltipla com siglas (COM, PAT, LGL, etc.)
- **Período** — atalhos 7d / 30d / 90d / 1 ano / Todos + datas manuais
- **Análise de Conversão** — exibe também: Canal, Configurar canais, Adicionar/Remover canal
- **Atividades** — exibe também: Responsável, Status, Busca, Recarregar

---

## Aba Análise de Conversão

### Funil de etapas

Mostra os negócios **na etapa atual**, filtrados pelo período de **entrada** no funil (startTime ou createdAt). Cada etapa exibe:

- Número de negócios
- % de representatividade sobre o total do período
- Coluna Ganhos com % próprio
- Total de leads gerados no período (separado no final)

**Limitação conhecida:** não é possível saber onde um lead estava numa data específica no passado — a API v3 do Agendor não expõe histórico de movimentação.

### Canal

Configurado via modal "Configurar canais de origem". Associa cada origem de lead a um canal (ex: Meta Ads, Google, Indicação). Persistido em `localStorage`.

---

## Aba Atividades

Consome o endpoint `/tasks` do proxy. Estrutura real da API Agendor:

```json
{
  "assignedUsers": [{ "name": "Everton Silva" }],
  "user": { "name": "Everton Silva" },
  "text": "Mensagem enviada...",
  "type": "WhatsApp",
  "dueDate": "2026-06-10T...",
  "finishedAt": "2026-06-10T...",
  "deal": { "id": 123, "title": "Nome do negócio" }
}
```

Mapeamento de campos:

| Campo API | Uso |
|---|---|
| `assignedUsers[0].name` ou `user.name` | Responsável |
| `finishedAt !== null` | Concluída |
| `dueDate` | Prazo |
| `text` | Texto da atividade |
| `type` | Tipo (normalizado para pt-BR no proxy) |

### Seções da aba

1. **Cards de métricas** — Total · Concluídas · Pendentes · Taxa de conclusão · Atrasadas · Sem atividade
2. **Por responsável** — duas tabelas lado a lado:
   - Tipos (WhatsApp / Ligação / Reunião / Outros / Total) com % na segunda linha
   - Status (Concluídas → Pendentes → Atrasadas → Total) com % na segunda linha
3. **Negócios por tipo de última atividade** — cards + barras de distribuição
4. **Lista de atividades** — ordenada por pendentes primeiro, depois por prazo
5. **Negócios sem atividade** — negócios em andamento sem nenhuma task associada

---

## Aba Comissões

Protegida por senha. Senha armazenada em variável JS no `index.html` (`SENHA_COMISSOES`). Sessão expira em 2h.

### Regras por responsável

| Responsável | Regra |
|---|---|
| Ronaldo Junior | Tabela padrão — 100% próprios + 1/3 dos deals de Everton e Fabíola |
| Everton Silva | Tabela padrão — Ronaldo recebe 1/3 |
| Fabíola Carvalho | Tabela Fabíola (desconto sobre honorário) + R$30/reunião — Ronaldo recebe 1/3 |
| Brenda Medeiros | Regra por origem: 80% indicação colaborador · 15% demais · 0% diretoria |
| Diogo Vieira | Legado até 01/06/2026 — mesma regra da Brenda |
| Luiz Santos | Diretoria — sem comissão |

### Tabela padrão

| Condição Comercial | Taxa |
|---|---|
| Preço base | 30% |
| Indicação de Amigo | 80% |
| IRPF grátis primeiro ano | 28% |
| Isenção do primeiro mês | 25% |
| 15% de desconto no primeiro ano | 22% |
| Gratuidade dois meses | 20% |
| Parceria Leo Marconi | 20% |
| Gratuidade três meses | 15% |

### Tabela Fabíola

Fórmula: `Comissão = Valor × (1 − desconto) + R$30 × reuniões`

| Condição | Desconto | % Efetiva |
|---|---|---|
| Preço base | 0% | 100% |
| IRPF grátis | 15% | 85% |
| Isenção primeiro mês | 18% | 82% |
| Indicação de Amigo / Parceria | 20% | 80% |
| Gratuidade dois meses | 40% | 60% |
| 15% desconto primeiro ano | 40% | 60% |
| Gratuidade três meses | 50% | 50% |

**Ciclo de comissão:** dia 26 do mês anterior ao dia 25 do mês atual.

---

## Identidade Visual

Segue o manual da marca Lucralize:

| Token | Valor |
|---|---|
| Fundo da página | `#0E1428` |
| Superfície (cards) | `rgba(255,255,255,0.07)` |
| Teal principal | `#0C7488` |
| Teal claro | `#91DBE4` |
| Vermelho | `#F62642` |
| Fonte títulos | Comfortaa (bold) |
| Fonte corpo | Poppins |
| Fonte mono | DM Mono |

---

## Canais de Origem

Configurados via modal na aba Análise de Conversão. Persistidos em `localStorage`:

| Chave | Conteúdo |
|---|---|
| `lucralize_canais_lista` | Array com os nomes dos canais cadastrados |
| `lucralize_canal_map` | Objeto `{ "origem": "canal" }` |

---

## localStorage — outras chaves

| Chave | Conteúdo |
|---|---|
| `lucralize_bonus_reuniao` | Objeto `{ dealId: qtd }` — reuniões marcadas manualmente na Fabíola |

---

## Deploy

### Dashboard (index.html)

```bash
git add index.html
git commit -m "feat: descrição da alteração"
git push origin main
```

GitHub Pages publica automaticamente em ~1min.

### Proxy (app.py)

```bash
git add app.py
git commit -m "feat: descrição da alteração"
git push origin main
```

Railway faz o redeploy automaticamente ao detectar o push.

---

## Normalização de tipos de atividade (proxy)

O `app.py` normaliza o campo `type` das tasks para português antes de servir ao frontend:

| API Agendor | Normalizado |
|---|---|
| `whatsapp` | WhatsApp |
| `call` / `phone` | Ligação |
| `meeting` | Reunião |
| `email` | E-mail |
| `task` | Tarefa |
| Outros | mantém original |

---

## Dependências externas

| Biblioteca | Versão | Uso |
|---|---|---|
| Chart.js | 4.4.1 | Gráficos de linha e barra |
| Google Fonts | — | Comfortaa, Poppins, DM Mono |

Carregadas via CDN — sem build step, sem node_modules.
