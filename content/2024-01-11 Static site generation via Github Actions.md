Title: Static Site Generation with Pelican and  Github Actions
Date: 2024-01-09 10:34
Category: WebDev
Tags: webdev, software
Authors: Sen
Summary: A rundown on how I built this site, using Pelican (a Python-based static site generator) and Github Actions. Yes, this Geocities-looking crap is more modern than it seems.

Despite the "a e s t h e t i c s" of this site, I do like playing with modern technologies. The look of my site doesn't change much (how can you improve on <a href="/the-www-peaked-in-1999.html">perfection?</a>) but the underlying tech building/hosting it changes all the time as I learn and play with new things.

This current iteration of the website is hosted on my own server so that I can play around static site generators. The posts and images are pushed to Github, and then a Github action copies them to my server via SCP and trigger a bash script when the copy is complete. That bash script does some filtering/formatting of the files, and then triggers the Pelican static site generator. Why Pelican? I dunno, it just looked cool and I wanted to play with it. Previous versions of this site used Jekyll but I like how much simpler Pelican is.

First you need to <a href="https://docs.getpelican.com/en/latest/quickstart.html">install Pelican</a> itself, which is just:

    :::bash
    python -m pip install "pelican[markdown]"

Then create a folder for your site, and run the quickstart script:

    :::bash
    mkdir -p ~/projects/yoursite
    cd ~/projects/yoursite
    pelican-quickstart

Once you have the basic site running on your server and can build it manually, you can set up the Github side of things.

The Github Action to copy posts/images from Github to the server is pretty simple. It has one action to copy the files over, and then another action to run the Bash script sitting on my server that then runs Pelican to rebuild the site. 

Set up your host/username/port/key in your Repo settings, then stick a file called deploy.yml in your Github repository in */.github/workflows* with the following:

    :::yaml
    name: Deploy

    on: [push]

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v1

          - name: Copy repository contents via scp
            uses: appleboy/scp-action@master
            with:
              HOST: ${{ secrets.HOST }}
              USERNAME: ${{ secrets.USERNAME }}
              PORT: ${{ secrets.PORT }}
              KEY: ${{ secrets.SSHKEY }}
              source: "."
              target: "/home/username/path/to/files/"

          - name: Executing remote command
            uses: appleboy/ssh-action@master
            with:
              host: ${{ secrets.HOST }}
              USERNAME: ${{ secrets.USERNAME }}
              PORT: ${{ secrets.PORT }}
              KEY: ${{ secrets.SSHKEY }}
              script: "/home/username/buildpelican.sh"

If you have a default Pelican install, then the Bash file is as simple as:

    :::bash
    source "/home/username/virtualenvs/pelican/bin/activate"
    cd "/home/username/path/to/files//"
    pelican content -s pelicanconf.py

After all that, you write your posts locally on your computer, push them to Github, and the rest happens automatically. I love how simple this makes it to post to my site as it's no different to just writing locally. No mental load of "publishing" to the site, or logging in to do edits. Just write, hit save, push to Github.

Does that mean I'll actually post more now? ... lol.
