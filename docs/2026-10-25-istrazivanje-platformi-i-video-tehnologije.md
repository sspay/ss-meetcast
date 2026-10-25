# Istraživanje: postojeće katoličke molitvene platforme i open source video

Datum: 2026-10-25. Izvor: web pretraga (uglavnom sažeci). Brojke korisnika i
termini događaja **nisu provjereni** na službenim stranicama, a neki su iz
2020.–2024.

## Ideja

Globalna društvena mreža koja okuplja N ljudi oko M molitvenih nakana. Uz svaku
nakanu postoji redoviti virtualni događaj, na primjer krunica za mir u svijetu
svake srijede u 21:00, na koji ljudi dolaze u video ili audio sobu i mole zajedno.

## Zaključak

Nijedan pronađeni projekt ne spaja **nakanu, ponavljajući termin, sobu uživo i
zajednicu koja se vraća**. Postojeći projekti pokrivaju jedan dio:

```mermaid
flowchart LR
  N[Nakana] --> T[Ponavljajući termin] --> S[Soba uživo] --> Z[Zajednica koja se vraća]
  Hozana -.-> N
  Hozana -.-> Z
  Rosario -.-> N
  Hallow -.-> Z
  Prayminder -.-> T
  ZoomYT[Zoom/YouTube/FB krunice] -.-> S
  ZoomYT -.-> T
```

### Mreže oko nakana, bez videa

- **Hozana** (FR, laici, katolička i ekumenska): oko 1 M članova, preko 1.000
  zajednica (župe, biskupije, redovi, pojedinci). Članovi dobivaju objave
  zajednice, posjetitelji ostavljaju nakane (2021.: oko 700 nakana dnevno).
  Moderacija je stroga: politika i komercijala su zabranjeni. Nema susreta uživo.
  Konceptualno je najbliža ideji. <https://hozana.org/en/page/about>
- **Rosario** (tim Hozane): digitalna „živa krunica" (Pauline Jaricot). Grupe
  od 5, svatko dnevno moli deseticu, otajstva se dodjeljuju automatski, grupa se
  veže uz nakanu. Moli se asinkrono. <https://apps.apple.com/app/id1619096568>
- **Hallow**: najveća katolička aplikacija (oko $70 godišnje). Župne zajednice,
  nakane do 240 znakova s gumbom „Pray for All", sezonski izazovi (Pray25,
  Pray40). Sadržaj je snimljen, nema soba uživo.
- **Prayminder** (SE, Prayminder AB, dvoje ljudi): osobni popis nakana s
  podsjetnicima (dnevno, tjedno, točan termin). Besplatna, bez reklama, a
  molitve ostaju na uređaju. Navode 27–30 M izmoljenih molitava. Nema zajedničkih
  popisa ni sinkronizacije. Za ideju je zanimljiv njihov tok podsjetnika: kod
  nas bi podsjetnik vodio ravno u zajednički termin. **Ime „Prayminder" je
  zauzeto.** Postoji i zasebna „Prayer Minder" drugog autora.
  <https://apps.apple.com/us/app/prayminder/id1309466339>
- **Click To Pray** (Papina svjetska molitvena mreža): mjesečne papine nakane i
  eRosary narukvica. Moli se pojedinačno. Mogući partner ili izvor nakana.

### Krunice uživo, ali na tuđim alatima

Stotine pojedinačnih inicijativa: Paradisus Dei (Zoom petkom, link mailom),
St. Therese (Zoom, pon–pet 21:00 EST), Catholic Online (YouTube srijedom, nakane
se čitaju naglas), EWTN, župe sa Zoom krunicom na prijavu, povremene vatikanske
svjetske krunice za mir.

Zajednički problemi: nema kataloga termina, linkovi se dijele mailom, nema veze
s nakanom ni kontinuiteta, vremenske zone svaki organizator rješava sam.

### Mali pokušaji iste ideje

