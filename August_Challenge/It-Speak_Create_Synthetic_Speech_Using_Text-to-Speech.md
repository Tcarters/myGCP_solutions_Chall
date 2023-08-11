# It Speaks! Create Synthetic Speech Using Text-to-Speech


### Requirement

```bash
   gcloud config set compute/region us-west1

```

## Task 1. Enable the Text-to-Speech API


## Task 2. Create a virtual environment

```bash
    ## Install the virtualenv environment:

    sudo apt-get install -y virtualenv

    ## Build the Environment
    python3 -m venv venv

    ## Activate the virtual env
    source venv/bin/activate

```

## Task 3. Create a service account

```bash
    gcloud iam service-accounts create tts-qwiklab

    ## generate a key to use service
    gcloud iam service-accounts keys create tts-qwiklab.json --iam-account tts-qwiklab@qwiklabs-gcp-00-45992abb7de8.iam.gserviceaccount.com

    ##
    export GOOGLE_APPLICATION_CREDENTIALS=tts-qwiklab.json
```

## Task 4. Get a list of available voices

- The following curl command gets the list of all the voices you can select from when creating synthetic speech using the Text-to-Speech API:


```bash
    curl -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
    -H "Content-Type: application/json; charset=utf-8" \
    "https://texttospeech.googleapis.com/v1/voices"

    curl -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
    -H "Content-Type: application/json; charset=utf-8" \
    "https://texttospeech.googleapis.com/v1/voices?language_code=en"

```

## Task 5. Create synthetic speech from text

```bash
    touch synthesize-text.json

    {
    'input':{
        'text':'Cloud Text-to-Speech API allows developers to include
           natural-sounding, synthetic human speech as playable audio in
           their applications. The Text-to-Speech API converts text or
           Speech Synthesis Markup Language (SSML) input into audio data
           like MP3 or LINEAR16 (the encoding used in WAV files).'
    },
    'voice':{
        'languageCode':'en-gb',
        'name':'en-GB-Standard-A',
        'ssmlGender':'FEMALE'
    },
    'audioConfig':{
        'audioEncoding':'MP3'
    }
}

```

```bash
    curl -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
  -H "Content-Type: application/json; charset=utf-8" \
  -d @synthesize-text.json "https://texttospeech.googleapis.com/v1/text:synthesize" \
  > synthesize-text.txt

```

- Create a file

```bash
    touch tts_decode.py

    ## CODE IN FILE

    import argparse
from base64 import decodebytes
import json
"""
Usage:
        python tts_decode.py --input "synthesize-text.txt" \
        --output "synthesize-text-audio.mp3"
"""
def decode_tts_output(input_file, output_file):
    """ Decode output from Cloud Text-to-Speech.
    input_file: the response from Cloud Text-to-Speech
    output_file: the name of the audio file to create
    """
    with open(input_file) as input:
        response = json.load(input)
        audio_data = response['audioContent']
        with open(output_file, "wb") as new_file:
            new_file.write(decodebytes(audio_data.encode('utf-8')))
if __name__ == '__main__':
    parser = argparse.ArgumentParser(
        description="Decode output from Cloud Text-to-Speech",
        formatter_class=argparse.RawDescriptionHelpFormatter)
    parser.add_argument('--input',
                       help='The response from the Text-to-Speech API.',
                       required=True)
    parser.add_argument('--output',
                       help='The name of the audio file to create',
                       required=True)
    args = parser.parse_args()
    decode_tts_output(args.input, args.output)

```

- Get result

```bash
    python tts_decode.py --input "synthesize-text.txt" --output "synthesize-text-audio.mp3"

```

- Cretae an HTML file

```bash
    touch index.html

    <html>
  <body>
  <h1>Cloud Text-to-Speech codelab</h1>
  <p>
  Output from synthesizing text:
  </p>
  <audio controls>
  <source src="synthesize-text-audio.mp3" />
  </audio>
  </body>
    </html>
    </ql-code-block>

```

