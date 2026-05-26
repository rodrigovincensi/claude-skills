# Projeto: Vallure Odontologia

Site da clínica Vallure Odontologia (Panambi/RS). Contém uma página institucional (`index.html`) e uma landing page de implantes (`implantes.html`).

---

## 📁 Estrutura de arquivos

```
C:\Rodrigo\Vallure Odontologia\
├── index.html              # Site principal (institucional)
├── implantes.html          # Landing page de implantes
├── vallure-site.md         # ESTE arquivo (contexto do projeto)
├── fotos/                  # Imagens e vídeos do site
│   ├── favicon.ico, favicon.png, favicon-96.png
│   ├── og-preview.jpg      # Imagem do preview (WhatsApp/redes sociais)
│   ├── antes-depois-caso-1.jpeg ... caso-12.jpeg
│   ├── foto-vanessa.jpeg   # Foto profissional da dentista
│   ├── implante-animacao.mp4 + implante-animacao-poster.jpg
│   ├── video-eolanda.mp4   # Vídeo curto Eolanda (seção Resultados)
│   ├── depoimento-eolanda-comprimido.mp4 + depoimento-eolanda-poster.jpg
│   └── ... outras fotos
└── Logomarca/
    ├── logo-completa.png   # Logo principal usada no site
    ├── Logo Bege.png
    ├── Logo VA Feed.png
    └── ...
```

---

## 🚀 Deploy

- **Hospedagem:** Cloudflare Pages
- **Domínio:** vallureodontologia.com.br
- **ZIP final:** `C:\Rodrigo\vallure-site.zip`
- **Processo:** gerar ZIP → upload manual no Cloudflare Pages

### ⚠️ Regras CRÍTICAS do ZIP

1. **Forward-slash (/) nos caminhos**, NÃO backslash (\\). Cloudflare roda Linux e não encontra arquivos com backslash.
   - ❌ NÃO usar `Compress-Archive` (gera backslash)
   - ✅ Usar `System.IO.Compression.ZipFile` (PowerShell)

2. **Excluir do ZIP:**
   - Arquivos `.mov`
   - `depoimento-eolanda.mp4` (original, 36.5MB — passa do limite do Cloudflare de 25MB por arquivo)
   - Manter apenas `depoimento-eolanda-comprimido.mp4` (24MB)

3. **Limite Cloudflare:** 25MB por arquivo individual no ZIP.

### Script padrão pra gerar ZIP

```powershell
Add-Type -Assembly 'System.IO.Compression.FileSystem'
$zipPath = 'C:\Rodrigo\vallure-site.zip'
$sourceDir = 'C:\Rodrigo\Vallure Odontologia'
if (Test-Path $zipPath) { Remove-Item $zipPath }
$zip = [System.IO.Compression.ZipFile]::Open($zipPath, 'Create')
foreach ($name in @('index.html','implantes.html')) {
    [System.IO.Compression.ZipFileExtensions]::CreateEntryFromFile($zip, "$sourceDir\$name", $name) | Out-Null
}
Get-ChildItem "$sourceDir\fotos" -File | Where-Object {
    $_.Extension -ne '.mov' -and $_.Name -ne 'depoimento-eolanda.mp4'
} | ForEach-Object {
    [System.IO.Compression.ZipFileExtensions]::CreateEntryFromFile($zip, $_.FullName, "fotos/$($_.Name)") | Out-Null
}
Get-ChildItem "$sourceDir\Logomarca" -File | ForEach-Object {
    [System.IO.Compression.ZipFileExtensions]::CreateEntryFromFile($zip, $_.FullName, "Logomarca/$($_.Name)") | Out-Null
}
$zip.Dispose()
```

ZIP final fica em ~47-48MB.

---

## 🎨 Identidade visual

### Cores (CSS variables)
- `--gold: #bba87b` — bege/dourado, cor de destaque
- `--gold-light` — variação mais clara
- `--dark: #1a1a1a` — preto principal
- `--black: #0d0d0d` — preto mais escuro
- `--cream: #faf7f2` — fundo creme das seções claras
- `--white: #ffffff`
- `--text-light` — texto secundário

### Tipografia
- **Brand (títulos):** Cinzel (serif elegante)
- **Body:** Inter ou similar (sans-serif)

