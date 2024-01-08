# HuHa's youtube-dl Tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License


## Modern replacement: yt-dlp

Try installing package `yt-dlp` from your distro.


## Install youtube-dl

YouTube keeps changing their web site all the time, so you need the latest
youtube-dl, not one from a 3 years old Linux distribution that rarely, if ever,
gets updated.

    cd /usr/local/bin
    sudo wget https://yt-dl.org/downloads/latest/youtube-dl
    sudo chmod 755 youtube-dl

If `wget` is not available, use `curl`:

    cd /usr/local/bin
    sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o youtube-dl
    sudo chmod 755 youtube-dl

See also

  https://ytdl-org.github.io/youtube-dl/download.html


## Update!

If you haven't used it for a couple of days, always update it:

    sudo youtube-dl --update


## Default Call: Best Video, Best Audio

Don't forget the quotes, otherwise the shell will try to expand the `?`
wildcard and throw an error "no matches found".

    youtube-dl 'https://www.youtube.com/watch?v=...'

You might want to update the file's timestamp to just now:

    touch ...


## Get Only the Audio as MP3

    youtube-dl --extract-audio --audio-quality 0 --audio-format mp3 'https://www.youtube.com/watch?v=...'

You might want to edit the ID3 tags with a good tool like `kid3`:

    kid3 *.mp3


## Check what Formats are Available for the Video

    youtube-dl -F 'https://www.youtube.com/watch?v=...'


## Select a Specific Format

Pick the video and the audio format from the `-F` output and combine them like this:

    youtube-dl -f '244+251' ''https://www.youtube.com/watch?v=...'

Important: Video format **first**, then the audio format!


## Limit Video Resolution, but Auto-Select Formats

To download max. 720p to keep the video size down:

    youtube-dl -f 'bestvideo[height<=720]+bestaudio' 'https://www.youtube.com/watch?v=...'

_This doesn't seem to work anymore._


## Reference

- youtube-dl GitHub Repo + Docs:

  https://github.com/ytdl-org/youtube-dl/

- Examples in the docs:

  https://github.com/ytdl-org/youtube-dl/blob/master/README.md#format-selection-examples
