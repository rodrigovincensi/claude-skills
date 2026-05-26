# Skill: compactar-conversa

## Objetivo
Resumir a conversa atual para fácil continuidade em um novo chat.

## Quando usar
Ativar com: `/compactar-conversa`

## Comportamento ao ativar

1. Leia toda a conversa do contexto atual
2. Extraia apenas o que é crítico:
   - Objetivo principal do projeto
   - Decisões tomadas (e por quê)
   - Problemas encontrados e soluções aplicadas
   - Trechos de código importantes (cole o código real)
   - Próximo passo pendente (se houver)
3. Formate em 5 a 7 bullet points
4. Coloque tudo dentro de um bloco de código para fácil cópia

## Formato de saída

```
## Contexto para novo chat

- **Projeto:** [nome e objetivo em 1 linha]
- **Decisão:** [decisão crítica tomada + motivo]
- **Código:** `[trecho relevante ou arquivo:linha]`
- **Problema resolvido:** [problema → solução aplicada]
- **Estado atual:** [o que está funcionando agora]
- **Pendente:** [próximo passo ou bloqueio]
```

## Regras
- Nada além do bloco formatado
- Sem introdução, sem resumo após o bloco
- Se não houver código relevante, omita esse bullet
- Máximo 7 bullets — corte o menos crítico se passar
