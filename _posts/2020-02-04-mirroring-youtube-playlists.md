---
layout: post
title: "Mirroring YouTube Playlists"
---

Recently I wanted to set up a periodic job to mirror some of my personal
youtube playlists. There's plenty of reasons one might want to do this. For me
it's simple: one copy is none copy, and two copies is one copy, so I want a
second copy of youtube videos I care about stored locally. This protects
against videos getting removed, copyright stuck, youtube shutting down, or
anything else that might make the youtube video otherwise unavailable.

The final straw for me was the recent YouTube Kids change. YouTube has
implemented a system whereby it flags videos as "For Kids", and a video flagged
as "For Kids" can not be added to playlists, or interacted with beyond viewing,
and may eventually start disappearing from search results. This seems kinda
fine on the surface, but the automated systems flagging videos are _very bad_
at actually detecting kid-friendly content. For example, Pony Music Videos
(PMVs), which are fan made cuts of My Little Pony video footage over top
unrelated songs, have been getting flagged quite a bit, [even when the song has
obviously explicit lyrics.](https://youtu.be/rl2MUeiOngg)

## The Script

Anyways, I just wanted a simple script adding some extra features around
[youtube-dl](https://ytdl-org.github.io/youtube-dl/index.html), so I made one!
You can find the full script at
[https://github.com/faithanalog/x/blob/master/youtube-archiver/archive-playlists](https://github.com/faithanalog/x/blob/master/youtube-archiver/archive-playlists).
I'm going to go over a few specifics in case you're interested in making your
own rather than just using my script or someone else's. But, if you don't care about that, and you want to use my script, here's what to do:

First, create a list of playlists in a file called, for example, `playlists.txt`

```
Nightcore Songs
https://www.youtube.com/playlist?list=PLckeMyCaCCIN_JU1V4oADW50DlGOREoLj

Vaporwave
https://www.youtube.com/playlist?list=PLgP_WFDJWjxTRPJtV4DV99lGB92rq5we_
```

Optionally, create a list of [SOCKS proxies](https://en.wikipedia.org/wiki/SOCKS)
you want to use in another file, for example, `proxies.txt`

```
127.0.0.1:9050
example.com:1080
```

Then, you can run my script!

```
# If you don't have any proxies to use
archive-playlists playlists.txt

# If you do have proxies to use
archive-playlists playlists.txt proxies.txt
```

This will create folders with the names provided by `playlists.txt`, and
download each playlist to its own folder. You can run it periodically however
you feel most comfortable (crontab, systemd timers, etc.) and it should keep
all the local copies up to date.


## Details

The requirements I had for my script were pretty simple:

- Download videos, thumbnails, subtitles, and video metadata
- Write playlist information to its own file
- When re-running the script, download only the newly required data
- Ideally, avoid getting myself IP-banned

Most of this can be taken care of with various youtube-dl flags! I have a
function called `dl_playlist()` which implements all of this. Arguments passed
to `dl_playlist()` are transparenltly passed on to any youtube-dl commands,
which allows me to pass it a playlist URL, and optionally a proxy.

First, we download the playlist metadata:

```bash
playlist_json="playlist-$(date --rfc-3339=date).json"
youtube-dl \
    --flat-playlist \
    -J \
    "$@" < /dev/null > "$playlist_json"

temp_playlist_json="$(mktemp -p ./)"
cp "$playlist_json" "$temp_playlist_json"
mv "$temp_playlist_json" playlist.json
```

This writes the current metadata to a dated file, copies that to a temporary
file, and then renames the temporary file to `playlist.json`. This allows us to
have a history of the playlist over time, which may make it easier to figure
out what videos are missing when they get taken down. `playlist.json` will
always contain the latest playlist information, to make it easier for me to
write tools for this later.

The process of copying to a temporary file and then renaming the temporary file
may seem overkill, but `mv` is an atomic operation, so it eases my mind about
any race conditions I might run into if I'm running other tools over this data
to process it.

Next, we can download the playlist videos.

```bash
youtube-dl \
    --download-archive ytdl-archive.txt \
    --write-info-json \
    --write-description \
    --write-thumbnail \
    --all-subs \
    -i \
    -f bestvideo+bestaudio \
    -r 500K \
    --sleep-interval $min_sleep \
    --max-sleep-interval $max_sleep \
    "$@" < /dev/null
```

Here's where all the fun flags come into play!

- `--download-archive ytdl-archive.txt` tracks which videos have been fully
  downloaded in a file called ytdl-archive.txt. This allows youtube-dl to skip
  loading the page for the video entirely on subsequent runs.
- `--write-info-json`, `--write-description`, and `--write-thumbnail` are all
  fairly self explanatory. I don't even really need to write the description,
  since the info json contains it, but it might be convenient to have in its
  own file.
- `--all-subs` instructs youtube-dl to download all available subtitles. I
  don't really know which subtitles I need, so might as well just have them all
  available.
- `-i` makes youtube-dl ignore download errors. Without this, if a video is
  missing, youtube-dl will stop running after trying to retrieve it, and skip
  the rest of the playlist.
- `-f bestvideo+bestaudio` makes youtube-dl download the best quality video and
  audio files separately, and merge then into a single `.mkv` file.
- `-r 500K` rate-limits downloads to 500 Kilobytes per second. This is part my
  attempts to avoid the ire of automated IP-bans. I don't have any source on
  what a safe range for download rates is, but this seems to work for my
  usecase, so I'm keeping it.
- `--sleep-interval` and `--max-sleep-interval` together specify the minimum
  and maximum sleep times. youtube-dl will pick a random time between these
  values between downloads. I was a bit confused about the semantics of what
  counts as a download- youtube-dl sleeps after downloading a thumbnail, video
  file, or audio file, but _doesn't_ sleep after downloading the info json.
  Anyways, I use a range of 30-90 seconds, and this seems to keep things slow
  enough that it shouldn't be a problem.

The rest of the script is just housekeeping around this. It loads playlists
from a file, creates separate directories for each one, and randomly shuffles
through a list of [SOCKS proxies](https://en.wikipedia.org/wiki/SOCKS) for each
playlist. My VPN provider provides SOCKS proxies for each of their exit nodes,
so this is a really convenient way for me to distribute my downloads across a
broader range of IP addresses.

And that's it!  The script is pretty well commented I feel, but if you have any
questions about it, [feel free to ask!](/contact.html)

Thanks to everyone on the fediverse who helped me iterate on this script and
iron out the details. ❤️

– artemis