- Start an HTTP server

```bash
    python -m http.server 8080
```

## Task 6. Create synthetic speech from SSML

```bash
    touch synthesize-ssml.json

    {
    'input':{
        'ssml':'<speak><s>
           <emphasis level="moderate">Cloud Text-to-Speech API</emphasis>
           allows developers to include natural-sounding
           <break strength="x-weak"/>
           synthetic human speech as playable audio in their
           applications.</s>
           <s>The Text-to-Speech API converts text or
           <prosody rate="slow">Speech Synthesis Markup Language</prosody>
           <say-as interpret-as=\"characters\">SSML</say-as>
           input into audio data
           like <say-as interpret-as=\"characters\">MP3</say-as> or
           <sub alias="linear sixteen">LINEAR16</sub>
           <break strength="weak"/>
           (the encoding used in
           <sub alias="wave">WAV</sub> files).</s></speak>'
    },
    'voice':{
        'languageCode':'en-gb',
        'name':'en-GB-Standard-A',
        'ssmlGender':'FEMALE'
    },
    'audioConfig':{
        'audioEncoding':'MP3'
    }
}

```

- Call the Text-to-Speech API 

```bash
    curl -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
  -H "Content-Type: application/json; charset=utf-8" \
  -d @synthesize-ssml.json "https://texttospeech.googleapis.com/v1/text:synthesize" \
  > synthesize-ssml.txt

```

- Run

```bash
    python tts_decode.py --input "synthesize-ssml.txt" --output "synthesize-ssml-audio.mp3"


    ## Create a new index.html file
    <html>
  <body>
  <h1>Cloud Text-to-Speech Demo</h1>
  <p>
  Output from synthesizing text:
  </p>
  <audio controls>
    <source src="synthesize-text-audio.mp3" />
  </audio>
  <p>
  Output from synthesizing SSML:
  </p>
  <audio controls>
    <source src="synthesize-ssml-audio.mp3" />
  </audio>
  </body>
</html>


    ## Start server
    python -m http.server 8080

```

## Task 7. Configure audio output and device profiles

```bash
    touch synthesize-with-settings.json

    {
    'input':{
        'text':'The Text-to-Speech API is ideal for any application
          that plays audio of human speech to users. It allows you
          to convert arbitrary strings, words, and sentences into
          the sound of a person speaking the same things.'
    },
    'voice':{
        'languageCode':'en-us',
        'name':'en-GB-Standard-A',
        'ssmlGender':'FEMALE'
    },
    'audioConfig':{
      'speakingRate': 1.15,
      'pitch': -2,
      'audioEncoding':'OGG_OPUS',
      'effectsProfileId': ['headphone-class-device']
    }
    }

```

- Use the following code to call the Text-to-Speech API using the curl command:


```bash
    curl -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
  -H "Content-Type: application/json; charset=utf-8" \
  -d @synthesize-with-settings.json "https://texttospeech.googleapis.com/v1beta1/text:synthesize" \
  > synthesize-with-settings.txt

```

- Execute 

```bash
    python tts_decode.py --input "synthesize-with-settings.txt" --output "synthesize-with-settings-audio.ogg"

    ## Create index.html
    <html>
  <body>
  <h1>Cloud Text-to-Speech Demo</h1>
  <p>
  Output from synthesizing text:
  </p>
  <audio controls>
    <source src="synthesize-text-audio.mp3" />
  </audio>
  <p>
  Output from synthesizing SSML:
  </p>
  <audio controls>
    <source src="synthesize-ssml-audio.mp3" />
  </audio>
  </body>
  <p>
  Output with audio settings:
  </p>
  <audio controls>
    <source src="synthesize-with-settings-audio.ogg" />
  </audio>
</html>

    ## Start a server
    python -m http.server 8080
```


------------------------------------------------_FINAL_---------------------------------