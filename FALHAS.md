# FALHAS — advviral

| data | o que quebrou | menor correção | prompt \| infra |
|---|---|---|---|
| 2026-10-13 | avatar leu "consumidor ponto gov ponto bê erre" (o Nei: gov.br se diz "gov br") | regra no roteiro.md + checar.py reprova "ponto gov" na fala (nos três motores) | prompt |
| 2026-10-13 | piloto sem vídeo: perfil HeyGen ocupado por outro processo, o Submit não pegou e o ID salvo era do rascunho ("ID não existe"); a versão com foto caiu na voz do CTA | reenvio com REUSAR_ROTEIRO=1 (avatar ok); CTA sem dois-pontos ("…agentes de IA com o curso gratuito do inema ponto club"): o tts_direct partia no ":" e o pedaço curto saía embolado (83% × 96% depois) | infra + prompt |
