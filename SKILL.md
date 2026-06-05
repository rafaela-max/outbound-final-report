---
name: outbound-final-report
description: >
  Gera relatórios profissionais de outbound B2B em .docx — tanto relatório mensal (para clientes
  ativos) quanto relatório de encerramento (para projetos finalizados). Use sempre que o usuário
  mencionar "relatório de outbound", "relatório mensal de campanha", "report mensal", "fechamento
  de projeto outbound", "relatório final de campanha", "entrega final para cliente",
  "resumo de performance outbound", ou quando precisar documentar resultados de cold email /
  prospecção ativa para um cliente, seja em ciclo mensal ou no encerramento do projeto.
  Aceita CSV do Snov.io (padrão) e de outras ferramentas (Instantly, Apollo, Lemlist).
  Sempre solicita: CSV de performance, print da base completa de listas e contexto do cliente
  (site, apresentação ou arquivo). No modo mensal, realiza pesquisa de mercado atualizada via
  web search para incluir insights e pontos de melhoria com embasamento externo validado.
  Entrega arquivo .docx formatado com identidade visual profissional.
---

# Relatório de Outbound B2B (Mensal ou Encerramento)

## Visão Geral

Gera um relatório .docx de outbound B2B para entrega ao cliente. Suporta dois modos:

- **Mensal** — para clientes ativos; cobre o mês corrente, inclui pesquisa de mercado atualizada
  e uma seção de insights e pontos de melhoria com embasamento externo
- **Encerramento** — para projetos finalizados; cobre todo o período com aprendizados e
  recomendações de continuidade

O modo é definido pelo usuário no início. Se não informado, pergunte antes de prosseguir.

---

## Inputs Necessários

Antes de começar, colete as seguintes informações. Parte pode já estar na conversa.
Se qualquer item obrigatório estiver faltando, solicite explicitamente antes de prosseguir.

**Obrigatórios:**
- **Modo do relatório**: mensal ou encerramento
- Nome do cliente
- Nome do dono / responsável comercial do cliente
- Período coberto (para mensal: mês/ano; para encerramento: mês/ano início - mês/ano fim)
- Nome do remetente das campanhas (quem aparecia como sender)
- Nome da agência elaboradora do relatório
- **CSV de performance geral** — exportado da ferramenta de outbound, contendo todas as campanhas do período
- **Print da base completa de listas** — screenshot da tela de listas da ferramenta, mostrando nome de cada lista e total de leads por lista
- **Contexto do cliente** — URL do site, URL de apresentação comercial (Google Slides, Canva, etc.) ou arquivo (.pptx, .pdf)

**Opcionais (enriquecem o relatório):**
- Informações qualitativas: destaques do período, observações, contexto de mercado
- Para relatório mensal: acumulado dos meses anteriores (para mostrar evolução)
- Ferramenta de outbound usada (padrão: Snov.io)

### Como processar cada input

**CSV de performance:**
Leia via pandas. Veja o Passo 1 para o código completo. Ferramenta padrão: Snov.io (PT).
Se for outra ferramenta, adapte os nomes de coluna conforme a tabela do Passo 1.

**Print da base de listas:**
Leia visualmente o screenshot para extrair nome de cada lista, total de leads e agrupamentos.
Monte a tabela de listas a partir disso — não use apenas o CSV, pois ele não reflete listas
criadas mas ainda não disparadas.

**Contexto do cliente (URL ou arquivo):**
- URL de site: use `web_fetch` na página principal
- URL de apresentação (Google Slides, Canva): use `web_fetch`; se inacessível, peça o arquivo
- Arquivo (.pptx, .pdf): leia com `extract-text` ou `pdftotext`

Extraia: o que a empresa faz, segmentos atendidos, diferenciais, produtos/serviços principais.
Use para redigir a seção de contexto com precisão — nunca invente informações sobre o cliente.

---

## Passo 1: Leitura e Análise do CSV