### Estilo geral
- Premium, escuro, alto padrão
- Fundo predominantemente escuro (`--dark`) com seções creme (`--cream`)
- Detalhes em dourado (`--gold`)
- Animações suaves com `cubic-bezier(.4,0,.2,1)`
- Bordas arredondadas com `--radius`
- Box-shadows profundos em cards (`0 24px 64px rgba(0,0,0,.55)`)

---

## 🏥 Informações da clínica

- **Nome:** Vallure Odontologia
- **Endereço:** Rua Sete de Setembro, 477 — Centro, Panambi/RS
- **WhatsApp:** (55) 9 9927-1163 (link: `https://wa.me/5555999271163`)
- **Email:** vallureodontologia@gmail.com
- **Dentista responsável:** Dra. Vanessa Cardoso dos Santos
  - Cirurgiã-Dentista
  - Especialista em Implantodontia
  - CRO/RS 29210

### Horário de atendimento
- Segunda a Sexta: 09h00 – 18h30
- Sábado: 09h00 – 12h00

---

## 🛠️ Integrações instaladas

### Google Tag Manager
- **Container ID:** `GTM-MHQWTNL3`
- Instalado em ambos HTMLs:
  - Script no `<head>` (top of file)
  - `<noscript>` logo após `<body>`
- Gerenciamento centralizado pelo gestor de tráfego (Gabriel Bandeira GM Clínicas)
- Pixel/Analytics/Ads são adicionados via painel GTM, sem mexer no código

### Open Graph (preview redes sociais)
- **Index:** título "Vallure Odontologia - Um tratamento à altura do seu valor."
- **Implantes:** título "Implantes — Vallure Odontologia", descrição cortada em "natural"
- Imagem compartilhada: `fotos/og-preview.jpg`

### Favicon
- Gerado a partir de `fotos/logo-cortada.png` (o "VA" bege)
- Fundo transparente, preenche 100% do espaço
- 3 tamanhos: 16, 32, 96px + `.ico`

---

## 📋 Restrições e regras de conteúdo

### ❌ NÃO mencionar/prometer
- "Tomografia 3D inclusa" — não temos
- Garantias específicas que não foram acordadas
- Números inventados (ex: "+500 pacientes" se não tiver fonte)

### ✅ Pode/deve dizer
- "+200 sorrisos transformados" (validado)
- "Avaliação 100% gratuita"
- "Sem compromisso"
- "Tomografia 3D" e "guia cirúrgico digital" — pode citar como método/recurso, sem prometer inclusão gratuita

### Formas de pagamento
- Cartão e boleto em até 12x
- Pix e dinheiro à vista

### Texto padrão dos botões de agendamento
- Todos dizem **"Agendar Avaliação"** (não "Agendar Consulta")

---

## 🧩 Estrutura específica das páginas

### index.html (site principal)
- Header fixo com nav: Sobre · Serviços · Diferenciais · Depoimentos · Antes & Depois · Contato · `Implantes ↗` (chip dourado linkando pra landing)
- Mobile menu com ham → X
- Hero com foto de fundo + tag dourada
- Seção "Sobre a clínica"
- "Nossos Serviços" (6 cards com modais)
- "Por que nos escolher" — 3 diferenciais numerados (01, 02, 03 em dourado)
- "Depoimentos" — 8 avaliações REAIS do Google (5★) em slider
- "Antes & Depois" — 9 casos em slider (caso-1 a caso-12, ordem específica)
- "Contato" + mapa
- Footer com "Desenvolvido por [Rodrigo Vincensi](https://www.linkedin.com/in/rodrigo-vincensi/)" em dourado

### implantes.html (landing page)
- Header fixo: O que é · Por que escolher · Como funciona · A especialista · Resultados · Depoimentos · Dúvidas · `← Site principal` (chip dourado)
- Hero com vídeo de fundo + headline "Recupere seu sorriso com **implantes dentários**" + faixa de trust (3 chips animados subindo)
- "O que é" — texto + foto + botão "Tirar dúvidas agora"
- "Por que escolher" — 6 benefícios
- "Como funciona" — 4 passos
- **"A especialista"** — seção com foto da Dra. Vanessa + credenciais
- "Resultados" — 3 casos antes/depois COM vídeo Eolanda sticky no desktop (right column)
- "Depoimento" — vídeo da Eolanda (depoimento-eolanda-comprimido.mp4) com poster
- "FAQ" — 7 perguntas (inclui Quanto custa, Posso parcelar, Sem osso)
- "CTA Final" — fundo dourado
- Footer minimal com nav + endereço + "← Voltar ao site principal" centralizado

