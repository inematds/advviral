# advviral

[![advviral — Radar Direito](guia/assets/banner.jpg)](https://inematds.github.io/advviral/guia/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## O que é

O advviral é um sistema que faz, sozinho, vídeos curtos (9:16) sobre o que os maiores advogados do YouTube no Brasil disseram nos vídeos deles sobre os direitos das pessoas: consumidor, previdência, trabalho, dívidas e bancos, imóvel, família e golpes. Ele assiste aos vídeos do YouTube por transcrição, separa os fatos, os números e as polêmicas, e monta um vídeo com números grandes animados, a fonte de cada dado na tela e o Nei comentando (avatar). Tudo é relatado e atribuído, nada vira conselho para o seu caso e proposta de lei é chamada de proposta. Não substitui orientação de um advogado. Roda num computador com GPU, com a assinatura do Claude Code e do HeyGen. É o irmão do [saudeviral](https://github.com/inematds/saudeviral) e do [finviral](https://github.com/inematds/finviral) (mesmo motor, outro tema).

## 📖 Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/advviral/guia/**

## Como funciona

1. **Coleta:** lista os vídeos dos canais de `canais.json` (só metadados, yt-dlp), baixa o áudio e transcreve localmente (inemavox, Whisper large-v3).
2. **Roteiro:** o `claude -p` (assinatura, sem API) lê as transcrições e grava três arquivos:
   - `base.md`: cada fato ou número, quem disse, o vídeo e o trecho literal;
   - `roteiro.json`: falas e visuais;
   - `legenda.txt`: o texto para aprovação.
3. **Conferência (`auto/checar.py`):** confere se cada trecho do `base.md` está de fato na transcrição. Também confere se os números da fala estão por extenso, se as imagens existem, se o CTA está presente e se aparece "Não substitui orientação de um advogado". Se reprovar, o roteiro volta uma vez para correção. Se reprovar de novo, o vídeo não é feito.
4. **Avatar:** o HeyGen gera o avatar do Nei pelo estúdio (assinatura, sem API de geração). Se falhar, sai a versão com a foto e a voz do Nei (Chatterbox).
5. **Vídeo:** o `motor/palco.html` é renderizado quadro a quadro pelo Playwright, com contadores, carimbo, legenda palavra a palavra e fonte em toda cena. O avatar fica no canto.
6. **Publicação:** o vídeo é publicado no YouTube **Inema Pro (lives12)** pelo `yt-pubx` (`auto/publicar.sh`) e vai sempre ao bot v3 no Telegram, com o link. Se o avatar falhar, a versão com foto só vai ao bot.
7. **Sábado e domingo — o longo 16:9:** depois do curto, sai "A semana explicada" (5 a 7 min, um capítulo por influenciador), com as cenas animadas do explicavideo v2 (`motor/longo.html`, `auto/roteiro-longo.md`, `auto/checar.py --longo`, `auto/longo.sh`).

## Agenda (systemd, usuário)

| Quando | Rodada | O que sai |
|---|---|---|
| Segunda a sexta, 08:00 | `diario` | 40–60 s com o que eles publicaram ontem e anteontem |
| Sábado, 08:00 | `semanal` | 90 s a 2 min: balanço da semana + longo 16:9 "A semana explicada" |
| Domingo, 08:00 | `domingo` | ~90 s: os direitos, as mudanças e as armadilhas da semana + longo 16:9 |

O horário é 08:00 para não disputar a GPU e o HeyGen com o finviral (02:00) e o saudeviral (05:00). O bot v3 sempre recebe uma de três coisas: o vídeo (com o link do lives12), "sem pauta" ou "falhou em X".

```bash
cp systemd/advviral-* ~/.config/systemd/user/ && systemctl --user daemon-reload
systemctl --user enable --now advviral-diario.timer advviral-semanal.timer advviral-domingo.timer
systemctl --user list-timers 'advviral-*'
```

## Avulso (fora da agenda)

```bash
auto/rodar.sh diario 2026-10-09                # rodada de um dia específico
SEM_AVATAR=1 SEM_BOT=1 auto/rodar.sh semanal   # teste: sem HeyGen e sem enviar
echo "2026-10-17 diario" >> auto/pular.txt     # pular uma rodada
```

## Influenciadores

A lista fica em `canais.json`, com `"ativo": false` para pausar. Não há ranking oficial de influenciadores jurídicos: a base é a busca por tema e a medição de 13/10/2026 (`pesquisa/influenciadores.md`). Hoje são 10: Escobar Advogados e Ingrácio Advocacia (previdência), Seus Direitos (consumidor), Advogado Ricardo Reis e Dr. Rafaél Batista (dívidas e bancos), Advogado William Carvalho e Magalhães & Moreno (trabalho), Seu Direito Online (imóvel e tributo), Marcello Benevides (família, imóvel e empresa) e Dr. José Milagre (direito digital e golpes). Canais de aula para concurso e para advogados ficaram de fora; os juristas de doutrina (Tartuce, Lenza, Streck) estão pequenos ou parados no YouTube.

Se aparecer em destaque outro advogado ou jurista consolidado e confiável, o sistema sugere a inclusão na mensagem do bot. Nome emergente nunca entra, e a inclusão só acontece com o ok do Nei.

## Regras de cada vídeo

- Quadro 0 cheio: data dd/mm/aaaa e o fato mais forte.
- Só entra fato ou número que está na transcrição, com a fonte (canal, data, título) na tela.
- Tudo relatado e atribuído ("segundo ele"). Nada de "faça X" para o caso de quem assiste: "cada caso é um caso: procure um advogado ou a defensoria pública".
- Lei, prazo, valor e decisão do STF só como foram ditos; proposta de lei é chamada de proposta.
- Política e caso criminal: atribuídos, sem lado, ninguém é culpado antes de condenação. Propaganda do escritório fica de fora.
- "Não substitui orientação de um advogado." em todo vídeo.
- Fala no tom do Nei ("você"), com números e anos por extenso. Fim: gancho de comentário e CTA de gestão de agentes de IA no inema.club.

## Requisitos

- yt-dlp
- inemavox (`~/projetos/inemavox`)
- cardshorts (voz Chatterbox `nei`)
- Node com Playwright (`motor/node_modules` → link para o do eventosextremospub)
- ffmpeg
- `claude` (Claude Code logado na assinatura)
- Xvfb `:99` e o perfil do HeyGen do explicavideos
- bot v3 (`refazvideo/ferramentas/enviar_bot_v3.mjs`)

Licença MIT. O conteúdo citado pertence aos canais de origem, sempre creditados na tela.
