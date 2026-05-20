# Agendador Instagram — Vallure Odontologia
## Documentação completa do sistema

---

## O que é esse sistema

Agendamento automático de carrosseis no Instagram da Vallure Odontologia, sem depender do Meta Business Suite e sem precisar do PC ligado na hora de postar.

**Fluxo resumido:**
1. Rodrigo cria os carrosseis (pasta Gerador Carrossel Vallure)
2. Pede pra Claude agendar ("quero agendar os posts")
3. Claude roda o `scheduler.py` no PC — faz upload das imagens e monta a fila no GitHub
4. GitHub Actions publica automaticamente nos horários certos, sem PC ligado

---

## Por que não usa o Meta diretamente para agendar

O Instagram tem uma funcionalidade `scheduled_publish_time` na API que permite agendar posts. Porém o Meta só libera isso para parceiros oficiais (Buffer, Later, Hootsuite). Qualquer outro app recebe erro `(#3) User must be on whitelist`. Não tem como desbloquear pelo painel — é uma whitelist controlada pelo Meta.

**Solução:** publicar imediatamente via API, mas no horário certo, usando o GitHub Actions como servidor sempre ligado.

---

## Estrutura de pastas

```
C:\Rodrigo\Claude AI\
├── Gerador Carrossel Vallure\
│   └── exports\                        ← pastas dos carrosseis gerados
│       ├── 2026-05-14-standard-light-protocolo\
│       │   ├── carousel\               ← slide_1.png, slide_2.png...
│       │   ├── captions\               ← legenda-carrossel.txt
│       │   └── _status.json            ← criado pelo scheduler (status local)
│       └── ...
│
└── Agendador Vallure\
    ├── scheduler.py                    ← script principal (roda no PC)
    ├── force_publish.py                ← publicação manual emergencial (bypassa janela de horário)
    ├── config.json                     ← credenciais e configurações
    ├── SISTEMA.md                      ← este arquivo
    ├── STATUS.html                     ← gerado pelo --status (calendário visual)
    ├── VER PLANO (dry-run).bat
    ├── VER STATUS.bat
    ├── AGENDAR POSTS.bat
    └── vallure-scheduler\              ← clone do repositório GitHub
        ├── publisher.py                ← script que roda no GitHub Actions (publicação)
        ├── check_alert.py              ← script que roda no GitHub Actions (alerta diário)
        ├── publish_queue.json          ← fila de posts (fonte da verdade)
        └── .github\workflows\
            ├── publish.yml             ← acionado pelo cron-job.org para publicar
            ├── check.yml               ← acionado pelo cron-job.org diariamente para alertar
            └── refresh-token.yml       ← renova o token toda segunda
```

---

## config.json

```json
{
  "ig_user_id":       "17841472334658749",
  "access_token":     "TOKEN_LONGO_AQUI",
  "app_id":           "899887016437089",
  "app_secret":       "8c35fdd17a77fece95d791a314b3c9cb",
  "imgbb_api_key":    "5a3959e15a783ad9a64e6a930ed4c2d0",
  "gmail_user":       "rodrigovincensi@gmail.com",
  "gmail_app_password": "pyiynuochumwfflj",
  "alert_email":      "rodrigovincensi@gmail.com",
  "github_token":     "TOKEN_GITHUB_AQUI",
  "github_repo":      "rodrigovincensi/vallure-scheduler"
}
```

**Importante:**
- `access_token` expira em 60 dias — o GitHub renova sozinho toda segunda via `refresh-token.yml`
- `github_token` é o Personal Access Token com escopos `repo` + `workflow` — não expira (configurado como "No expiration")
- Se trocar de PC, copiar este config.json completo

---

## Repositório GitHub

**URL:** https://github.com/rodrigovincensi/vallure-scheduler  
**Privado:** sim  
**Branch:** main

### Secrets configurados no repositório:
| Secret | Conteúdo |
|--------|----------|
| `IG_USER_ID` | `17841472334658749` |
| `IG_ACCESS_TOKEN` | Token longo do Instagram (60 dias, auto-renovado) |
| `IG_APP_ID` | `899887016437089` |
| `IG_APP_SECRET` | `8c35fdd17a77fece95d791a314b3c9cb` |
| `GH_PAT` | Personal Access Token do GitHub (usado pelo publisher, check_alert e refresh-token) |
| `GMAIL_USER` | `rodrigovincensi@gmail.com` (remetente dos alertas) |
| `GMAIL_PASS` | Senha de app do Gmail (não a senha normal — gerada em Segurança da conta Google) |
| `ALERT_EMAIL` | `rodrigovincensi@gmail.com` (destinatário dos alertas) |

---

## App Meta for Developers

**App:** Vallure Scheduler  
**App ID:** `899887016437089`  
**App Secret:** `8c35fdd17a77fece95d791a314b3c9cb`  
**Status:** Live (publicado)  
**Permissões:** `instagram_basic`, `instagram_content_publish`  
**URL política de privacidade:** `https://vallure.com.br` (campo obrigatório para publicar o app)

