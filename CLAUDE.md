# mescla.ai — landing page

Site de uma página só. A fonte da verdade é **`index.html`** na branch `main`.

Repositório **público** — aqui não entra coordenada de infraestrutura, ID de
conta, IP ou chave. Isso vive em `neoari/mescla`, que é privado, junto com o resto
da aplicação.

## Como o site é publicado

```
index.html (GitHub: neoari/mescla-lp, main)
   ↓ wget no start do container
container nginx "mescla-lp" na VPS
   ↓
Traefik (TLS Let's Encrypt)
   ↓
proxy Cloudflare  →  https://mescla.ai
```

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

Alterou o `docker-compose.yml`? Aí `docker restart` não basta — ele não relê o
arquivo. Use `docker compose up -d --force-recreate`.

## Cuidado

A mesma VPS hospeda outros serviços não relacionados. Mexer em Traefik ou
reiniciar containers fora do `mescla-lp` afeta coisas que não têm a ver com o site.

A Cloudflare injeta ofuscação de e-mail no HTML servido — por isso o que o `curl`
devolve difere do arquivo local por algumas linhas. Isso é esperado, não é drift.

## Coordenadas

Ficam no `CLAUDE.md` de `neoari/mescla` (privado): IDs de VPS, IPs, conta e zona
Cloudflare, e o mapa das máquinas.
