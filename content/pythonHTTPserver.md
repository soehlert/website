## Problem
I was trying to get BittorrentSync set up on my home server running Centos 6.6, but the BTSync site is set up strangely and I couldn't copy the download link in order to do a quick wget on my server. So that leaves me with the tar.gz on my laptop.

## Attempts
Ok I have a few ways to proceed. My first inclination was scp that tarball over to my server and go from there. Something seems to be wrong with my key though..hmm, maybe I can do something more fun. I could also install X and download the tarball directly on there. That seems so...wrong. Wait a second, I got an idea.

## Solution
This won't be ground breaking to everyone, but it was still kind of fun. I just fired up a quick http server using python!

```python -m SimpleHTTPServer```

This starts a small webserver on port 8000. Then I sshed to my server (which worked for some reason with my key) and did a quick wget. There we go!