**Atenção:** O app precisa estar em modo **Live** (não Development). Em Development, a API retorna `(#3) User must be on whitelist` mesmo para o dono do app ao tentar criar containers de carrossel.

---

## Horários de publicação (SCHEDULE)

```
Segunda a Sexta:  09:00 light  |  18:30 premium
Sábado:           10:00 premium
Domingo:          19:00 light
```

Tipos de post detectados pelo nome da pasta:
- Contém `-light-` → tipo `light`
- Contém `-premium-` → tipo `premium`

---

## Workflows GitHub Actions

### publish.yml — publica os posts
Acionado por `workflow_dispatch` — **não usa schedule cron interno do GitHub**.  
O cron-job.org bate na porta do GitHub via API nos horários exatos.

**Por que não usa schedule cron do GitHub?**  
O cron interno pode atrasar horas em contas gratuitas. Um post das 09:00 chegou a rodar às 05:46 UTC — fora da janela, nada publicou. O `workflow_dispatch` via API externa é quase instantâneo (segundos).

O que faz: lê `publish_queue.json` do GitHub via API, verifica se há posts com `scheduled_for` dentro de ±15 minutos do horário atual, cria os containers no Instagram e publica. Atualiza o status no JSON para `published` via API (sem git push).

**Janela de tolerância:** 15 minutos antes e depois. Suficiente para absorver os segundos de atraso do Actions ao iniciar.

### check.yml — alerta de conteúdo baixo
Acionado por `workflow_dispatch` todo dia às 08:00 BRT pelo cron-job.org.  
Roda `check_alert.py`: verifica quantos dias de conteúdo pendente restam.  
Envia email via Gmail SMTP se faltar **≤ 3 dias** de conteúdo.  
Se tiver conteúdo suficiente, não faz nada.

### refresh-token.yml — renova o token
Acorda toda segunda-feira às 07:00 BRT (usa schedule cron — atraso não importa aqui).  
O que faz: chama a API do Meta para trocar o token atual por um novo (mais 60 dias), e atualiza o secret `IG_ACCESS_TOKEN` no GitHub via API. Completamente automático — nunca precisa renovar manualmente enquanto esse workflow rodar.

---

## cron-job.org — o relógio do sistema

**URL:** https://cron-job.org  
**Conta:** rodrigovincensi@gmail.com

Serviço gratuito que dispara requisições HTTP em horários exatos. Substitui o schedule cron interno do GitHub Actions, que é impreciso em contas gratuitas.

### 5 cronjobs configurados (todos em America/Sao_Paulo):

| Horário | Dias | Workflow disparado |
|---------|------|--------------------|
| 09:00 | Seg-Sex | publish.yml |
| 18:30 | Seg-Sex | publish.yml |
| 10:00 | Sábado | publish.yml |
| 19:00 | Domingo | publish.yml |
| 08:00 | Todo dia | check.yml |

Cada cronjob faz um POST para:
```
https://api.github.com/repos/rodrigovincensi/vallure-scheduler/actions/workflows/[ARQUIVO]/dispatches
```
Com header `Authorization: Bearer [GH_PAT]` e body `{"ref":"main"}`.

---

## publish_queue.json — estrutura

```json
[
  {
    "id":            "2026-05-14-standard-light-protocolo",
    "type":          "light",
    "scheduled_for": "2026-05-20T09:00:00",
    "imgbb_urls":    ["https://i.ibb.co/abc/slide_1.png", "..."],
    "caption":       "texto da legenda completa...",
    "status":        "pending",
    "created_at":    "2026-05-16T15:30:00"
  }
]
```

**Status possíveis:**
- `pending` → aguardando publicação
- `published` → publicado com sucesso (inclui `published_at` e `ig_post_id`)
- `failed` → falhou 3 vezes seguidas

---

## _status.json local (por pasta)

Criado pelo `scheduler.py` em cada pasta de carrossel após enfileirar:

```json
{
  "status":        "queued",
  "scheduled_for": "2026-05-20T09:00:00",
  "slides":        ["slide_1.png", "slide_2.png"],
  "processed_at":  "2026-05-16T15:30:00"
}
```

**Importante:** este arquivo local **não é atualizado** quando o post é publicado (o PC pode estar desligado). A fonte da verdade do status real é o `publish_queue.json` no GitHub. O `_status.json` local serve apenas para o scheduler saber que aquela pasta já foi enfileirada e não enfileirar de novo.

---

## Comandos do scheduler.py

| Comando | O que faz |
|---------|-----------|
| `python scheduler.py` | Agenda todos os posts pendentes |
| `python scheduler.py --dry-run` | Mostra o plano sem executar nada |
| `python scheduler.py --status` | Abre calendário visual no navegador |
| `python scheduler.py --check` | Verifica dias restantes e envia email se < 3 dias |
| `python scheduler.py --inicio DD/MM` | Força início a partir de uma data (opcional) |
| `python scheduler.py --sim` | Confirma automaticamente sem pedir s/n |

