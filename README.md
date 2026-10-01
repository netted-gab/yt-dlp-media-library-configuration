Summary:

A yt-dlp configuration for downloading music albums from YouTube or YT Music with:

- preferred audio format (e.g. Opus)
- embedded cover art
- embedded metadata
- cleaner titles
- artist/album folder organization
- playlist track numbering

***

The repository provides a yt-dlp configuration for downloading and organizing music, from YouTube but, especially, as it works really well (01/10/2026), for YouTube Music.
Being fairly simple and linear, it should be easily customizable, tailored to the needs of your machines, devices, players you listen music on.

Tips:

1. using yt music links works best, particularly for the album cover
2. you can amass multiple links (of multiple albums) in a .txt file, one for every row, and call the .txt instead. Queuing the whole thing
4. placing the file in your yt-dlp directory SHOULD automatically make it recognisable as the configuration file. If it doesn't work, call it via terminal. If there is another config file being used automatically you can:
--ignore-config --config-location "C:\user\...(where you placed the config file)"

***

## Requirements:

- yt-dlp
- FFmpeg

***

## Optional (in theory):

- Firefox (if using `--cookies-from-browser firefox`)
- Node.js (if using `--js-runtimes node`)

Because there are many different ways to bypass YouTube anti-bot mechanisms. This were the easiest I could find.

***

## Installation

Copy `yt-dlp.conf` wherever you want to keep your yt-dlp configuration.

***

## Usage

```bash
yt-dlp --config-location "C:\...\.conf folder location" URL
```

If placed in the same folder as yt-dlp executable, and depending on the version, you may not need to call the configuration. If you store the configuration file in another folder you'll need to call for it.

***

Example output:

```text
Artist/
└── Album/
    ├── 01 - Song.opus
    ├── 02 - Song.opsu
    └── cover.jpg
```

This is how the file should appear in the folder. Check Screenshot_1.png.
It will create a main folder named "Artist name", a subfolder named "Album name", and the whole playlist you downloaded inside.

note on Screenshot_1.png:
as I moved my library from m4a to opus, Windows 11 File Explorer stopped recognizing track indexes and various tags. Check your device, or preferred player, for file compression and metadata compatibility (older machines might need mp3).

***

--cookies-from-browser firefox

YouTube uses cookies to identify browser sessions. Reusing your own browser cookies allows yt-dlp to make requests with the same authentication you have. It basically allows access to the content YOUR account is permitted to view. (If you are not logged in it's still using the ones you are storing as a temporary user of YT platforms)

***

--js-runtimes node

Documentation says it is not required to download most from YouTube videos. I found it essential in almost every occasion. It makes yt-dlp execute JavaScript to work around certain YouTube changes or anti-bot mechanisms.

***

--extractor-args "youtube:player-client=web_embedded,tv,web"

Choose from which client you want yt-dlp to check for the preferred audio format. Here I choose: web_embedded, tv, web; which is a configuration that works with the initial part of the script. Most of the other ones will need the implementation of Tokens, and android and ios clients might work without cookies (not tested).

***

--format "bestaudio/best"

Look for the best audio track. If there is no audio track, look for the best video-audio combined (ffmpeg is needed to separate them).

--extract-audio

Extract the audio from the file (ffmpeg is used if installed)

--audio-format opus

Forces the convertion to the desired format (Opus). If the original audio is already an .opus (most common) it will directly be saved without quality loss. Obviusly, if it's not .opus, it will be recoded (potential quality loss).

--continue

Mhis tells yt-dlp to not stop and restart on connection lost. It should be useless on newer versions of yt-dlp but I'd keep it any case.

--format-sort "abr"

Modify the order from which we choose the best format. Give priority to the Avarage Bitrate (abr), the highest mean bitrate expressed in kbps.

***

--embed-thumbnail

Acquire the thumbnail as cover (in YT Music it's the actual cover)

--write-thumbnail

Save the Cover in the folder

--convert-thumbnails jpg

Convert it in jpg

# Crop squared 1:1 and conversion to yuv420p
--ppa "ThumbnailsConvertor+ffmpeg:-vf crop=ih:ih,format=yuv420p"

Crop the Cover in a 1:1 format.
Convert the color format into one that the majority of the portable reader can show. Note that some of the artworks may appear different, as it happened once to me, for an especially dark image.

***

--replace-in-metadata uploader "(?i)\s*-\s*Topic$" ""

Remove "- Topic" from the channell name

--parse-metadata "%(artist)s|%(uploader)s:^(?:NA|None|)\s*\|(?P<artist>.+)$"

Copy the uploader tag in the artist tag if artist is empty

--parse-metadata "%(playlist_index)s:%(track_number)s"

Assign indexes to tracks following the playlist order (as it should follow the album)

--parse-metadata "%(playlist_title)s:%(album)s"

Copy playlist title into album tag

--replace-in-metadata album "(?i)^(?:Album|EP|Single)\s*-\s*" ""
--replace-in-metadata playlist_title "(?i)^(?:Album|EP|Single)\s*-\s*" ""

Remove the prefixes: ("Album - ", "EP - ", "Single - ").
Depending on your taste you might want to, instead, keep them.

--parse-metadata "%(artist)s|%(title)s:^\s*\|(?P<artist>.+?)\s+-\s+(?P<title>.+)$"

When the track is named "artist - name_of_the_track" it chooses the first part as artist and the rest as the name of the track. This was needed in some occasions when the title of the track had " - " in the name. It shouldn't go in conflict with most artist names that have "-" in the name, as it's usually not spaced, such as: Alt-J.

--replace-in-metadata artist "(?i)\s*(?:,|\s+&\s+|\s+x\s+|\s+feat\.\s+|\s+ft\.\s+).*" ""

Force the primary artist to be the only one in the tag (removes feat., commas, etc.)

--replace-in-metadata title "(?i)\s*[\(\[]Official Music Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Official Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Official Lyric Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Official Lyrics Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Official Audio[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Lyric Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Lyrics[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Visualizer[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Visualiser[\)\]]" ""

Clean some superfluous wording.

--add-metadata

Incorporate the metadatas acquired in the file.

--windows-filenames

Force Windows rules over file name.
On Windows, for example, you can't name a file or a directory with double dots, like: Fred Again..; with this, you will find it instead as: Fred Again.# but the metadata artist will be correct.

***

-o "Libreria/%(artist,uploader)s/%(album,playlist_title)s/%(track_number,playlist_index)02d - %(title)s.%(ext)s"

Choose how you want to save the file (directory tree). Here: Artist\Album\Track.opus .

-o "thumbnail:Libreria/%(artist,uploader)s/%(album,playlist_title)s/cover.%(ext)s"

Save a copy of the cover art.

***
## Known limitations:

- Featured artists do not create separate artist folders anymore but the compromise was to simply delete them from artist tag.
- Most metadata tags are still empty or filled with the confused strings YouTube provide. Such as genre being "music" and year being seemingly random numbers.
- If downloading from YouTube and not YouTube Music the artwork quality will depend on the uploaded thumbnail.

> Comments: Limitations from the lack of YouTube metadata can be solved with a metadata editor like Mp3tag.