```python
import pandas as pd

df = pd.read_csv('/mnt/user-data/uploads/arquivo.csv')

# 1. Remover campanhas de teste
real = df[~df['Nome da campanha'].str.contains('TESTE|TEST|teste', na=False, case=False)]

# 2. Métricas gerais
metricas = ['Entregues','Devolvidos','Abertos','Cliques','Respostas','Respondido automaticamente']
totais = real[metricas].sum()

# 3. Por campanha
# ATENÇÃO: o CSV do Snov.io exporta múltiplas linhas por lead (uma por e-mail da sequência).
# Isso faz com que somar "Abertos" pelo CSV produza totais inflados.
# Regras:
# - "Abertos" e "Cliques" do CSV = válidos apenas como VOLUME BRUTO de interações. Use para engajamentos notáveis.
# - "Abertura %" = NUNCA calcule pelo CSV. Leia diretamente do print da tela da plataforma.
#   O Snov.io exibe a taxa correta (por leads únicos) na coluna "Aberturas do e-mail %" da tela de campanhas.
# - "Bounce %" = pode calcular pelo CSV (Devolvidos é registrado uma vez por lead).

por_campanha = real.groupby('Nome da campanha')[metricas].sum()
# Abertura% NÃO deve ser calculada aqui — preencher manualmente com os valores do print da tela:
# abertura_pct = {'Campanha X': 62.0, 'Campanha Y': 55.0}
por_campanha['Bounce%'] = (por_campanha['Devolvidos'] / (por_campanha['Entregues'] + por_campanha['Devolvidos']) * 100).round(1)

# 4. Cargos (decision makers)
print(real['Cargo'].value_counts().head(20))

# 5. Distribuição geográfica
real['Estado'] = real['Localização'].str.extract(r',\s([^,]+),\s*Brazil')
print(real['Estado'].value_counts().head(10))

# 6. Engajamentos notáveis
respondentes = real[real['Respostas'] > 0][['Nome do cliente potencial','Empresa','Cargo','Nome da campanha']]
auto_resp    = real[real['Respondido automaticamente'] > 0][['Nome do cliente potencial','Empresa','Cargo','Nome da campanha']]
cliques      = real[real['Cliques'] > 0][['Nome do cliente potencial','Empresa','Cargo','Nome da campanha','Cliques']]
```

**Nomes de colunas por ferramenta:**

| Ferramenta   | Coluna campanha    | Coluna abertos  | Coluna respostas | Coluna devolvidos | Coluna cliques |
|---|---|---|---|---|---|
| Snov.io (PT) | `Nome da campanha` | `Abertos`       | `Respostas`      | `Devolvidos`      | `Cliques`      |
| Snov.io (EN) | `Campaign name`    | `Opened`        | `Replied`        | `Bounced`         | `Clicked`      |
| Instantly    | `Campaign Name`    | `Emails Opened` | `Replies`        | `Bounced`         | `Link Clicks`  |
| Apollo       | `Sequence Name`    | `Opens`         | `Replies`        | `Bounces`         | `Clicks`       |
| Lemlist      | `campaignName`     | `openCount`     | `replyCount`     | `bounceCount`     | `clickCount`   |

Se a ferramenta não for Snov.io, confirme com o usuário os nomes de coluna antes de rodar.

---

## Passo 2: Calcular KPIs de Destaque

Calcule e guarde para usar no relatório:

- **Total de listas criadas** e **total de leads mapeados** (do print)
- **Total de contatos alcançados** (excluindo testes, do CSV)
- **E-mails entregues** + **taxa de entrega** (entregues / total)
- **Total de aberturas brutas** (do CSV, válido como volume) + **taxa de abertura %** (lida do print da tela da plataforma — nunca calculada pelo CSV)
- **Respostas diretas** + **taxa de resposta**
- **Cliques** e **auto-respostas**
- **Bounce rate** (devolvidos / total enviado)

Para **relatório mensal**: se houver dados de meses anteriores, inclua comparativo mês a mês
nos KPIs principais (ex: "↑ 12% vs. mês anterior").

**Benchmark de mercado (incluir no relatório):**
- Taxa de resposta B2B no Brasil: 1% a 5% para listas bem segmentadas
- Taxa de abertura saudável em cold email B2B: 30-65% (calculada pela plataforma sobre leads únicos, conforme exibido na tela do Snov.io)
- Bounce rate aceitável: abaixo de 5%

---

## Passo 3 (apenas modo mensal): Pesquisa de Mercado e Geração de Insights

Este passo é exclusivo do modo mensal. Execute antes de gerar o .docx.

