# Free-Offline-AI-Receptionist-for-Small-Businesses
# 🤖 Open-Source Voice Call Assistant

**A free, offline voice assistant for automatically handling phone calls using BlueZ, Ollama, and voice recognition.**

---

## 🎯 Mission

This project was created to help small businesses and individuals who are being charged excessive fees by commercial AI voice assistant services. Why pay hundreds of dollars per month when you can run your own voice assistant completely free and offline?

I built this system to demonstrate that with a Linux PC, some open-source tools, and a bit of setup, anyone can have their own AI-powered call assistant without subscription fees or privacy concerns.

---
## CHECK OUT THE YOUTUBE VIDEO: [https://youtu.be/DtHNSL8JV_U]
## 🌟 How It Works

The system creates a complete voice call handling pipeline:

```
CALLER 
  ↓
📱 Phone (Auto-answers via Automate app)
  ↓
🔵 Bluetooth Connection
  ↓
💻 PC running BlueZ (acts as Bluetooth headset)
  ↓
🎤 PulseAudio Monitor of Sounds (captures audio)
  ↓
🗣️ Vosk (Speech-to-Text)
  ↓
🧠 Ollama AI Model (generates response)
  ↓
🔊 pyttsx3 (Text-to-Speech)
  ↓
🎧 Monitor of Sounds output
  ↓
💻 PC BlueZ
  ↓
🔵 Bluetooth Connection
  ↓
📱 Phone
  ↓
CALLER hears AI response
```

---

## ✨ Features

- **100% Free & Open Source** - No subscription fees, no API costs
- **Completely Offline** - Your conversations stay private
- **Automatic Call Answering** - Uses Automate app to pick up calls
- **Natural Conversations** - Powered by Ollama AI models
- **Customizable Responses** - Edit the system prompt to fit your needs
- **Low Resource Requirements** - Runs on basic hardware without GPU

---

## 📋 Prerequisites

### Hardware Requirements
- A Linux PC (Ubuntu/Debian recommended)
- Bluetooth-enabled phone
- Bluetooth adapter on PC (built-in or USB)
- Microphone and speakers (or monitor of sounds routing)

### Software Requirements
- **Linux OS** (tested on Ubuntu 20.04+)
- **Python 3.8+**
- **BlueZ** (Bluetooth stack for Linux)
- **PulseAudio**
- **Ollama** (for running AI models)
- **Automate app** on Android phone (for auto-answering calls)

---

## 🚀 Installation Guide

### Step 1: Install System Dependencies

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install BlueZ and Bluetooth tools
sudo apt install bluez bluez-tools pulseaudio pulseaudio-module-bluetooth -y

# Install Python and pip
sudo apt install python3 python3-pip python3-dev -y

# Install PortAudio (required for sounddevice)
sudo apt install portaudio19-dev -y
```

### Step 2: Install Python Dependencies

```bash
pip3 install pyttsx3 vosk sounddevice numpy
```

### Step 3: Install Ollama

```bash
# Download and install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull the AI model (phi3 is lightweight and fast)
ollama pull phi3:latest
```

### Step 4: Download Vosk Speech Recognition Model

```bash
# Create directory for models
mkdir -p ~/vosk-models
cd ~/vosk-models

# Download English model (or choose your language)
wget https://alphacephei.com/vosk/models/vosk-model-en-us-0.22.zip

# Extract the model
unzip vosk-model-en-us-0.22.zip

# Note the path - you'll need it in the code
echo "Model path: $(pwd)/vosk-model-en-us-0.22"
```

### Step 5: Configure BlueZ as Bluetooth Headset

```bash
# Edit BlueZ configuration
sudo nano /etc/bluetooth/main.conf
```

Add or modify these lines:
```ini
[General]
Class = 0x200420
DiscoverableTimeout = 0
Enable=Source,Sink,Media,Socket
```

```bash
# Restart Bluetooth service
sudo systemctl restart bluetooth
```

### Step 6: Configure PulseAudio

```bash
# Load Bluetooth modules
pactl load-module module-bluetooth-discover
pactl load-module module-bluetooth-policy

# Set loopback from Bluetooth to Monitor of Sounds
pactl load-module module-loopback source=bluez_source.XX_XX_XX_XX_XX_XX sink=alsa_output.pci-0000_00_1f.3.analog-stereo
```

**Note:** Replace `XX_XX_XX_XX_XX_XX` with your phone's Bluetooth MAC address (find it with `pactl list sources`).

### Step 7: Pair Your Phone

```bash
# Start Bluetooth control
bluetoothctl

# Inside bluetoothctl:
power on
agent on
default-agent
discoverable on
pairable on

