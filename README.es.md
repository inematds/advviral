# advviral

[![advviral — Radar Derecho](guia/assets/banner-es.jpg)](https://inematds.github.io/advviral/guia/es/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## Qué es

advviral es un sistema que crea, de forma automática, vídeos cortos (9:16) sobre lo que dijeron en sus vídeos los abogados más importantes de YouTube en Brasil acerca de los derechos de las personas: consumidores, pensiones, trabajo, deudas y bancos, vivienda, familia y estafas. Revisa los vídeos de YouTube mediante transcripciones, separa los hechos, las cifras y las polémicas, y monta un vídeo con cifras grandes animadas, la fuente de cada dato en pantalla y Nei comentando (avatar). Todo se relata y se atribuye; nada se convierte en consejo para tu caso, y las propuestas de ley se identifican como propuestas. No sustituye la orientación de un abogado. Se ejecuta en un ordenador con GPU, con las suscripciones de Claude Code y HeyGen. Es el hermano de [saudeviral](https://github.com/inematds/saudeviral) y [finviral](https://github.com/inematds/finviral) (el mismo motor, otro tema).

## 📖 Guía de uso

Guía completa (landing + paso a paso): **https://inematds.github.io/advviral/guia/es/**

## Cómo funciona

1. **Recopilación:** lista los vídeos de los canales de `canais.json` (solo metadatos, yt-dlp), descarga el audio y lo transcribe localmente (inemavox, Whisper large-v3).
2. **Guion:** `claude -p` (suscripción, sin API) lee las transcripciones y guarda tres archivos:
   - `base.md`: cada hecho o cifra, quién lo dijo, el vídeo y el fragmento literal;
   - `roteiro.json`: diálogos y elementos visuales;
   - `legenda.txt`: el texto para aprobar.
3. **Verificación (`auto/checar.py`):** comprueba que cada fragmento de `base.md` aparezca realmente en la transcripción. También comprueba que las cifras del diálogo estén escritas con palabras, que las imágenes existan, que el CTA esté presente y que aparezca «No sustituye la orientación de un abogado». Si no aprueba, el guion vuelve una vez para corregirse. Si vuelve a fallar, no se produce el vídeo.
4. **Avatar:** HeyGen genera el avatar de Nei desde el estudio (suscripción, sin API de generación). Si falla, se crea la versión con la foto y la voz de Nei (Chatterbox).
5. **Vídeo:** `motor/palco.html` se renderiza cuadro a cuadro con Playwright, con contadores, sello, subtítulos palabra por palabra y la fuente en cada escena. El avatar aparece en una esquina.
6. **Publicación:** el vídeo se publica en YouTube **INEMA TIA (lives2)** mediante `yt-pubx` (`auto/publicar.sh`) y siempre se envía al bot v3 de Telegram con el enlace. Si falla el avatar, la versión con foto solo se envía al bot.
7. **Sábado y domingo — el vídeo largo 16:9:** después del corto, se publica «La semana explicada» (de 5 a 7 min, un capítulo por influenciador), con las escenas animadas de explicavideo v2 (`motor/longo.html`, `auto/roteiro-longo.md`, `auto/checar.py --longo`, `auto/longo.sh`).

## Agenda (systemd, usuario)

| Cuándo | Ronda | Qué se publica |
|---|---|---|
| De lunes a viernes, 08:00 | `diario` | 40–60 s con lo que publicaron ayer y anteayer |
| Sábado, 08:00 | `semanal` | 90 s a 2 min: resumen de la semana + vídeo largo 16:9 «La semana explicada» |
| Domingo, 08:00 | `domingo` | ~90 s: los derechos, los cambios y las trampas de la semana + vídeo largo 16:9 |

El horario es a las 08:00 para no competir por la GPU y HeyGen con finviral (02:00) y saudeviral (05:00). El bot v3 siempre recibe una de estas tres cosas: el vídeo (con el enlace de lives2), «sin tema» o «falló en X».

```bash
cp systemd/advviral-* ~/.config/systemd/user/ && systemctl --user daemon-reload
systemctl --user enable --now advviral-diario.timer advviral-semanal.timer advviral-domingo.timer
systemctl --user list-timers 'advviral-*'
```

## Ejecución puntual (fuera de la agenda)

```bash
auto/rodar.sh diario 2026-10-09                # ronda de un día específico
SEM_AVATAR=1 SEM_BOT=1 auto/rodar.sh semanal   # prueba: sin HeyGen y sin enviar
echo "2026-10-17 diario" >> auto/pular.txt     # omitir una ronda
```

## Influenciadores

La lista está en `canais.json`, con `"ativo": false` para pausarlos. No existe una clasificación oficial de influenciadores jurídicos: la base es la búsqueda por tema y la medición del 13/10/2026 (`pesquisa/influenciadores.md`). Actualmente son 10: Escobar Advogados e Ingrácio Advocacia (pensiones), Seus Direitos (consumidores), Advogado Ricardo Reis y Dr. Rafaél Batista (deudas y bancos), Advogado William Carvalho y Magalhães & Moreno (trabajo), Seu Direito Online (vivienda e impuestos), Marcello Benevides (familia, vivienda y empresas) y Dr. José Milagre (derecho digital y estafas). Se excluyeron los canales de clases para oposiciones y abogados; los juristas académicos (Tartuce, Lenza, Streck) tienen poca actividad o están inactivos en YouTube.

Si otro abogado o jurista consolidado y confiable aparece destacado, el sistema sugiere incluirlo en el mensaje del bot. Nunca se añade a figuras emergentes, y la inclusión solo ocurre con la aprobación de Nei.

## Reglas de cada vídeo

- Cuadro 0 completo: fecha dd/mm/aaaa y el hecho más contundente.
- Solo se incluye un hecho o una cifra que aparezca en la transcripción, con la fuente (canal, fecha, título) en pantalla.
- Todo se relata y se atribuye («según él»). Nada de «haz X» para el caso de quien mira: «cada caso es un caso: consulta a un abogado o a la defensoría pública».
- Leyes, plazos, importes y decisiones del STF, solo tal como se dijeron; las propuestas de ley se identifican como propuestas.
- Política y casos penales: atribuidos, sin tomar partido; nadie es culpable antes de una condena. Se excluye la publicidad del bufete.
- «No sustituye la orientación de un abogado» en todos los vídeos.
- Diálogo con el tono de Nei («tú»), con cifras y años escritos con palabras. Cierre: gancho para comentarios y CTA sobre gestión de agentes de IA en inema.club.

## Requisitos

- yt-dlp
- inemavox (`~/projetos/inemavox`)
- cardshorts (voz Chatterbox `nei`)
- Node con Playwright (`motor/node_modules` → enlace al de eventosextremospub)
- ffmpeg
- `claude` (Claude Code con sesión iniciada en la suscripción)
- Xvfb `:99` y el perfil de HeyGen de explicavideos
- bot v3 (`refazvideo/ferramentas/enviar_bot_v3.mjs`)

Licencia MIT. El contenido citado pertenece a los canales de origen, siempre acreditados en pantalla.