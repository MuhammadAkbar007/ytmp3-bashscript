#!/bin/bash


# === Configuration ===
YTDLP="$HOME/install_me/yt-dlp"           # Path to manually downloaded yt-dlp
SAVE_DIR="$HOME/Downloads/mp3s"           # Define output directory
GENIUS_TOKEN="_s5x_ipHY92DX3qf1fEqrOvftNjZp_Bz9DBSOUSAZXmad700O_9momuwFlXPuZcC"
URL="$1"

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
[[ -z "$URL" ]] && error_exit "🙅‍♂️ No URL provided!"

if ! is_valid_youtube_url "$URL"; then
    error_exit "👾 Invalid YouTube URL provided!"
fi

# 1. Get title from YouTube
TITLE=$("$YTDLP" --get-title "$URL")
CLEANED_TITLE=$(echo "$TITLE" | sed -E 's/\(.*\)|\[.*\]//g' | xargs)
echo -e "\n${GREEN}${CLEANED_TITLE}${RESET}\n"

search_genius() {
    local query="$1"
    local response=$(curl -s -G "https://api.genius.com/search" \
            --data-urlencode "q=$query" \
        -H "Authorization: Bearer $GENIUS_TOKEN")

    echo "$response" | jq  # Pretty-print full JSON
}

echo -e "${CYAN}🔍 Searching Genius for metadata...${RESET}"
GENIUS_RESPONSE=$(curl -s -G "https://api.genius.com/search" \
        --data-urlencode "q=$CLEANED_TITLE" \
    -H "Authorization: Bearer $GENIUS_TOKEN")

# Use jq to extract the first hit
hit=$(echo "$GENIUS_RESPONSE" | jq '.response.hits[0].result')

# Extract fields
GENIUS_TITLE=$(echo "$hit" | jq -r '.title')
GENIUS_ARTIST=$(echo "$hit" | jq -r '.artist_names')
GENIUS_URL=$(echo "$hit" | jq -r '.url')
GENIUS_DATE=$(echo "$hit" | jq -r '.release_date // empty')
GENIUS_ALBUM=$(echo "$hit" | jq -r '.album.name // empty')
GENIUS_THUMBNAIL=$(echo "$hit" | jq -r '.song_art_image_thumbnail_url // empty')
GENIUS_COVER=$(echo "$hit" | jq -r '.song_art_image_url // empty')

mkdir -p "$SAVE_DIR"                      # Ensure directory exists
cd "$SAVE_DIR" || exit 1                  # Switch to download folder

VIDEO_ID=$(extract_youtube_id "$URL")
echo -e "${CYAN}🎬 YouTube ID: $VIDEO_ID${RESET}"

# === Download using yt-dlp ===
echo -e "${YELLOW}⬇️ Downloading MP3...${RESET}"

[[ ! -x "$YTDLP" ]] && error_exit "🚫 yt-dlp not found or not executable at $YTDLP"

# === Auto-update yt-dlp if it's older than 23 days ===
if [[ -x "$YTDLP" ]]; then
    # Get last modification time in epoch seconds (works on GNU/Linux)
    last_modified=$(stat -c %Y "$YTDLP")
    now=$(date +%s)
    age_days=$(( (now - last_modified) / 86400 ))

    if (( age_days > 23 )); then
        echo "🔄 yt-dlp is older than 23 days ($age_days days). Updating..."
        "$YTDLP" -U || echo "⚠️ Auto-update failed, continuing with old version..."
    fi
else
    echo "❌ yt-dlp not found or not executable at $YTDLP"
    exit 1
fi

"$YTDLP" -x \
    --audio-format mp3 \
    --embed-thumbnail \
    --audio-quality 0 \
    -o "$SAVE_DIR/%(title)s.%(ext)s" \
    "$URL" &

show_spinner

# === Success Message ===
echo -e "\n\n${GREEN}✅ Download completed successfully! Saved in: $SAVE_DIR 🎶${RESET}"

