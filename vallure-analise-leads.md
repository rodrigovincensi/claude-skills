---
name: vallure-analise-leads
description: >
  Atualiza o dashboard HTML de análise de leads da Vallure Odontologia quando o usuário
  disser que atualizou os dados, pedir para gerar o dashboard, atualizar o relatório da
  agência ou qualquer variação disso. NUNCA gera o HTML do zero — sempre preserva o design
  existente do dashboard.html e atualiza apenas os números.
---

# Dashboard de Análise de Leads — Vallure Odontologia

## Fluxo correto

1. **Rodar o script de extração** para coletar os números atualizados das planilhas
2. **Ler o output** e identificar o que mudou em relação ao dashboard atual
3. **Atualizar apenas os dados** no `dashboard.html` existente — nunca reescrever o HTML inteiro

## Passo 1 — Extrair os dados

```
python "C:\Rodrigo\Vallure Odontologia\Análise Leads\calcular.py"
```

O script lê as duas planilhas e imprime todos os números necessários de forma organizada.

## Passo 2 — Atualizar o dashboard.html

Com os números em mãos, use a ferramenta Edit para atualizar **somente** os valores que mudaram no arquivo:

```
C:\Rodrigo\Vallure Odontologia\Análise Leads\dashboard.html
```

Os pontos que podem mudar a cada atualização são:

- **KPIs** no topo: total leads, fechamentos, custo, orçamento
- **Arrays JS** no `<script>`: `leadsTotal`, `leadsAnuncio`, `semanasLabels`, `semanasQtd`, `receitaAnuncio`, `custoMes`
- **Tabela financeira**: linhas e totais
- **Texto do banner** de alerta e da conclusão (se os números mudaram significativamente)
- **Pie legend** manual: percentuais e contagens por origem
- **Data** no footer

## Regras importantes

- NUNCA usar o `gerar_dashboard.py` — ele sobrescreve o HTML e destrói o design
- NUNCA reescrever o dashboard.html inteiro — só editar o que mudou
- Sempre conferir se os totais batem (ex: soma dos custos mensais = custo total)
- Após editar, confirmar ao usuário quais números foram atualizados
