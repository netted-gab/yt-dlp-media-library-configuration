Summary:

A yt-dlp configuration for downloading music albums from YouTube or YT Music with:

- preferred audio format (e.g. m4a)
- embedded cover art
- embedded metadata
- cleaner titles
- artist/album folder organization
- playlist track numbering

***

The repository provides a documented yt-dlp configuration for downloading and organizing music from YouTube and YouTube Music. It's intended as a starting point or a blueprint you can tailor to your or your machine's needs.

***

## Requirements:

- yt-dlp
- FFmpeg

***

## Optional:

- Firefox (if using `--cookies-from-browser firefox`)
- Node.js (if using `--js-runtimes node`)

note on Node.js: It is not required to download from most YouTube videos or playlists, but I found it useful in multiple occasions when yt-dlp needed to execute JavaScript to work around certain YouTube changes or anti-bot mechanisms.

***

## YouTube PO Tokens

Some YouTube clients may require a GVS PO Token. When this happens, yt-dlp may report errors such as HTTP 403 or `Requested format is not available`.

This configuration supports automatic PO Token generation through the `bgutil-ytdlp-pot-provider` plugin and its `script-node` method.

### Additional requirements

- bgutil-ytdlp-pot-provider
-  Node.js 20+ (if using `--js-runtimes node` and/or the bgutil PO Token configuration)
- bgutil-ytdlp-pot-provider (if YouTube requires a GVS PO Token)

The `generate_once.js` script provided by bgutil is used to generate the required PO Token when needed. The plugin itself is kept separate from yt-dlp and must be installed/configured according to the bgutil project instructions.

### Configuration

The following options are included in `yt-dlp.conf`:

```text
--extractor-args "youtubepot-bgutilscript:script_path=PATH_TO_BGUTIL\server\build\generate_once.js"
--extractor-args "youtube:player-client=mweb"
```

### IMPORTANT:
PATH_TO_BGUTIL is a placeholder. Replace it with the actual path to your bgutil-ytdlp-pot-provider directory on your machine.

***

## Installation

Copy `yt-dlp.conf` wherever you want to keep your yt-dlp configuration.

***

## Usage

```bash
yt-dlp --config-location yt-dlp.conf URL
```

If placed in the same folder as yt-dlp executable, and depending on the version, you may not need to call the configuration. If you store the configuration file in another folder you'll need to call for it.

***

Example output:

```text
Artist/
└── Album/
    ├── 01 - Song.m4a
    ├── 02 - Song.m4a
    └── cover.jpg
```

This is how the file should appear in the folder. Check Screenshot_1.png.
It will create a main folder named "Artist name", a subfolder named "Album name", and the whole playlist you downloaded inside.

note: the actual cover will be temporarily stored only. So, if you look inside the folder before the download is completed you may see it being saved (as a jpg file) and then disappear. It's actually in the metadata of each one of the tracks. (you can check with metadata editor like Mp3tag)

***

--cookies-from-browser firefox

YouTube uses cookies to identify browser sessions and reusing your own browser cookies allows yt-dlp to make requests with the same authentication you have. It basically allows access to content your account is permitted to view. (If you are not logged in it's still using the ones you are storing as a temporary user of YT platforms)

***

--js-runtimes node

As stated before you don't need this for the majority of downloads.

***

--format bestaudio

select the best audio quality available from the file we are about to download.

--extract-audio

extract said audio.

--audio-format m4a

this defines the new format we want to save the audio into. This is where you want ffmpeg installed to make it work.

--continue

this tells yt-dlp to not stop and restart on connection lost. It should be useless on newer versions of yt-dlp but I'd keep it any case.

--no-overwrites

this prevents files that are in the same folder with the same name to be overwritten.

***

--parse-metadata "playlist_index:%(track_number)s"

As explained in the conf file, it simply copies the track position in the playlist into the metadata tag: track, which namely is the track number.
We trust YT on placing them in the correct order as the Artist intended.

```bash
--replace-in-metadata title "(?i)\s*[\(\[]Official Music Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Official Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Official Lyric Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Official Lyrics Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Official Audio[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Lyric Video[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Lyrics[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Visualizer[\)\]]" ""
--replace-in-metadata title "(?i)\s*[\(\[]Visualiser[\)\]]" ""
```

These options remove common phrases used in youtube titles. Such as "Official Video" or "Lyric Video".

--add-metadata

This tells yt-dlp to embed the metadata to the file we are downloading.

--embed-thumbnail
--convert-thumbnail jpg

This acquires the thumbnail and the next line converts it as jpg.

--windows-filenames

This is for windows users or if you are downloading on a hard drive with NTFS or FAT32/exFAT formatting. It checks the file for characters that aren't welcome in windows and may stop your storing or break the file. Linux and Mac have more flexibilities on this and shouldn't need the line at all.

***

-o "%(artist,uploader)s/%(album,playlist_title)s/%(track_number,playlist_index)02d - %(title)s.%(ext)s"

This is where we stop. It's how and where the file is going to be stored with its metadata.

the metadata tag filling is decided like this:

- artist, is going to be the uploader of the file, and the main folder
- album, is going to be the playlist title, and the subfolder
- track number, is going to be the position (index) of the track in the playlist, and the first part of the file name. After that it places " - " and the title written in the title tag.

You can check Screenshot_1.png, taken on File Explorer in W11.

***

## Known limitations:

- Featured artists may create separate artist folders.
- Genre metadata depends on what YouTube provides.
- Album artwork quality depends on the uploaded thumbnail.

> Comments: most of the limitations come from the lack of YouTube metadata. This is most easily solved with a metadata editor like Mp3tag once we have gathered all we could from YT. The yt-dlp configuration file then provides a solid base to work on for track libraries, especially big ones.
> The first problem (the featured artists one) can be tackled by renaming the main folder with the main artist name, at least on Windows 11, essentially merging the folders in one, combining the files into the expected directory structure.