Orabimus (krunica u pregledniku i soba uživo, izgleda kao projekt jednog
developera), RosaryOn (iOS, tjedne krunice posvećene zemljama). Kod protestanata:
Church App i 3-16 Prayer Network (audio sobe), Pray.com (feed nakana). Nijedan
od katoličkih pokušaja nije dosegao vidljivu veličinu.

### Hrvatska

Nije pronađena online platforma. Postoje javne krunice na trgovima (prva subota)
i žive krunice po župama.

## Izbor video tehnologije

**Pravi P2P (mesh) ne dolazi u obzir.** Radi do otprilike 4–6 sudionika, jer
svatko šalje stream svakome. Treba **self-hosted SFU**, a snimanje se radi na
serveru.

| | LiveKit | Jitsi Meet | BigBlueButton | MiroTalk |
|---|---|---|---|---|
| Licenca | Apache 2.0 | Apache 2.0 | LGPL | AGPL |
| Što je | SFU + SDK-ovi | gotova aplikacija | virtualna učionica | gotova aplikacija |
| Ugradnja u vlastiti UI | odlična | iframe, izgleda kao Jitsi | teška | srednja |
| Snimanje | Egress (MP4, HLS, samo audio, stream) | Jibri: jedan Chrome po snimci, teško skalirati | ugrađeno | osnovno |

**Odluka (prijedlog): LiveKit.** Razlozi:

- soba je model platforme (nakana i termin), a tokeni i uloge (predvoditelj
  objavljuje, ostali slušaju) dolaze iz našeg backenda;
- audio-only način s puno slušatelja odgovara krunici;
- snimanje samo zvuka je jeftino;
- LiveKit Cloud za početak, self-host kasnije bez promjene koda.

Jitsi samo kao brzi MVP za provjeru dolaze li ljudi svaki tjedan.

Napomene:

- TURN (LiveKit ga ima ugrađenog) je nužan za mobilne i korporativne NAT-ove.
- Snimka molitve je podatak o vjerskom uvjerenju (GDPR čl. 9.). Treba izričita
  privola i vidljiva oznaka snimanja, a po defaultu bi se snimao samo predvoditelj.
- Kapacitet: 4 vCPU podnosi stotine audio ili desetke video sudionika. Ovo je
  gruba procjena, nije izmjereno.

## Provjera LiveKita i konkurencije (2026-10-25)

### Porijeklo

