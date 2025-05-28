#!/bin/bash

# === Configuration ===
YTDLP="$HOME/install_me/yt-dlp"           # Path to manually downloaded yt-dlp
SAVE_DIR="$HOME/Downloads/mp3s"           # Define output directory

mkdir -p "$SAVE_DIR"                      # Ensure directory exists
cd "$SAVE_DIR" || exit 1                  # Switch to download folder

# === Color Definitions ===
RED="\e[31m"
GREEN="\e[32m"
YELLOW="\e[33m"
CYAN="\e[36m"
RESET="\e[0m"

# === Spinner ===
show_spinner() {
    local pid=$!
    local spin='|/-\'
    local i=0

    while kill -0 $pid 2>/dev/null; do
        i=$(( (i+1) % 4 ))
        printf "\r🌀 Downloading... ${spin:$i:1}"
        sleep 0.1
    done
}

# === Error Exit ===
error_exit() {
    echo -e "${RED}❌ Error: $1${RESET}"
    exit 1
}

# === Validate YouTube URL ===
is_valid_youtube_url() {
    local url="$1"
    local regex='^(https?://)?(www\.)?(youtube\.com/(watch\?v=|embed/|v/|.+\?v=)|youtu\.be/|youtube-nocookie\.com/(embed/|watch\?v=))([a-zA-Z0-9_-]{11})(\?[^[:space:]]*)?$'

    if [[ "$url" =~ $regex ]]; then
        return 0
    else
        return 1
    fi
}

# === Extract YouTube Video ID ===
extract_youtube_id() {
    local url="$1"
    # Match various YouTube formats and extract ID using parameter expansion
    if [[ "$url" =~ (youtu\.be/|v=|\/v\/|embed/)([a-zA-Z0-9_-]{11}) ]]; then
        echo "${BASH_REMATCH[2]}"
    else
        echo ""
    fi
}

# === Main Logic ===
URL="$1"
[[ -z "$URL" ]] && error_exit "🙅‍♂️ No URL provided!"

if ! is_valid_youtube_url "$URL"; then
    error_exit "👾 Invalid YouTube URL provided!"
fi

VIDEO_ID=$(extract_youtube_id "$URL")
echo -e "${CYAN}🎬 YouTube ID: $VIDEO_ID${RESET}"

# === Download using yt-dlp ===
echo -e "${YELLOW}⬇️ Downloading MP3...${RESET}"

[[ ! -x "$YTDLP" ]] && error_exit "🚫 yt-dlp not found or not executable at $YTDLP"

"$YTDLP" -x \
    --audio-format mp3 \
    --embed-thumbnail \
    --audio-quality 0 \
    -o "$SAVE_DIR/%(title)s.%(ext)s" \
    "$URL" &

show_spinner

# === Success Message ===
echo -e "\n\n${GREEN}✅ Download completed successfully! Saved in: $SAVE_DIR 🎶${RESET}"

