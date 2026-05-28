# 📊 outbound-final-report

Skill para o Claude gerar relatórios profissionais de outbound B2B em `.docx` — modo mensal (clientes ativos) e modo encerramento (projetos finalizados).

---

## O que essa skill faz

- Lê o CSV de performance exportado do Snov.io (ou Instantly, Apollo, Lemlist)
- Lê o print da base completa de listas para mapear o ICP construído
- Lê o contexto do cliente via site, apresentação ou arquivo
- Calcula KPIs: entrega, abertura, resposta, bounce e engajamentos notáveis
- **Modo mensal:** realiza pesquisa de mercado via web search e gera insights acionáveis com fontes validadas
- **Modo encerramento:** gera aprendizados do projeto e recomendações de continuidade
- Entrega um `.docx` formatado com capa, tabelas, bloco de KPIs visuais e identidade visual profissional

---

## Inputs necessários

| Input | Formato | Obrigatório |
|---|---|---|
| Modo do relatório | mensal ou encerramento | Sim |
| Nome do cliente e responsável | texto | Sim |
| Período coberto | mês/ano ou intervalo | Sim |
| CSV de performance | `.csv` exportado da ferramenta | Sim |
| Print da base de listas | screenshot | Sim |
| Contexto do cliente | URL do site, apresentação ou arquivo `.pptx`/`.pdf` | Sim |
| Nome da agência elaboradora | texto | Sim |
| Dados de meses anteriores (acumulado) | texto ou planilha | Opcional |

---

## Ferramentas suportadas

- **Snov.io** (padrão — PT e EN)
- Instantly
- Apollo
- Lemlist

---

## Como instalar

1. Faça o download do arquivo `SKILL.md` deste repositório
2. No Claude, vá em **Settings > Skills**
3. Faça upload do arquivo
4. Pronto — na próxima conversa, o Claude reconhece automaticamente quando você precisa de um relatório de outbound

---

## Como usar

Basta iniciar uma conversa com algo como:

> "Preciso gerar o relatório mensal de outbound do cliente X"

> "Vamos fechar o relatório final do projeto Y"

O Claude vai solicitar os inputs necessários e gerar o `.docx` ao final.

---

## Estrutura do relatório

### Modo mensal
`Capa` → `Contexto do projeto` → `Base de listas e ICP` → `Performance do mês` → `Engajamentos notáveis` → `Insights e pontos de melhoria` → `Próximos passos`

### Modo encerramento
`Capa` → `Contexto do projeto` → `Listas e ICP` → `Performance geral` → `Engajamentos notáveis` → `Aprendizados` → `Entregas do projeto` → `Considerações finais`

---

## Desenvolvido por

[AN1](https://an1.com.br) — Marketing e Vendas B2B