### Avaliações reais (Google) — ordem no slider
1. Ionara Gavião
2. Carlos André Fagundes
3. Eolanda Zimmermann
4. Maiara Thaís Tolfo
5. Sara Martins
6. Simone de Moura
7. Daniele Oliveira
8. Aline Dorneles

### Antes & Depois — ordem dos casos (index)
1. antes-depois-caso-1.jpeg (Reabilitação estética com facetas)
2. antes-depois-caso-11.jpeg (Protocolo sobre implantes)
3. antes-depois-caso-12.jpeg (Reabilitação com implantes)
4. antes-depois-caso-2.jpeg (Prótese total)
5. antes-depois-caso-4.jpeg (Implante e harmonização)
6. antes-depois-caso-5.jpeg (Lentes de contato)
7. antes-depois-caso-8.jpeg (Reabilitação oral completa)
8. antes-depois-caso-9.jpeg (Prótese sobre implante)
9. antes-depois-caso-3.jpeg (Clareamento — por último)

### Antes & Depois implantes.html (3 casos focados)
1. antes-depois-caso-11.jpeg (Protocolo sobre implantes)
2. antes-depois-caso-4.jpeg (Implante e harmonização)
3. antes-depois-caso-9.jpeg (Prótese sobre implante)

---

## ⚙️ Detalhes técnicos importantes

### Mobile menu (implantes.html)
- Hamburger vira X no header (não duplica)
- Logo no header tem `onclick="closeLpMenu(); window.scrollTo({top:0})"` — fecha menu e vai pro topo
- `padding: 80px 0 40px` no menu — para itens não ficarem colados no topo
- Tap effect: `.tapped { color: var(--gold) }` (só cor, sem fundo)
- Chips "← Site principal" (implantes) e "Implantes ↗" (index) com borda dourada e mesma fonte/CAPS

### Overflow horizontal
- `html { overflow-x: clip }` e `body { overflow-x: clip }` — `clip` em vez de `hidden` porque `hidden` quebra `position: sticky`

### Sticky video (Resultados implantes)
- Layout flex: `display: flex; align-items: stretch`
- Coluna esquerda: imagens dos casos empilhadas
- Coluna direita: `position: sticky; top: 110px` — vídeo Eolanda acompanha scroll dos 3 casos
- Vídeo do hero do implantes tem fix JS para forçar autoplay em browsers que bloqueiam

### Animações scroll-reveal
- `.reveal`, `.reveal-left`, `.reveal-right` com IntersectionObserver
- Faixa de trust no hero implantes: `animation-delay` em cascata (0.5s, 0.68s, 0.86s)

### Lightbox antes/depois
- Clicar em qualquer card abre fullscreen
- ESC ou clique fora fecha
- Cursor `zoom-in` no hover, `zoom-out` no overlay

---

## 🔍 Outros pontos importantes

- **Header transparente no topo** → ganha fundo escuro com blur ao rolar 60px
- **Nav scroll-spy**: link da seção atual ganha pill dourado (fundo bege, texto escuro)
- **WhatsApp flutuante** pulsante em verde no canto inferior direito
- **Tap targets** de no mínimo 44px no mobile
- **Imagens com `loading="lazy"`** para performance

---

## 📞 Contatos do projeto

- **Cliente:** Vallure Odontologia
- **Desenvolvedor:** Rodrigo Vincensi ([LinkedIn](https://www.linkedin.com/in/rodrigo-vincensi/))
- **Gestor de tráfego:** Gabriel Bandeira (GM Clínicas)

---

## 🎯 Como pedir ajustes em conversa nova

Manda algo tipo:

> *"Lê o `C:\Rodrigo\Vallure Odontologia\vallure-site.md` pra entender o contexto. Preciso ajustar [X] no [arquivo Y]."*

Em 30s eu sei tudo: estrutura, paleta, restrições, processo de deploy. Daí é só executar.
