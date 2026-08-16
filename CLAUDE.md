# mescla.ai — landing page

Site de uma página só. A fonte da verdade é **`index.html`** na branch `main`.

## Como o site é publicado

```
index.html (GitHub: neoari/mescla-lp, main)
   ↓ wget no start do container
container nginx "mescla-lp" na VPS Hostinger
   ↓
Traefik (TLS Let's Encrypt)
   ↓
proxy Cloudflare  →  https://mescla.ai
```

A stack fica em `/docker/mescla-lp/docker-compose.yml` na VPS.

## Deploy — os três passos são obrigatórios

O container baixa o `index.html` do GitHub **apenas quando inicia**. Um `git push`
sozinho não muda nada no ar.

```bash
git add index.html && git commit -m "..." && git push
ssh vps 'docker restart mescla-lp'
curl -sI https://mescla.ai | grep -i last-modified
```

O terceiro passo não é opcional. Se o `wget` falhar, o compose faz fallback para a
cópia anterior no volume: **o site continua no ar servindo a versão velha, sem erro
visível**. O `last-modified` é a única confirmação de que o deploy pegou.

Se o `last-modified` não avançou, veja o que aconteceu:

```bash
ssh vps 'docker logs --tail 30 mescla-lp'
```

## Coordenadas

- VPS Hostinger id `1543823` — `ssh vps` (root). Não use `BatchMode=yes`: a chave
  SSH vem do Keychain do macOS e o SSH falha nesse modo.
- Cloudflare: conta `77b77d3ccd5162d2cdb83cb6fbc9c2ab`, zona mescla.ai
  `23f9505e5eb2c02e1b50b75458d06170`. DNS: A `mescla.ai` → 187.127.4.212 (proxied),
  CNAME `www` → `mescla.ai`.

## Cuidado

A mesma VPS hospeda n8n, Evolution API, Langfuse, gbrain-pg e polinexus-ai. Mexer em
Traefik ou reiniciar containers fora do `mescla-lp` afeta serviços não relacionados.

A Cloudflare injeta ofuscação de e-mail no HTML servido — por isso o que o `curl`
devolve difere do arquivo local por algumas linhas. Isso é esperado, não é drift.
