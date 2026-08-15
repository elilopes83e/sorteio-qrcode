# Sorteio ao vivo (missão revelada no celular sorteado)

Recria a demo clássica de palestra: a plateia escaneia um QR Code, entra numa
página web pelo celular, e quando o apresentador clica em "Sortear", o
celular sorteado recebe, em tempo real, uma missão aleatória apresentada com
uma animação estilizada (roleta de nome, cartão com versículo, atmosfera de
brasas — visual inspirado em `erjmissao.html`).

100% front-end, sem build, hospedável no GitHub Pages. A sincronização em
tempo real entre os celulares usa o [Supabase Realtime](https://supabase.com)
(Presence + Broadcast), que é gratuito e não exige nenhuma tabela no banco.

## Como funciona

- `host.html` — página do apresentador (telão). Mostra o QR Code, a
  contagem de celulares conectados, quantos já foram sorteados, e os botões
  **Sortear** / **Reset Geral**.
- `index.html` — página do participante (celular). Ao abrir, entra na "sala"
  via Supabase Presence e fica aguardando. Quando é sorteado, recebe um
  broadcast e revela uma missão aleatória (nome, contexto bíblico, tarefa e
  versículo), com a animação de roleta + brasas.
- `missoes.js` — lista de missões e versículos reaproveitada de
  `erjmissao.html` (fácil de editar/trocar por outro conteúdo).
- `config.js` — guarda a URL e a chave pública (`anon key`) do seu projeto
  Supabase.

### Sorteio sem repetição

Cada pessoa sorteada é adicionada a uma lista de "já sorteados" (guardada na
aba do apresentador, dura enquanto a página do host ficar aberta) e não pode
ser sorteada de novo. O botão **Reset Geral** limpa essa lista e devolve
todos os celulares à tela de espera — a partir daí, todo mundo volta a valer
para um novo sorteio, incluindo quem já tinha sido sorteado antes.

## Configuração

1. Crie uma conta gratuita em [supabase.com](https://supabase.com) e crie um
   novo projeto.
2. Em **Project Settings > API**, copie a **Project URL** e a
   **anon public key**.
3. Edite o arquivo [config.js](config.js) e substitua os placeholders:

   ```js
   window.APP_CONFIG = {
     SUPABASE_URL: "https://SEU-PROJETO.supabase.co",
     SUPABASE_ANON_KEY: "SUA-ANON-KEY-PUBLICA",
   };
   ```

   > A anon key é feita para ser usada no client (navegador) — não tem
   > problema ela ficar pública no repositório/Pages. Este app não lê nem
   > escreve em nenhuma tabela do banco, só usa os canais Realtime
   > (Presence/Broadcast), então não é preciso criar tabelas nem ativar
   > replication.

4. Não é necessário nenhum outro passo no painel do Supabase — Realtime
   Presence/Broadcast já vem habilitado por padrão em projetos novos.

## Testar localmente

```bash
npx serve .
# ou
python -m http.server 8000
```

Abra `host.html` numa aba e `index.html` em outras (ou pelo celular, usando
o IP da sua máquina na rede local). A contagem de conectados no host deve
subir conforme as abas/celulares entram. Clique em **Sortear** e confirme
que só um deles revela a missão; sorteie de novo e confirme que só quem
ainda não saiu pode ser sorteado. **Reset Geral** volta tudo ao normal e
libera todo mundo para um novo sorteio.

> Se for testar com `npx serve .`: essa ferramenta redireciona
> `/index.html` para `/` e descarta o `?room=` da URL nesse redirect. Isso
> não acontece no GitHub Pages (onde o app realmente vai rodar), mas se for
> testar localmente com várias salas, acesse a raiz (`/?room=...`) direto
> em vez de `/index.html?room=...`.

## Publicar no GitHub Pages

1. Suba este diretório para um repositório no GitHub.
2. Em **Settings > Pages**, selecione a branch principal (`main`) e a pasta
   raiz (`/`) como fonte.
3. Aguarde o deploy e acesse a URL gerada
   (`https://SEU-USUARIO.github.io/SEU-REPO/host.html`) no telão.
4. O QR Code em `host.html` já aponta automaticamente para `index.html` no
   mesmo domínio — funciona sem nenhuma configuração extra.

### Várias sessões/demos no mesmo projeto Supabase

Use o parâmetro `?room=` na URL do host para isolar salas diferentes, ex.:
`host.html?room=palestra-sp`. O QR Code gerado já propaga o mesmo `room`
para `index.html` automaticamente.