### Arquivos .bat disponíveis:
- `AGENDAR POSTS.bat`
- `VER PLANO (dry-run).bat`
- `VER STATUS.bat`
- `check_diario.bat`

---

## Fluxo quando Rodrigo traz conteúdo novo

1. Rodrigo diz: **"quero agendar os posts"** (ou variações)
2. Claude busca o `publish_queue.json` do GitHub e detecta automaticamente a data do último post agendado
3. Claude roda `--dry-run` e mostra a tabela com o cronograma completo:
   ```
   PASTA                     TIPO     SLIDES  SLOT
   2026-05-26-light-...      light    7       26/05 Ter 09:00
   2026-05-26-premium-...    premium  7       26/05 Ter 18:30
   ```
4. Rodrigo confirma
5. Claude roda com `--sim` — faz upload pro imgbb, adiciona na fila do GitHub
6. **Ao terminar, o calendário visual abre automaticamente no navegador**
7. Posts publicados automaticamente pelo cron-job.org + GitHub Actions nos horários certos

**Não precisa mais perguntar a última data** — o sistema lê do JSON.

---

## Frases que disparam ações automáticas

| O Rodrigo fala | Claude faz |
|----------------|------------|
| "quero agendar os posts" | Mostra o plano e agenda |
| "status" / "quero ver status" / "quero ver agendamentos" | Abre o calendário visual |

---

## Casos especiais e como resolver

### Post que deveria ter sido publicado mas não foi
Usar o `force_publish.py` — publica manualmente qualquer post da fila pelo prefixo da data, sem verificar janela de horário.  
Exemplo: `python force_publish.py 2026-05-20T09:00`

### Post deletado do Instagram após publicação
O status no JSON é `published` — não vai repostar automaticamente.  
Para repostar: Claude atualiza o status de volta para `pending` no `publish_queue.json` via GitHub API. O post será publicado no próximo horário disponível.

### Quer mudar slides de um post ainda pendente
As imagens já estão no imgbb com as URLs antigas na fila.  
Solução: Claude remove o post da fila, deleta o `_status.json` local da pasta, o usuário substitui os slides, e agenda de novo.

### Token do Instagram expira / dá erro de autenticação
O `refresh-token.yml` renova automaticamente toda segunda. Se por algum motivo falhar, o usuário gera um token curto no Graph API Explorer e Claude troca pelo longo e atualiza os secrets.

### Trocar de PC
1. Copiar a pasta `C:\Rodrigo\Claude AI\` completa para o novo PC
2. Instalar Python + `pip install requests PyNaCl`
3. O `config.json` tem todas as credenciais — basta ter ele na pasta certa
4. O GitHub continua publicando normalmente — independente do PC

### Primeira vez em uma nova sessão do Claude
Claude lê este arquivo (`SISTEMA.md`) e já sabe tudo o que precisa para operar o sistema corretamente.

---

## Lições aprendidas (problemas que já resolvemos)

| Problema | Causa | Solução |
|----------|-------|---------|
| `(#3) User must be on whitelist` | Meta bloqueia `scheduled_publish_time` para apps fora da whitelist | Publicar imediatamente via GitHub Actions no horário certo |
| Token expira em 2h | Token curto do Graph API Explorer | Trocar por token longo (60 dias) via `fb_exchange_token` |
| `Error validating client secret` | App Secret incorreto copiado | Copiar novamente clicando em "Mostrar" nas configurações do app |
| `400 Bad Request` sem mensagem | `raise_for_status()` antes de ler o JSON | Ler o JSON primeiro, extrair `error.message`, aí lançar exceção |
| Pasta EXPORTS errada | `ROOT / "exports"` em vez de `ROOT.parent / "Gerador Carrossel Vallure" / "exports"` | Corrigido no código |
| Erro Unicode no terminal | Caracteres especiais (─, →, ✅) | Substituídos por ASCII |
| App em modo Development | Forgot to switch to Live | Ir em Publicar no painel do Meta e publicar o app |
| GitHub Actions schedule atrasou horas | Schedule cron é impreciso em contas gratuitas — pode adiantar ou atrasar muito | Migrar para cron-job.org + workflow_dispatch (instantâneo) |
| Post não publicado (janela errada) | Schedule rodou às 05:46 UTC; post era às 12:00 UTC, fora da janela | Cron-job.org resolve — dispara no segundo exato |

---

## Serviços externos utilizados

| Serviço | Para quê | Gratuito |
|---------|----------|----------|
| imgbb.com | Hospedar imagens com URL pública (Meta não aceita arquivo local) | Sim |
| GitHub | Repositório + GitHub Actions (servidor 24h) | Sim |
| Meta Graph API | Publicar no Instagram | Sim |
| Gmail SMTP | Enviar alertas de conteúdo acabando | Sim |

---

*Atualizado em 20/05/2026 — Sistema em produção com cron-job.org + GitHub Actions.*
