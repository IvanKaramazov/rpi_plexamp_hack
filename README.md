# rpi_plexamp_hack

This repo offers a modified version of the server.prod.js beta code distributed by Plex. I have made a few changes to get the code to work with current Plex infrastructure, but, NB: 1) there may well be other existing bugs in the code that I have not encountered, and 2) new buys may appear at any time. So no guarantees.

I have done nothing malicious to this file, only the bare minimum code changes I could identify that were required to get it running reliably. However, the file is 85K lines long as prettified automatically by my code editor (JsFormat package in Sublime Text 3 is what I used), so you'll probably have to take my word on that. Up to you.

## Changes made
* Formatted file using JsFormat in Sublime Text 3. As deliveverd from Plex, the file is minified and loaded all in one line, which is of course a nightmare for editing. I have left the line formatting introduced by this tool in place in case future edits are required. It may be introducing some bugs (in at least one case it did with a formatted string definition, though I fixed it). This formatting also increases the file a small amount thanks to newline and space characters, but I don't think it really mattes.
* Converted the hostname of the media providers endpoint accessed in the fetchProviders method (defined at line 445 in this file) to constant which can and must be set by the user at the top of the file (see instructions below). Also modified the code that process the media provider list to work with the output found at this endpoint, which is slightly different from whatever the script expected before.
* Converted the hostname where plexamp attempts to communicate with the local MPD service to a variable which can be set by the user at the top of the file. This may not be necessary for everyone, but on my Rpi 0 running Raspbian Lite the default behavior (looking on 127.0.0.1) failed, so I changed it to "localhost". Probably it would be better to set this on the MPD end but I couldn't get it to work. The modified method is `connect`, defined at line 25244 in my server.prod.js.
* Short-circuited the `appendTrackWithLoudness` method (defined at line 25368 in this file) so that it simply usese the plain `appendTrack` method defined above it. Plexamp was hanging for me on many attempts to connect certain tracks, and this method was generating the the error in the logs, saying the "addidwithloudness" command was unrecognized. This means that Plexamp's track loudness smoothing features are disabled on this player, presumably, but that was the simplest means of getting around the issue that I could find. Possibly the issue has to with bugs introduced by the file's prettification but I don't know.

## Instructions
1. Get a working server.json file. Canonical instructions say to used an old Plexamp v1 or v2 installation to generate this, but it only worked partially for me. The file I got from that had only the `player` and `user` info defined in the JSON, but a third `server` object is required. Your final version should look like this:  
```{
  "player": {
    "name": "YOUR_SERVER_NAME",
    "identifier": "XXXXXXXXXXXXXXXXXX"
  },
  "user": {
    "id": YOUR_ID_INTEGER,
    "token": "XXXXXXXXXXXXXXXXXX"
  },
  "server": {
    "identifier": "XXXXXXXXXXXXXXXXXX",
    "library": "/library/sections/X"
  }
}
```  
 Again, the player and user sections should come through just fine. Server I had to create by hand. To get your server identifier, get an X-Plex-Token (instructions available via Google) and use it to visit the root address of your Plex server, likely at http://YOUR_SERVER_IP:32400/?X-Plex-Token=XXXXXXXXXXXXXXXX. Your server id will be contained in the `machineIdentifier` attribute of the top-level `MediaContainer` tag in the XML that you encounter. To get the library info, you need to use the same token to visit http://YOUR_SERVER_IP:32400/library/sections?X-Plex-Token=XXXXXXXXXXXXXXXX. There find the `key` integer value within any `Directory` tags living in the tree under the `MediaContainer` tag. Replace the X at the end of `"/libaray/sections/X"` string in the above example with this value. I have not tested how to deal with multiple libraries on the server yet, but I'm hopeful that simply making a list of library sections will work. Will update when I have more info.  
 Once you have added the server definition to your server.json, save it in the same spot the canonical instructions indicate on your RPi.

2. Download or copy and paste the contents of the server.prod.js file in this repo to your RPi. The existing server.prod.js should be in ~/plexamp/server/. I would recommend renaming it to server.prod.js.old and then replacing it with the new file you've created.
3. Get your server's hostname and place it in line 2 of the file, in the `plex_server_hostname` definition. To find this, visit your server's web interface, find any track and hit "Get Info" on its three-dot menu, then click "View XML" in the box that pops up. You will visit a URL like this: `https://192-168-1-X.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.plex.direct:32400/library/metadata/22765?checkFiles=1&many_other_variables=blahblahblah`. We care about the protocol and _hostname_ and port here, nothing else, i.e. everything up through the port definition at ":32400". Copy that; do not copy anything starting with "library". Paste this in place of the similar string in line 2 of the server.prod.js file you're creating. No trailing slash after the port, please.
4. [OPTIONAL] Figure out what address your RPi's MPD instance is listening on and set that in line 4 of this file, in the `mpd_ip_address` definition. I've left the default of "127.0.0.1" in place, but on my machine I had to update it to "localhost", which worked fine.
5. Save the file, make sure this version is in the right place and has replaced the old file distributed by Plex.
6. `sudo systemtctl restart plexamp`
7. If it doesn't work for you, please reach out and let me know.
