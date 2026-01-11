---
title: How to convert images into webp format
description: Learn how I convert my photos from my iphone or other image formats into webp files optimized for this git website.
date: 2026-01-11 12:00:00:01 +/-0000
categories: [Linux, Optimize]   # Example: [Networking, Homelab]
tags: [cwebp, heic, website, jpeg]    # Add relevant keywords
toc: true
comments: true
#image:
# path: '/assets/img/headers/example.jpg'
# alt: 'Terminal screen showing Git initialization and configuration steps'
 # lqip: true   # Optional low-quality image placeholder (Chirpy supported but it broke my Git push)
pin: false
published: true
# author: "Kevin Feiertag"
#permalink: /posts/Git/
seo:
  title: "How to convert images into webp format"
description: "Learn how I convert my photos from my iphone or other image formats into webp files optimized for this git website."
#  canonical: "https://wangineer.com/posts/Git/"
  keywords: "cwebp, heic, website, jpeg"


---
## Scenario
I often need to convert images from "jpeg" or "heic" to webp to save space on this git website.  Here is how to use ***cwebp***.  If you need to convert webp files to some other format, look into ***dwebp***

 - cwebp — converts images to WebP
 - dwebp — converts WebP to other formats

## The Install Process

**Step 1**: Install epel-release
```
sudo dnf install epel-release
```

**Step 2**: Almalinux9 do below
```
sudo dnf config-manager --set-enabled crb
```

**Step 2b**: Almalinux8 do below
```
sudo dnf config-manager --set-enabled powertools
```

**Step 3**: Refresh your Metadata
```
sudo dnf makecache
```

**Step 4**: Install webptools
```
sudo dnf install libwebp-tools
```

**Step 5**: Verify verison
```
cwebp -version
```

## Install the ***heif*** tool to convert **heic** formats (Apple/iphone images)
```
sudo dnf install libheif libheif-tools
```

## How to convert the images

To convert
 - First line converts ***heic*** to ***png***
 - Second line converts ***png*** to ***webp***
```
heif-convert input.heic temp.png
cwebp temp.png -o output.webp
```
Or you can use it via one line
 - This removes the .png file that gets created in the middle
```
heif-convert input.heic temp.png && cwebp temp.png -o output.webp && rm temp.png
```

Below is with reduced quality (-60)

```
heif-convert IMG_2511.HEIC temp.png && cwebp -60 temp.png -o IMG_2511.webp && rm temp.png
```

## Step 4: Optimize for Chirpy Theme (1200x630)
If I am needing the photo to use as a header for a post, I will need to format the dimensions to the below size.

```
1200x630
```