# From your phone, pair with the PC
# Accept pairing request

# Trust the device
trust XX:XX:XX:XX:XX:XX
exit
```

### Step 8: Setup Automate App on Android

1. Download **Automate** app from Google Play Store
2. Import the provided flow file: `auto_answer_call.flow` (link in repository)
3. Grant necessary permissions (Phone, Bluetooth)
4. Enable the flow
5. Test by calling your phone

The flow automatically:
- Detects incoming calls
- Connects to PC via Bluetooth
- Answers the call
- Routes audio to PC

---

## 🔧 Configuration

### Update the Voice Assistant Code

Edit the Python script and update these variables:

```python
# Change the Vosk model path to where you extracted it
VOSK_MODEL_PATH = "/home/yourusername/vosk-models/vosk-model-en-us-0.22"

# Customize the assistant's behavior
SYSTEM_PROMPT = """
You are an offline call assistant named "Aura" for [YOUR NAME/BUSINESS].
Your role is to answer calls because [REASON].
[ADD YOUR CUSTOM INSTRUCTIONS HERE]
"""

# You can also change the AI model
MODEL_NAME = "phi3:latest"  # or try "llama2", "mistral", etc.
```

### Adjust Voice Settings

```python
# In the speak() function, customize TTS
engine.setProperty('rate', 150)    # Speed (50-300)
engine.setProperty('volume', 0.9)  # Volume (0.0-1.0)

# You can also change the voice
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)  # Try different indices
```

---

## 🎮 Usage

### Running the Voice Assistant

```bash
# Navigate to your script directory
cd ~/voice-assistant

# Run the assistant
python3 voice_assistant.py
```

The assistant will:
1. Start listening for audio input
2. Wait for a call to come in
3. Transcribe the caller's speech
4. Generate AI responses
5. Speak back to the caller
6. Continue the conversation until the call ends

### Stopping the Assistant

- Press `Ctrl+C` in the terminal
- Or say "exit" during a call (if configured)

---

## 🐛 Troubleshooting

### No Audio from Phone
```bash
# Check PulseAudio sources
pactl list sources short

# Restart PulseAudio
pulseaudio -k
pulseaudio --start
```

### Bluetooth Connection Issues
```bash
# Restart Bluetooth
sudo systemctl restart bluetooth

# Re-pair the device
bluetoothctl
remove XX:XX:XX:XX:XX:XX
# Then pair again
```

### Ollama Not Responding
```bash
# Stop background service
sudo systemctl stop ollama

# Run manually to see errors
ollama serve
```

### Poor Speech Recognition
- Speak clearly and at moderate pace
- Reduce background noise
- Try a different Vosk model (larger = more accurate)
- Adjust microphone input levels in PulseAudio

---

## 🔮 Future Plans

- **Windows Support**: Working on a Windows version using alternative Bluetooth solutions
- **Better Models**: Optimizing for GPUs to use larger, more capable AI models
- **Web Interface**: Adding a web dashboard to monitor calls and responses
- **Multi-language Support**: Adding support for multiple languages
- **Call Recording**: Optional feature to save call transcripts
- **SMS Integration**: Ability to send follow-up text messages

---

## 🤝 Contributing

This project is open source and welcomes contributions! Whether you:
- Found a bug
- Have a feature suggestion
- Want to improve documentation
- Can help with Windows compatibility
- Want to optimize performance

**All contributions are appreciated!**

---

## 📜 License

This project is released under the MIT License - feel free to use, modify, and distribute as you wish.

---

## ⚠️ Disclaimer

This software is provided as-is for educational and personal use. Please ensure compliance with local laws regarding call recording and automation. Always inform callers if they're speaking with an AI assistant.

---

## 💬 Contact & Support

**Creator**: Kevin (8th Grade Student at Soman Highschool)

**Why I Built This**: I was frustrated seeing small businesses getting charged $200-500/month for basic AI call assistants. This technology should be accessible to everyone, not just those who can afford expensive subscriptions.

**Need Help?**
- 💬 GitHub Issues: [https://github.com/XOZ3E]
- 💬 Telegram: [https://t.me/XCZGITHUB]
- 💬 Discord: [https://discord.gg/tYj5vw2BNM]

---

## 🌟 Support the Project

If this project helped you or your business save money, consider:
- ⭐ Starring the repository
- 🔄 Sharing it with others who might benefit
- 📝 Contributing improvements
- ☕ Buying me a coffee (link if you want donations)

**Together, we can make AI technology accessible to everyone!**

---

**Made with ❤️ to help small businesses thrive without breaking the bank.**

