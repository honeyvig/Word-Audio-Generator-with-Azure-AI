# Word-Audio-Generator-with-Azure-AI
one webpage (no  pretty design needed) just UI input , give a text area, I input some english words, , it will generate the text to speech with Azure, and give the words back to me as one word per file mp3, after form submission.
===================
To create a simple web application that takes user input in a text area and generates text-to-speech (TTS) with Azure's Cognitive Services, followed by saving each word as an individual MP3 file, we can break the task into several parts:

    Web Interface (UI): We'll use Flask (a Python web framework) to create a simple webpage where users can input text.
    Azure TTS Integration: We'll use Azure's Text-to-Speech API to convert the text into speech.
    Generate MP3 Files: For each word, the application will generate an individual MP3 file and return them to the user.
    Handle Form Submission: After submitting the form, we process the text and generate the MP3 files for each word.

Steps to Implement:

    Set Up Azure TTS:
        You need to have an Azure Cognitive Services account and a Speech API key. Once you have this, you'll be able to use the azure-cognitiveservices-speech library.

    Flask Web Application:
        We'll create a Flask app with an input form and a backend to handle the form submission, generate MP3s, and send them back to the user.

    Generate MP3 Files:
        For each word, we will use Azure's TTS API to generate an MP3 and save them with unique filenames.

Prerequisites:

    Azure Cognitive Services API Key and Region.
    Install dependencies:

    pip install flask azure-cognitiveservices-speech

Full Implementation Code:
1. Flask Web Application (app.py)

import os
import azure.cognitiveservices.speech as speechsdk
from flask import Flask, render_template, request, send_from_directory

# Initialize Flask app
app = Flask(__name__)

# Azure Speech API credentials
API_KEY = 'YOUR_AZURE_API_KEY'
REGION = 'YOUR_AZURE_REGION'

# Directory to store generated MP3 files
OUTPUT_DIR = 'generated_audio'
os.makedirs(OUTPUT_DIR, exist_ok=True)

# Function to generate speech for each word
def generate_mp3_for_word(word, index):
    speech_config = speechsdk.SpeechConfig(subscription=API_KEY, region=REGION)
    audio_config = speechsdk.audio.AudioOutputConfig(use_default_speaker=False, file_name=f"{OUTPUT_DIR}/word_{index}.mp3")

    # Create the speech synthesizer
    synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)

    # Synthesize the word to MP3
    synthesizer.speak_text_async(word)

# Home route
@app.route('/')
def home():
    return render_template('index.html')

# Handle form submission
@app.route('/generate', methods=['POST'])
def generate():
    # Get input text from the form
    text = request.form['text']
    
    # Split text into words
    words = text.split()

    # Generate MP3 files for each word
    for index, word in enumerate(words):
        generate_mp3_for_word(word, index)

    # Return a response that allows the user to download the files
    mp3_files = [f'word_{i}.mp3' for i in range(len(words))]
    return render_template('download.html', files=mp3_files)

# Route to serve the generated MP3 files
@app.route('/download/<filename>')
def download(filename):
    return send_from_directory(OUTPUT_DIR, filename)

if __name__ == '__main__':
    app.run(debug=True)

2. HTML Templates (index.html and download.html)

    index.html: A simple form to input text.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Text to Speech Generator</title>
</head>
<body>
    <h1>Text to Speech Converter</h1>
    <form action="/generate" method="POST">
        <textarea name="text" rows="5" cols="50" placeholder="Enter text here..."></textarea><br><br>
        <input type="submit" value="Generate Speech">
    </form>
</body>
</html>

    download.html: This page will display download links for each generated MP3 file.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Download Generated MP3s</title>
</head>
<body>
    <h1>Download MP3 Files</h1>
    <p>Your audio files are ready. Click to download:</p>
    <ul>
        {% for file in files %}
            <li><a href="{{ url_for('download', filename=file) }}" download>{{ file }}</a></li>
        {% endfor %}
    </ul>
</body>
</html>

Explanation:

    Flask Setup:
        The web app is built using Flask, and it runs on a basic server with two main routes: the home page (with the form) and a generate route to handle text input, process it with Azure's TTS API, and generate individual MP3 files.
        The /generate route splits the input text into words, generates an MP3 file for each word, and provides download links for the generated files.

    Azure TTS:
        The generate_mp3_for_word function uses Azure Cognitive Services Speech SDK to convert each word to speech and save it as an MP3 file.
        Each MP3 file is saved with a unique name: word_<index>.mp3.

    Download Links:
        After generating the MP3 files, the app renders a page (download.html) with a list of downloadable files, each file linked via the /download route.

    Directory Management:
        The generated MP3 files are saved in the generated_audio/ directory, and this directory is served via Flask for downloading.

Running the App:

    Replace 'YOUR_AZURE_API_KEY' and 'YOUR_AZURE_REGION' with your Azure Speech API key and region.
    Run the Flask app:

    python app.py

    Open a browser and go to http://127.0.0.1:5000/.
    Input some text in the textarea and submit the form. After processing, you'll be presented with download links for the generated MP3 files.

Notes:

    This is a simple, non-pretty UI for testing purposes. You can further enhance it with CSS and other front-end features.
    Ensure your Azure API key and region are securely stored (e.g., using environment variables) instead of hard-coding them into the script.

