# NB Whisper Webapp

En enkel webapplikasjon for norsk tale-til-tekst transkripsjon ved bruk av Modal API.

## Funksjonalitet

- **Ta opp lyd** direkte i nettleseren
- **Automatisk transkripsjon** av norsk tale
- **Sanntidsrespons** fra Modal GPU
- **Responsivt design** for mobil og desktop

## Teknologi

- **Frontend**: Vanilla HTML/CSS/JavaScript (ingen dependencies)
- **Backend**: Modal serverless med NbAiLab/nb-whisper-large
- **Audio API**: MediaRecorder Web API
- **Hosting**: GitHub Pages

## Live Demo

🔗 **URL**: https://torrylol.github.io/nb-whisper-webapp/

## Bruk

1. Åpne webappen i nettleseren
2. Klikk på "Start Opptak"
3. Gi nettleseren tillatelse til mikrofon
4. Snakk tydelig på norsk
5. Klikk på "Stopp Opptak"
6. Vent 5-15 sekunder på transkripsjon

## Lokal Testing

### Metode 1: Python Server

```bash
python3 -m http.server 8000
# Åpne http://localhost:8000
```

### Metode 2: Direkte i Nettleser

Åpne `index.html` direkte i nettleseren. **Merk**: CORS kan forhindre API-kall fra `file://` protokoll.

## Teknisk Arkitektur

```
Browser → MediaRecorder → Audio Blob → FormData → Modal API → GPU → Transcription
```

### Audio Recording

```javascript
// 1. Be om mikrofon-tilgang
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

// 2. Start opptak med MediaRecorder
const mediaRecorder = new MediaRecorder(stream);

// 3. Samle audio chunks
const audioChunks = [];
mediaRecorder.ondataavailable = (event) => {
    audioChunks.push(event.data);
};

// 4. Opprett Blob ved stopp
mediaRecorder.onstop = () => {
    const audioBlob = new Blob(audioChunks, { type: 'audio/webm' });
    sendToModal(audioBlob);
};
```

### API Call

```javascript
// Send til Modal endpoint
const formData = new FormData();
formData.append('file', audioBlob, 'recording.webm');

const response = await fetch('MODAL_ENDPOINT', {
    method: 'POST',
    body: formData
});

const result = await response.json();
// { "text": "Transkripsjonen her" }
```

## Modal Endpoint

**URL**: `https://torrylol--whisper-norwegian-transcribe-file.modal.run`

**Method**: POST
**Content-Type**: multipart/form-data
**Parameter**: `file` (UploadFile)

**Response**:
```json
{
  "text": "Transkribert tekst"
}
```

## Browser Support

- **Chrome/Edge**: ✅ Full support
- **Firefox**: ✅ Full support
- **Safari**: ✅ Full support (iOS 14.3+)
- **Opera**: ✅ Full support

**Krav**:
- HTTPS eller localhost (for mikrofon-tilgang)
- MediaRecorder API support
- Fetch API support

## Sikkerhet & Privacy

- **Ingen datalagringing**: Lyden sendes direkte til Modal og slettes umiddelbart
- **HTTPS**: All kommunikasjon er kryptert
- **Mikrofon-tilgang**: Brukeren må gi eksplisitt tillatelse

## Kostnader

**Per opptak** (ca 10 sekunder):
- Modal GPU-tid: 5-10 sekunder
- Kostnad med gratis tier: $0
- Etter gratis tier: ~$0.001

**Estimat**:
- Flere hundre opptak/måned gratis
- Ubegrenset bruk mulig med betalt plan

## Feilsøking

### "Kunne ikke få tilgang til mikrofon"

**Løsning**:
1. Sjekk at siden kjører på HTTPS eller localhost
2. Sjekk nettleserinnstillinger for mikrofon-tillatelser
3. Prøv å laste siden på nytt

### "Kunne ikke transkribere lyden"

**Mulige årsaker**:
1. Modal endpoint er nede (sjekk https://modal.com/status)
2. Nettverksproblemer
3. Ugyldig audioformat

**Løsning**: Prøv igjen om noen sekunder

### CORS Error

**Problem**: Cross-Origin Resource Sharing blokkert

**Løsning**:
- Modal fastapi_endpoint håndterer CORS automatisk
- Hvis problemet vedvarer, test fra GitHub Pages i stedet for lokal fil

## Fremtidige Forbedringer

- [ ] Last ned transkripsjon som tekstfil
- [ ] Lagre historikk i localStorage
- [ ] Støtte for fil-upload (i tillegg til opptak)
- [ ] Språkvalg (norsk, engelsk, etc.)
- [ ] Tidsstempler i transkripsjon
- [ ] Rediger transkripsjon før lagring

## Relaterte Prosjekter

- **Backend API**: [nb-whisper-modal](https://github.com/torrylol/nb-whisper-modal)
- **Whisper Model**: [NbAiLab/nb-whisper-large](https://huggingface.co/NbAiLab/nb-whisper-large)

## Lisens

Privat prosjekt - ikke for kommersiell bruk.
