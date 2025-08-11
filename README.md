
 ✨ **Multilingual Translation Studio**

*A powerful GUI-based app to translate, transcribe, and vocalize text/audio in 25+ languages.*


 🖥️ Features

* 🎙️ **Speech to Text** – Convert microphone/audio input to text
* 🌐 **Multilingual Translation** – Translate between 25+ global languages
* 📁 **File Upload** – Extract text from `.txt` or `.docx` files
* 🔊 **Text to Speech** – Play or save translated text as audio
* 🎵 **Save Audio/Text** – Export output as `.mp3`, `.wav`, or `.txt/.docx`
* 💫 **Animated Splash Screen** – Loading animation in multiple languages
* 🧑‍💻 **User-Friendly GUI** – Built using Python's Tkinter, with modern dark theme

### 🚀 Getting Started

#### ✅ Prerequisites

Make sure you have Python 3.10+ installed. Install required packages with:

bash
pip install -r requirements.txt

#### 🛠 Required Libraries
bash
pip install googletrans==4.0.0-rc1
pip install SpeechRecognition
pip install gTTS
pip install pygame
pip install pydub
pip install python-docx
```

You may also need:

* `ffmpeg` for `pydub` audio conversion (install via package manager)



### 📦 How to Run

bash
python main.py

📂 Project Structure

bash
Multilingual-Translator/
├── main.py
├── requirements.txt
├── temp/                # For storing generated audio
├── README.md

### 🔤 Supported Languages

* English, Hindi, Tamil, Telugu, French, Spanish, German, Arabic, Chinese, Japanese, Russian, etc. *(27 total)*


💡 Future Improvements

* Add language detection
* Add OCR support (image-to-text)
* Improve TTS voice control (speed, pitch, gender)
* Integrate with Whisper/OpenAI for more accurate speech recognition

### 🙋‍♂️ Author
C.Sai Prakash


