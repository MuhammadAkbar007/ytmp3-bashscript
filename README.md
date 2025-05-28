# 🎵 ytmp3 – YouTube to MP3 Downloader *bashscript*

A lightweight Bash script that uses `yt-dlp` & `genius api` to download YouTube videos as MP3s, complete with thumbnails.
Designed for personal use and quick reuse after distro installs.

---
## 📦 Requirements
- [`yt-dlp`](https://github.com/yt-dlp/yt-dlp/releases/) (latest version recommended)
- `ffmpeg` (for audio conversion)

```bash
sudo nala update -y;
sudo nala install ffmpeg -y
```

## 🛠️ Installation (One-time Setup)
Run the following to install the ytmp3 script and make it available system-wide (or at least for your user):

### 1. Clone this repo
```bash
git clone https://github.com/MuhammadAkbar007/ytmp3-bashscript.git ~/akbarDev/petProjects/ytmp3
```

### 2. Create ~/bin if it doesn't exist
```bash
mkdir -p ~/bin
```

### 3. Symlink the script
```bash
ln -s ~/akbarDev/petProjects/ytmp3/ytmp3.sh ~/bin/ytmp3
```

### 4. Ensure ~/bin is in your PATH (usually already is)
Add the following code to your shell configuration file:
(`~/.zshrc` or `~/.bashrc`)

```bash
if [[ ":$PATH:" != *":$HOME/bin:"* ]]; then
  export PATH="$HOME/bin:$PATH"
fi
```

### 5. Make sure it's executable
```bash
chmod +x ~/akbarDev/petProjects/ytmp3/ytmp3.sh
```

---

# 🚀 Usage
Downloads the video as an MP3 to ~/Downloads/mp3s/
```bash
ytmp3 "https://www.youtube.com/watch?v=VIDEO_ID"
```

# 🧹 Uninstallation
```bash
rm ~/bin/ytmp3
```

# 💡 Notes
 * Always keep `yt-dlp` updated:
 ```bash
~/install_me/yt-dlp -U
```

## ✍️Author
Created by [Akbar](https://github.com/MuhammadAkbar007)

