+++
layout = 'post'
title = "Building a Conversational Voice AI Bot"
date = 2025-06-18 01:52:31.730332380+05:45
draft = true
+++

Let's build a voice AI system that can respond to your queries and casual conversations. The system will be able to understand your voice commands, process them, and respond with appropriate answers. We will use Python and some popular libraries/tools to achieve this.
The Voice AI will have an identity, and it will be able to respond to you as your friend. It will have the ability to understand the context, remember past conversations, understand your mood and respond accordingly, and even tell jokes or stories. I am not doing anything fancy here, just using some basic libraries and tools to achieve this. I am trying to leverage the power of LLMs and voice recognition models to build something useful.

### What will we use?
- A basic HTML app to interact with the AI
- A voice recognition library to convert your voice to text (e.g. HumeAI, ElevenLabs etc)
- NestJS backend to transcribe the voice input and process it
- A language model to generate responses (e.g. Claude, OpenAI, or local LLMs)
- A text-to-speech service like HumeAI or ElevenLabs to convert the AI's text response back to voice
- Pinecone or any vector database to store and retrieve past conversations

### How will it work?
- You will speak to the AI using your microphone and audio is sent to the backend.
- The voice recognition library or service like HumeAI will convert your voice to text.
- The text will be stored in a vector database for future reference.
- The text will be sent to the language model to generate a response.
- The LLM generated text is then sent to a text-to-speech service like HumeAI or ElevenLabs to convert it back to voice.
- The voice will be sent to the frontend HTML app to play it back to you.

> We are not going to use websocket for this example, but you can use it to make the communication more real-time. We will use HTTP requests to send and receive data. Websocket is the goto way if you are building this in production.

## Frontend!!
The frontend is going to be a simple HTML app that will allow us to interact with the AI. We record the audio using the browser's audio API and send it to backend for processing. The app will have a simple UI with a button to start recording and a text area to display the transcribed text and an audio player to play the AI's response.
```html
<body>
  <h1>Voice AI Bot</h1>
  <button id="recordButton">Start Recording</button>
  <p id="status">Press the button to start recording</p>
  <h3>Transcription:</h3>
  <p id="transcription">-</p>
  <h3>Response:</h3>
  <p id="response">-</p>
  <audio id="responseAudio" controls></audio>

  <script>
    let mediaRecorder;
    let audioChunks = [];

    const recordButton = document.getElementById('recordButton');
    const status = document.getElementById('status');
    const transcription = document.getElementById('transcription');
    const responseText = document.getElementById('response');
    const responseAudio = document.getElementById('responseAudio');

    recordButton.addEventListener('click', async () => {
      if (recordButton.textContent === 'Start Recording') {
        try {
          const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
          mediaRecorder = new MediaRecorder(stream);
          audioChunks = [];

          mediaRecorder.ondataavailable = (event) => {
            audioChunks.push(event.data);
          };

          mediaRecorder.onstop = async () => {
            const audioBlob = new Blob(audioChunks, { type: 'audio/mp3' });
            const formData = new FormData();
            formData.append('audio', audioBlob, 'recording.mp3');

            status.textContent = 'Sending audio to server...';

            try {
              const response = await fetch('http://localhost:3000/audio/process', {
                method: 'POST',
                body: formData,
              });

              if (!response.ok) {
                throw new Error('Server error');
              }

              const result = await response.json();
              transcription.textContent = result.transcription || 'No transcription received';
              responseText.textContent = result.responseText || 'No response received';

              if (result.responseAudio) {
                const audioUrl = URL.createObjectURL(new Blob([new Uint8Array(atob(result.responseAudio).split('').map(c => c.charCodeAt(0)))], { type: 'audio/wav' }));
                responseAudio.src = audioUrl;
                responseAudio.play();
              }

              status.textContent = 'Processing complete';
            } catch (error) {
              status.textContent = `Error: ${error.message}`;
            }
          };

          mediaRecorder.start();
          recordButton.textContent = 'Stop Recording';
          status.textContent = 'Recording...';
        } catch (error) {
          status.textContent = `Error accessing microphone: ${error.message}`;
        }
      } else {
        mediaRecorder.stop();
        recordButton.textContent = 'Start Recording';
        status.textContent = 'Stopped recording';
      }
    });
  </script>
</body>
```
Ideally you will build UI with a good library like React or Svelte, but for simplicity, we are using plain old JavaScript. We send the audio blob to the backend(Nestjs app) in a post request to `/audio/process` endpoint. The browser's MediaRecorderAPI handles the audio recording and producing chunks that are later converted to blobs.

## Let's move to Backend!!
I am not going to show you the full NestJS setup and installation process here. You can refer to the [NestJS documentation](https://docs.nestjs.com/) for that. We will focus on the audio processing part. Let's create a new module for audio processing. We will use `multer` for handling file uploads.
```bash
npm i -D @types/multer
```
Let's create a controller for handling the audio processing request. Inside `src/audio/audio.controller.ts`:
```typescript
import { Controller, Post, UploadedFile, UseInterceptors, Res } from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';
import { AudioService } from './audio.service';

@Controller('audio')
export class AudioController {
    constructor(private readonly audioService: AudioService) {}

    @Post('process')
    @UseInterceptors(FileInterceptor('audio'))
    async uploadAudio(
        @UploadedFile() audio: Express.Multer.File,
    ) {
        return this.adioService.processAudio(audio);
    }
}
```

Notice that we have imported an `AudioService` which will handle the audio processing logic. We are using `FileInterceptor` from `@nestjs/platform-express` to handle the file upload. The audio file is sent as a form-data with the key `audio`.
Next, let's create the `AudioService` that will handle the audio processing logic. Inside `src/audio/audio.service.ts`:
```typescript

