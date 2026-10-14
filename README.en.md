# advviral

[![advviral — Law Radar](guia/assets/banner-en.jpg)](https://inematds.github.io/advviral/guia/en/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## What it is

advviral is a system that automatically makes short videos (9:16) about what Brazil’s biggest YouTube lawyers have said in their videos about people’s rights: consumer rights, social security, labor, debt and banks, real estate, family, and scams. It watches YouTube videos through their transcripts, separates the facts, figures, and controversies, and puts together a video with large animated numbers, the source of each data point on screen, and commentary by Nei (avatar). Everything is reported and attributed; nothing becomes advice for your specific case, and proposed legislation is called a proposal. It does not replace guidance from a lawyer. It runs on a computer with a GPU, using Claude Code and HeyGen subscriptions. It is a sibling of [saudeviral](https://github.com/inematds/saudeviral) and [finviral](https://github.com/inematds/finviral) (same engine, different topic).

## 📖 User guide

Complete guide (landing page + step-by-step instructions): **https://inematds.github.io/advviral/guia/en/**

## How it works

1. **Collection:** lists videos from the channels in `canais.json` (metadata only, yt-dlp), downloads the audio, and transcribes it locally (inemavox, Whisper large-v3).
2. **Script:** `claude -p` (subscription, no API) reads the transcripts and writes three files:
   - `base.md`: each fact or figure, who said it, the video, and the verbatim excerpt;
   - `roteiro.json`: dialogue and visuals;
   - `legenda.txt`: the text for approval.
3. **Verification (`auto/checar.py`):** checks that each excerpt in `base.md` actually appears in the transcript. It also checks that the dialogue spells out numbers, that the images exist, that the CTA is present, and that “Não substitui orientação de um advogado” appears. If it fails, the script goes back for one correction. If it fails again, the video is not made.
4. **Avatar:** HeyGen generates Nei’s avatar through the studio (subscription, no generation API). If it fails, a version with Nei’s photo and voice (Chatterbox) is produced.
5. **Video:** `motor/palco.html` is rendered frame by frame with Playwright, including counters, a stamp, word-by-word captions, and a source in every scene. The avatar appears in the corner.
6. **Publishing:** the video is published on YouTube **INEMA TIA (lives2)** using `yt-pubx` (`auto/publicar.sh`) and is always sent to the v3 bot on Telegram, with the link. If the avatar fails, only the photo version is sent to the bot.
7. **Saturday and Sunday — the long 16:9 video:** after the short video, “A semana explicada” is produced (5 to 7 minutes, one chapter per influencer), with animated scenes from explicavideo v2 (`motor/longo.html`, `auto/roteiro-longo.md`, `auto/checar.py --longo`, `auto/longo.sh`).

## Schedule (systemd, user)

| When | Run | What it produces |
|---|---|---|
| Monday to Friday, 08:00 | `diario` | 40–60 s about what they published yesterday and the day before |
| Saturday, 08:00 | `semanal` | 90 s to 2 min: weekly recap + long 16:9 “A semana explicada” |
| Sunday, 08:00 | `domingo` | ~90 s: the week’s rights, changes, and pitfalls + long 16:9 |

The time is set to 08:00 to avoid competing for the GPU and HeyGen with finviral (02:00) and saudeviral (05:00). The v3 bot always receives one of three things: the video (with the lives2 link), “sem pauta,” or “falhou em X”.

```bash
cp systemd/advviral-* ~/.config/systemd/user/ && systemctl --user daemon-reload
systemctl --user enable --now advviral-diario.timer advviral-semanal.timer advviral-domingo.timer
systemctl --user list-timers 'advviral-*'
```

## One-off runs (outside the schedule)

```bash
auto/rodar.sh diario 2026-10-09                # run for a specific day
SEM_AVATAR=1 SEM_BOT=1 auto/rodar.sh semanal   # test: without HeyGen and without sending
echo "2026-10-17 diario" >> auto/pular.txt     # skip a run
```

## Influencers

The list is in `canais.json`; set `"ativo": false` to pause a channel. There is no official ranking of legal influencers: the list is based on topic searches and the measurements from 13/10/2026 (`pesquisa/influenciadores.md`). There are 10 today: Escobar Advogados and Ingrácio Advocacia (social security), Seus Direitos (consumer rights), Advogado Ricardo Reis and Dr. Rafaél Batista (debt and banks), Advogado William Carvalho and Magalhães & Moreno (labor), Seu Direito Online (real estate and taxes), Marcello Benevides (family, real estate, and business), and Dr. José Milagre (digital law and scams). Channels with classes for civil service exams and lawyers were left out; legal scholars focused on doctrine (Tartuce, Lenza, Streck) have small or inactive YouTube channels.

If another established and reliable lawyer or legal scholar appears prominently, the system suggests adding them in the bot message. Emerging names are never added, and additions are made only with Nei’s approval.

## Rules for each video

- Full frame 0: date in dd/mm/yyyy format and the strongest fact.
- Include only facts or figures found in the transcript, with the source (channel, date, title) on screen.
- Everything is reported and attributed (“according to him”). No “do X” advice for viewers’ cases: “cada caso é um caso: procure um advogado ou a defensoria pública”.
- Laws, deadlines, amounts, and STF decisions are presented only as stated; proposed legislation is called a proposal.
- Politics and criminal cases are attributed, without taking sides; no one is guilty before conviction. Office advertising is left out.
- “Não substitui orientação de um advogado.” in every video.
- Dialogue in Nei’s tone (“você”), with numbers and years spelled out. Ending: a comment hook and a CTA for AI agent management at inema.club.

## Requirements

- yt-dlp
- inemavox (`~/projetos/inemavox`)
- cardshorts (Chatterbox voice `nei`)
- Node with Playwright (`motor/node_modules` → link to the one in eventosextremospub)
- ffmpeg
- `claude` (Claude Code logged in with a subscription)
- Xvfb `:99` and the HeyGen profile for explicavideos
- v3 bot (`refazvideo/ferramentas/enviar_bot_v3.mjs`)

MIT License. Quoted content belongs to the source channels, which are always credited on screen.