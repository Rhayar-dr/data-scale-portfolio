---
name: deploy
description: >-
  Publica/atualiza o site Data Scale ao vivo (datascaleconsultoria.trade e suas landing
  pages em public/). Use SEMPRE que o usuário pedir para "colocar no ar", "atualizar o
  site", "publicar", "fazer deploy", "subir as mudanças" ou alterar qualquer página em
  public/ que precise ir para produção. CRÍTICO: o site é servido por um Cloudflare
  Worker — o deploy correto é `npx wrangler deploy`. NUNCA use `wrangler pages deploy`,
  porque vai para o projeto errado e o domínio não muda.
---

# Deploy do site Data Scale

## Por que esta skill existe (leia antes de publicar)

O domínio de produção **datascaleconsultoria.trade** é servido por um **Cloudflare Worker**
estático (assets-only, serve `./public`, configurado em `wrangler.toml`, name
`data-scale-portfolio`). As rotas (o domínio) estão nesse Worker.

**A armadilha:** existe um projeto **Cloudflare Pages** com o MESMO nome
(`data-scale-portfolio` → `data-scale-portfolio.pages.dev`) e, às vezes, um cache
`.wrangler/cache/pages.json`. Eles **não** estão ligados ao domínio. Se você rodar
`wrangler pages deploy`, o comando "dá sucesso" mas o site real **não muda** — esse foi
exatamente o erro que esta skill existe para evitar.

**Regra de ouro:** para publicar, use `npx wrangler deploy` (Workers). Nunca `pages deploy`.

## Estrutura do site

- `public/index.html` — a **home** (servida na raiz do domínio).
- `public/<slug>/index.html` — cada landing page: `advocacia`, `auto-escola`,
  `imobiliarias`, `marketing`.
- Contatos padrão em todas as páginas: WhatsApp
  `https://api.whatsapp.com/send/?phone=%2B5547989171843&...` e Calendly
  `https://calendly.com/contato-datascaleconsultoria/30min`.

## Passo a passo

1. **Confirme que as mudanças estão em `public/`** (é o único diretório publicado).

2. **Deploy** — na raiz do repositório:
   ```bash
   npx wrangler deploy
   ```
   Na saída, confirme que apareceu `Uploaded data-scale-portfolio` e
   `Deployed data-scale-portfolio triggers`. Isso indica que o Worker e as rotas
   (incluindo o domínio) foram atualizados.

3. **Valide no ar** — o parâmetro `?cb=` fura o cache de borda, então mostra o conteúdo
   real recém-publicado:
   ```bash
   curl -sL "https://datascaleconsultoria.trade/?cb=$RANDOM" | grep -i -m1 '<title>'
   # as páginas internas devem continuar de pé:
   curl -s -o /dev/null -w "advocacia -> %{http_code}\n" https://datascaleconsultoria.trade/advocacia/
   ```
   Confira título e links esperados. Comandos que acessam a rede precisam rodar fora do
   sandbox.

4. **Commit (se o usuário quiser)** — para manter o repositório alinhado. O padrão do
   projeto é commitar direto na `main` e dar push; o deploy é independente do git, então
   isso é organização, não requisito:
   ```bash
   git add -A && git commit -m "<mensagem>" && git push origin main
   ```

## Se o domínio não atualizar

Quase sempre é porque o deploy foi para o lugar errado ou ficou em cache:

- Confirme que rodou `wrangler deploy` (Workers), **não** `pages deploy`.
- Refaça o teste com `?cb=$RANDOM` para furar o cache de borda.
- Ignore `data-scale-portfolio.pages.dev` e qualquer `.wrangler/cache/pages.json`:
  são irrelevantes para o domínio.