LiveKit (2021., Russ d'Sa i David Zhao) nije izveden iz Jitsija. Napisan je
od nule u Go-u na **Pionu** (WebRTC biblioteka za Go). Nasljeđuje iskustvo
starijih SFU-ova (simulcast, model sobe), ali s njima ne dijeli kod.

```mermaid
flowchart LR
  Kurento[Kurento ~2013] --> OV2[OpenVidu 2]
  mediasoup[mediasoup 2015] --> OV3[OpenVidu 3]
  Pion[Pion 2018] --> LiveKit[LiveKit 2021]
  LiveKit -->|fork, isti API i SDK| OV3
  LiveKit --> EC[Element Call / Matrix]
  mediasoup --> MiroTalk[MiroTalk SFU]
  Jitsi[Jitsi 2003, zaseban] 
  Janus[Janus 2014, zaseban]
```

### GitHub (stanje 2026-10-25, `gh api`)

Svi projekti su aktivni (zadnji push u listopadu 2026., nijedan nije arhiviran).

| Repo | ★ | Licenca | Napomena |
|---|---:|---|---|
| jitsi/jitsi-meet | 30.142 | Apache-2.0 | gotova aplikacija |
| livekit/livekit | 21.810 | Apache-2.0 | SFU server |
| pion/webrtc | 16.853 | MIT | biblioteka, temelj LiveKita |
| livekit/agents | 14.934 | Apache-2.0 | AI sudionici |
| bigbluebutton/bigbluebutton | 9.252 | LGPL-3.0 | učionica |
| meetecho/janus-gateway | 9.193 | GPL-3.0 | C, plugini |
| versatica/mediasoup | 7.405 | ISC | SFU biblioteka |
| miroslavpejic85/mirotalksfu | 3.136 | AGPL-3.0 | aplikacija na mediasoupu |
| jitsi/jitsi-videobridge | 3.107 | Apache-2.0 | Jitsijev SFU |
| OpenVidu/openvidu | 2.138 | Apache-2.0 | fork LiveKita |
| jech/galene | 1.425 | MIT | mali Go SFU |
| livekit-examples/meet | 977 | Apache-2.0 | primjer aplikacije za sastanke |
| Kurento/kurento | 442 | Apache-2.0 | i dalje se održava (push 2026-10-15) |
| livekit/egress | 372 | Apache-2.0 | snimanje |

Ispravak ranije tvrdnje: **Kurento nije napušten**, i dalje dobiva commitove.
Malen je i rijetko se bira za nove projekte.

### OpenVidu 3

Prema vlastitoj dokumentaciji, OpenVidu 3 je **fork LiveKita, 100%
kompatibilan po API-ju i SDK-ovima**. Koristi standardne `livekit-client` i
`livekit-server-sdk`. Kao media engine nudi Pion (kao LiveKit) ili
**mediasoup**, za koji tvrde da podnosi otprilike 2× više trackova na istom
stroju. To je njihov benchmark, ne neovisan. U paketu su Egress, Ingress,
S3-kompatibilna pohrana i admin dashboard. Ranije je OpenVidu prešao s
Kurenta na mediasoup, a zatim integrirao LiveKit.

Posljedica: kod pisan za LiveKit SDK radi i na OpenVidu 3. To je izlaz ako
self-hostani LiveKit postane premalo učinkovit ili ako zatreba komercijalna
podrška.

### Što je samo u LiveKit Cloudu

- **Napredno uklanjanje šuma** (Krisp, BVC, ai-coustics): ugrađeni modeli traže
  da media ide kroz LiveKit Cloud. Self-host dobiva standardno WebRTC
  uklanjanje šuma u pregledniku. ai-coustics radi i self-hostano, ali uz
  vlastitu licencu. Postoje open source dodaci: RNNoise
  (`livekit/agents` PR #6477) i DTLN (`aloware/livekit-plugins-dtln`).
  Jedan GitHub issue navodi da Krisp od svibnja 2026. postaje plaćen, što nije
  provjereno u službenom cjeniku.
- **Telefonija (SIP)**: Krisp na SIP trunku je samo u Cloudu.
- **Analitika**: Cloud ima dashboard (Analytics API na Scale planu). Kod
  self-hosta je to vlastito rješenje (Prometheus i slično).

Za krunicu je to prihvatljivo: predvoditelju je važna dobra slušalica s
mikrofonom, a ostali su većinom slušatelji. RNNoise se može dodati kasnije.

### Izvori

- <https://docs.livekit.io/transport/self-hosting/>
- <https://livekit.com/blog/noise-cancellation>
- <https://community.livekit.io/t/is-krisp-noise-cancellation-available-with-livekit-cloud/78>
- <https://github.com/livekit/agents/pull/6477>, <https://github.com/livekit/agents/issues/5507>
- <https://github.com/aloware/livekit-plugins-dtln>
- <https://openvidu.io/openvidu-vs-livekit/>, <https://openvidu.io/3.8/docs/comparing-openvidu/>
- <https://docs.openvidu.io/en/stable/openvidu3/>

## Otvoreno

- Ime projekta (Prayminder je zauzeto).
- Potvrditi odluku LiveKit ili Jitsi MVP.
- Kostur: LiveKit u Dockeru, backend koji izdaje tokene po nakani i terminu,
  jednostavan web klijent.
- Repo još nema remote.