### 3.1 Identificar o setor e os temas relevantes

A partir do contexto do cliente (site/apresentação), identifique:
- Setor principal de atuação
- Segmentos prospectados nas campanhas do mês
- Dores e objeções típicas do ICP

### 3.2 Realizar pesquisas com web_search

Execute ao menos 4 buscas combinando o setor do cliente com temas de outbound e mercado.
Priorize fontes com dados recentes (últimos 6 meses) e reconhecidas no mercado B2B.

**Buscas obrigatórias:**
```
"cold email B2B benchmark [ano atual] reply rate"
"outbound B2B [setor do cliente] tendências [ano atual]"
"[setor do cliente] Brasil mercado [ano atual]"
"cold email best practices [ano atual] B2B"
```

**Buscas complementares conforme contexto:**
```
"[setor do cliente] dores compra decisão B2B"
"personalização cold email conversão [ano atual]"
"[segmento prospectado] tendências mercado Brasil"
"follow-up sequência email B2B taxa resposta"
```

**Fontes prioritárias para validar dados:**
- Relatórios de ferramentas: HubSpot, Salesloft, Outreach, Woodpecker, Lemlist, Apollo
- Publicações setoriais do setor do cliente
- IBGE, CNI, ABRAMAT, CBIC ou associações do setor (para contexto de mercado Brasil)
- LinkedIn Sales Solutions reports

### 3.3 Estruturar os insights

Com base nos dados do CSV + pesquisa de mercado, monte de 4 a 6 insights para o relatório.
Cada insight deve seguir o formato:

**[Título curto e direto]**
Observação baseada nos dados do cliente → contexto de mercado validado → ação recomendada

Classifique cada insight em uma das categorias:
- **Melhoria de copy** — ajustes em assunto, abertura, CTA ou sequência
- **Otimização de lista** — segmento a expandir, praça a explorar, cargo a adicionar
- **Timing e cadência** — melhor horário, frequência, intervalo entre e-mails
- **Estratégia de canal** — LinkedIn paralelo, ligação, WhatsApp como complemento
- **Contexto de mercado** — dado externo que justifica ou desafia a abordagem atual

**Regras para os insights:**
- Todo dado de mercado citado deve ter fonte identificada (nome do relatório ou publicação)
- Não inclua insight sem embasamento — se não encontrou dado confiável, omita ou indique como hipótese
- Priorize insights acionáveis no próximo ciclo, não observações genéricas
- Se o setor do cliente tiver sazonalidade relevante (ex: construção civil no 2º semestre), mencione

### 3.4 Exemplo de insight bem formatado

**Aumentar volume nas campanhas de engenharia civil no RS e SC**
As campanhas dessas praças tiveram taxa de abertura acima de 150% — acima da média das demais.
Segundo relatório da Apollo (2024), segmentos com abertura >120% em cold email têm 2,3x mais
chance de converter quando abordados com follow-up via LinkedIn na mesma semana.
Recomendação: dobrar o volume de contatos nessas praças no próximo ciclo e ativar LinkedIn
em paralelo para os leads que abriram 2 ou mais e-mails sem responder.

---

## Passo 4: Estrutura do Relatório por Modo

### Modo Mensal — 7 seções

1. **Capa** — "Relatório Mensal de Outbound — [Mês/Ano]", cliente, elaborador
2. **Contexto do Projeto** — breve descrição do cliente + objetivo da operação outbound
3. **Base de Listas e ICP** — tabela de listas ativas no mês, total de leads mapeados
4. **Performance do Mês** — KPIs visuais do período + tabela por campanha + nota de benchmark
5. **Engajamentos Notáveis** — respondentes, cliques, auto-respostas com empresa e cargo
6. **Insights e Pontos de Melhoria** — 4 a 6 insights com embasamento de mercado (ver Passo 3)
7. **Próximos Passos** — ações concretas planejadas para o próximo ciclo, baseadas nos insights

### Modo Encerramento — 8 seções

