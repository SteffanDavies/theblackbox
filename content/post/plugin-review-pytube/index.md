---
title: "Library Review: pytube"
description: Exploring the pytube library for working with youtube content in python 
date: 2023-12-18
image: cover.jpg
tags: 
    - pytube
    - pytubefix
    - youtube
categories:
    - python
    - libraries
    - reviews
comments: true
draft: false
---

## Introduction

The [pytube](https://github.com/pytube/pytube) library for Python3 is a popular toolbox for accessing YouTube content. One of it's main quirks is that it does not rely on any third-party dependencies, while providing a number of simple tools for querying and downloading videos, audio-tracks and captions.

While pytube has proven to be a useful library for many people (as of writing it has over 2k forks and 9.7k stars on GitHub), some recent changes at YouTube have left it largely broken.

To make things worse, the project seems to be all but [abandoned](https://github.com/pytube/pytube/issues/1786) by it's maintainers, despite still having an active (and desperate) community sharing their frustrations in the Issues tab of the repository, and providing each other with [workarounds](https://github.com/pytube/pytube/issues/1626#issuecomment-1670435112). The [documentation](https://pytube.io/en/latest/) is also [outdated](https://github.com/pytube/pytube/pull/1361), asking the user to use deprecated functions in the tutorial.

Due to the growing discontent with the repositories' stagnation, part of the community forked the repo to create [pytubefix](https://github.com/JuanBindez/pytubefix), which solves a number of issues. Perhaps the maintainers of pytubefix expect their fork to eventually be merged back into the original pytube repo, but I'm skeptical this will happen soon.

For this reason, the examples listed in the article will use the pytubefix library as it is identical to pytube.

## Instalation


To start, we will be using a clean conda environment and install Python 3.12 and Jupyter Lab as our coding environment.

Conda is a popular package management solution for Python as it allows the user to easily create independent environments containing their own packages, avoiding conflicts between project dependencies. A guide for installing on Linux can be found [here](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html).

Once we have installed conda, we can create a new environment and install Jupyter Lab from the conda-forge channel. We will then add our python kernel from this conda environment to Jupyter.

```sh
conda create -n pytube
conda activate pytube
conda install -c conda-forge jupyterlab
ipython kernel install --user --name=pytube
```

Jupyter Lab is part of Project Jupyter and allows for the creation of interactive coding projects. Now that is installed we will create a folder for our project and launch Jupyter Lab:

```sh
mkdir pytube
cd pytube
jupyter lab
```

Now you should have a Jupyter Server running and your default browser should open in the corresponding localhost address. If not, check your console output for the link.

In the Jupyter interface launcher tab, under the Notebook section, click on *pytube* to launch a new Jupyter Notebook.

## Install and import libraries

Now that we have our Jupyter Notebook running, we should install pytube and import all our libraries for the project. Let's start by installing pytubefix.

```python
import sys
!{sys.executable} -m pip install pytubefix
```

Notice how we use the `!` before running our command. This instructs IPython to run a shell command instead of python code. `{sys.executable}` is our Python 3.12 kernel from the pytube conda environment. Installing directly to a `python` or `python3` keyword runs the risk of summoning our base python kernel instead, which we don't want.

Let's go ahead and import the libraries:

```python
from pytubefix import YouTube, Playlist, Channel, Search
from IPython.display import Image, Video, Audio
```

In case you are wondering, the `IPython.display` module provides `Image` and `Video` classes for visualizing each of these types of media within our Jupyter Notebook.

## Thumbnails

Retrieving video thumbnails is easy in pytube. We start by instantiating a YouTube object using the video URL. This object will contain a number of properties such as the video title:

```python
yt = YouTube('http://youtube.com/watch?v=2lAe1cqCOXo')
yt.title
```

The thumbnail can be presented using a single line of code, by leveraging the Ipython image class. Here we pass the thumbnail URL of the YouTube object and set an acceptable width:

```python
Image(url=yt.thumbnail_url, width=512)
```

## Videos

### Obtaining video info

We can work with a video by instantiating a `Youtube` object while passing the video url. Here we will instantiate the object and access it's properties:

```python
video = YouTube('http://youtube.com/watch?v=2lAe1cqCOXo')

title = video.title
views = video.views
url = video.watch_url


print(f"{video.title} - {video.watch_url} ({video.views:,} views)\n")
print(f"Author:\t{video.author}")
print(f"Date:\t{video.publish_date}")
print(f"\nDescription:\n{video.description}")

```

### Gathering streams

When downloading, you have to choose between <abbr title="Dynamic Adaptive Streaming over HTTP">DASH</abbr> (adaptive) and Progressive content streams for video, and an audio-only stream.
- **DASH**: Highest quality. However, audio and video are separate and must be joined using <abbr title="Fast Forward Moving Picture Experts Group">[FFmpeg](https://ffmpeg.org/)</abbr> or another tool.
- **Progressive**: Not as high quality. This is the "old" way of Youtube providing content. Audio and Video are together in the same file.

To make things simple, we will use the progressive stream as an example:

```python
adaptive_streams = video.streams.filter(adaptive=True)
progressive_streams = video.streams.filter(progressive=True)

print("Adaptive streams (DASH):\n")
for stream in adaptive_streams:
    print(stream)

print("\n\nProgressive streams:\n")
for stream in progressive_streams:
    print(stream)
```
In this case, there is a 360p and a 720p progressive stream available. We can obtain some metadata from the 360p stream properties:

```python
stream = video.streams.get_by_itag(18)

print(f"Title:\t\t{stream.title}")
print(f"Filename:\t{stream.default_filename}")
print(f"Bitrate:\t{stream.bitrate}")
print(f"Filesize:\t{stream.filesize_mb} MB")
```

### Downloading a video

Let's download the 360p version:

```python
stream.download(output_path="./downloads/", filename="360p.mp4", filename_prefix="test_")
```

One of the reasons why working in Jupyter is so great, at least for prototyping, is because we can view our work immediately:

```python
Video("./downloads/test_360p.mp4", width=512)
```

## Retrieving Captions

### Downloading Captions

Grabbing the media captions is just as easy. All we have to do is filter the `captions` property of our Youtbe object by language code (it's a dictionary-like object), and then call then use the `save_captions` function to write the result to a file (note that they are in .srt style).

```python
video.captions["en"].save_captions("captions_en.srt")
```
### Listing All Languages

If you are not interested in English captions, you can list the available languages by using the `caption_tracks` property:

```python
video.caption_tracks
```

## Playlists

### Getting Playlist Data

It's also possible to retrieve Playlists and their contents using pytube. Let's go ahead and see some of the information we can print out:

```python
playlist = Playlist('https://www.youtube.com/playlist?list=PLzH6n4zXuckoUWpzSEpQNW6I8rXIzyi8w')

print(f"\
Playlist title:\t\t{playlist.title}\n\
View count:\t\t{playlist.views:,}\n\
Playlist owner:\t\t{playlist.owner}\n\
Video count:\t\t{playlist.length:,}\n\
\n{playlist.description}\n\
")
```

>Playlist title:		Pong, Python & Pygame
>
>View count:		29,058
>
>Playlist owner:		Computerphile
>
>Video count:		4

>Squash-Pong created in python by Dr Isaac Triguero

### Listing Playlist Videos

As we can see, useful information is stored in the Playlist object's properties. From here, we can create separate lists of all the videos, titles and URLs and zip them up into a tuple:

```python
titles = [video.title for video in playlist.videos]
views = [video.views for video in playlist.videos]
urls = playlist.video_urls

videos = list(zip(titles, urls, views))

for video in videos:
    print(f"{video[0]} - {video[1]} ({video[2]:,} views)")
```

>Pong, Python & Pygame 00 - Computerphile - https://www.youtube.com/watch?v=JRLdbt7vK-E (89,593 views)
>
>Pong, Python & PyGame 01 - Computerphile - https://www.youtube.com/watch?v=hHtb-Ohyfu8 (60,029 views)
>
>Pong, Python & Pygame 10 - Computerphile - https://www.youtube.com/watch?v=Nk3Och0I4ZY (37,610 views)
>
>Pong, Python & PyGame 11 - Computerphile - https://www.youtube.com/watch?v=VyrAVNoEf0g (36,951 views)
>
> — <cite>Output</cite>

### Downloading Video playlists

Finally, we can download the playlist contents using the `download` function. In this case, we will retrieve the video streams with the highest quality possible:

```python
for video in playlist.videos:
    video.streams.get_highest_resolution().download(output_path="./downloads/")
```

### Downloading Audio

If we are only interested in the audio, we can fetch the audio track separately:

```python
for video in playlist.videos:
    video.streams.get_audio_only().download(mp3=True, output_path="./downloads/")
```
Now let's listen:

```python
Audio("downloads/Pong, Python & Pygame 00 - Computerphile.mp3")
```

## Channels

### Listing Content

If we are looking for more than just a playlist, and need all the content from a channel, this is also possible:

```python
channel = Channel("https://www.youtube.com/@Computerphile/")
channel.channel_name

```
Considering this is a large channel, let's list the names of the first 5 videos in 2021:

```python
for vid in channel.videos[:5]:
    print(vid.title)
```

### Filtering

We could even filter videos of a channel based on, for example, the day, month or year they were published, as pytube provides `datetime` properties for the videos. Note that sequentially iterating all the videos of a channel is an extremely slow process.

```python
videos = [video for video in channel.videos if video.publish_date.year==2022 if video.publish_date.month==11]
titles = [video.title for video in videos]
```

## Searching

### Searching YouTube

Pytube also contains a search functionality.

```python
    search = Search("Harvard CS50")

    print(f"""Results for {search.query} query:\n""")

    for result in search.results:
        print(result.title)
```
The search results are limited to avoid infinite loops. We can get more results using the `get_next_results` function, which will append more results to the `results` property.

```python
    search.get_next_results()
    for result in search.results():
    print(result.title)
```

### Completion suggestions

Search completion suggestions can also be obtained using the `completion_suggestions` property:

```python
    search.completion_suggestions
```

## Conclusion

Pytube is a cute little library for searching and downloading YouTube content. It's incredibly simple and easy to use. However, it's current state is almost **unusable** due to internal changes at YouTube breaking the code. In the meantime, pytubefix is recommended until issues are resolved.
