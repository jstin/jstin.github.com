---
layout: default
title: Install ImageMagick on Ubuntu
---

<h1>{{ page.title }}</h1>

If you need ImageMagick, install prerequisites:

    sudo apt-get install libperl-dev gcc libjpeg62-dev libbz2-dev libtiff4-dev libwmf-dev libz-dev libpng12-dev libx11-dev libxt-dev libxext-dev libxml2-dev libfreetype6-dev liblcms1-dev libexif-dev perl libjasper-dev libltdl3-dev graphviz gs-gpl pkg-config
    
Download and build:
    
    cd /tmp
    wget ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick.tar.gz
    tar xvfz ImageMagick.tar.gz
    cd ImageMagick-6.6.7-0/
    ./configure
    make
    sudo make install
    sudo ldconfig