1. **Capa** — "Relatório Final de Outbound — [Período]", cliente, elaborador
2. **Contexto do Projeto** — descrição do cliente + objetivos + escopo do que foi feito
3. **Criação de Listas e Segmentação de ICP** — tabela completa de listas, cobertura geográfica, perfis de cargo
4. **Performance Geral das Campanhas** — KPIs visuais do período total + tabela por campanha
5. **Engajamentos Notáveis** — respondentes, cliques, auto-respostas com empresa e cargo
6. **Aprendizados e Observações** — o que funcionou, pontos de atenção, benchmark de mercado
7. **Entregas do Projeto** — lista do que foi criado e entregue durante o projeto
8. **Considerações Finais** — encerramento com recomendações de continuidade

---

## Passo 5: Identidade Visual e Componentes

Use `docx` (npm). Consulte `/mnt/skills/public/docx/SKILL.md` para referência de API.

```javascript
const PRIMARY   = "1B3A5C";  // Azul escuro — títulos, capa
const ACCENT    = "2E6DA4";  // Azul médio — subtítulos, bordas
const LIGHT_BG  = "E8F0F7";  // Fundo claro — células KPI
const HEADER_BG = "1B3A5C";  // Cabeçalho de tabela
const ALT_ROW   = "F0F5FA";  // Linha alternada em tabelas
const GRAY      = "666666";  // Textos secundários
const GREEN     = "217346";  // Variações positivas
const ORANGE    = "C55A11";  // Alertas / pontos de atenção
```

**Bloco de KPIs visuais** — tabela 4 colunas, cada célula:
valor grande (bold, 40pt, PRIMARY) + label (18pt, GRAY) + sub opcional (16pt, GREEN)

**Tabela de listas** — colunas: Lista | Segmento | Leads (fonte: print)

**Tabela de campanhas** — colunas: Campanha | Contatos | Entregues | Abertos | Abertura% | Bounce% | Respostas

**Tabela de engajamentos** — colunas: Tipo | Contato / Empresa | Cargo | Campanha
Tipos: "Resposta direta", "Auto-resposta", "Clique em link"

**Bloco de insights (modo mensal)** — cada insight em caixa destacada:
título em bold (PRIMARY) + corpo em texto normal + tag de categoria em GRAY itálico + fonte da referência

**Nota obrigatória no rodapé da seção de performance:**
> "O total de aberturas exibido (volume bruto) é superior ao número de leads porque cada e-mail
> da sequência gera um novo registro de abertura pelo mesmo contato. A taxa de abertura %
> exibida é a calculada pela plataforma sobre leads únicos e deve ser lida diretamente da tela."

---

## Passo 6: Salvar e Entregar

```javascript
Packer.toBuffer(doc).then(buffer => {
  const modo = 'Mensal'; // ou 'Final'
  const nomeArquivo = `${nomeCliente}_Relatorio_${modo}_Outbound.docx`.replace(/\s+/g, '_');
  fs.writeFileSync(`/mnt/user-data/outputs/${nomeArquivo}`, buffer);
});
```

Use `present_files` para entregar o arquivo ao usuário.

---

## Checklist Final

- [ ] Modo correto aplicado (mensal vs. encerramento)
- [ ] Contexto do cliente escrito com base no site/apresentação real (sem informações inventadas)
- [ ] Tabela de listas construída a partir do print (não apenas do CSV)
- [ ] Total de listas e leads mapeados corretos e destacados nos KPIs
- [ ] Nome do cliente correto em toda a extensão do documento
- [ ] Nome do responsável comercial do cliente correto
- [ ] Nome da agência elaboradora correto
- [ ] Campanhas de teste excluídas das métricas
- [ ] Nota explicativa sobre taxa de abertura >100% incluída
- [ ] Para mensal: seção de insights com ao menos 4 itens, todos com fonte identificada
- [ ] Para mensal: seção de próximos passos baseada nos insights
- [ ] Para encerramento: seção de recomendações de continuidade presente
- [ ] Arquivo nomeado com nome do cliente e modo (Mensal ou Final)

---

## Recomendações de Continuidade (apenas modo encerramento)

Inclua ao menos 4 recomendações baseadas nos dados. Padrões a identificar:

- Segmento com maior taxa de clique → reativar com volume maior
- Segmento iniciado tarde ou com baixo volume → expandir
- Leads com múltiplas aberturas sem resposta → abordar via LinkedIn em paralelo
- CTA com atrito alto → testar alternativa de menor comprometimento
- Ausência de CRM integrado → recomendar integração para rastrear follow-up pós-abertura
