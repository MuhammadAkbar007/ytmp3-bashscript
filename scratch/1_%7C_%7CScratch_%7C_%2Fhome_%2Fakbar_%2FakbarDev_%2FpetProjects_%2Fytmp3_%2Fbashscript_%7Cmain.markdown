```bash
# 1. Get title from YouTube
TITLE=$("$YTDLP" --get-title "$URL")

CLEANED_TITLE=$(echo "$TITLE" \
        | sed -E 's/\(.*\)|\[.*\]//g' \
        | sed -E 's/^[[:space:]]+|[[:space:]]+$//g' \
        | sed -E 's/^(.*) - (.*)$/\2 - \1/' \
        | sed 's#[/:*?"<>|]##g' \
        | sed 's/`/'"'"'/g' \
        | sed -E 's/^([^-]*) - (.*)$/\L\1\E - \2/' \
        | sed -E 's/^(.)/\U\1/' \
    )```
