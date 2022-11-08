# LiveArea Images specifications
This document will guide you through making your LiveArea assets for Vita.

## Contents
- [Prerequisites](#prerequisites)
- [Files structure](#files-structure)
    * [`icon0.png`](#icon0png)
    * [`pic0.png`](#pic0png)
    * [`bg0.png`](#bg0png)
    * [`startup.png`](#startuppng)
    * [`template.xml`](#templatexml)
- [`CMakeLists.txt`](#cmakeliststxt)
- [Contribution](#contribution)

## Prerequisites
First of all, we need to prepare our tools. These "tools" are easier to install in Linux environment,
but you might try it in MSYS2 on Windows or maybe even on the Windows itself, anyway, it's your choice.

We need to install following tools:
- FFmpeg;
- pngquant.

I hope you can install them in your system without direct instructions, so we'll move on. 

## Files structure
Well, we need to know files hierarchy first and only after that analyze them in detail.

At the end we should get a directory `sce_sys` that contains:
```
sce_sys
│   icon0.png
│   pic0.png
│
└───livearea
    └───contents
            bg0.png
            startup.png
            template.xml
```
Where:
- [`icon0.png`](#icon0png) - little icon of the app. Resolution - `128x128`, always shown rounded in circle.
- [`pic0.png`](#pic0png) - picture that shown when app is loading. Resolution - `960x544`, i.e. it shown fullscreen.
- Directory `livearea/contents`:
    * [`bg0.png`](#bg0png) - background of the "paper" that shown when you tapped on icon, but didn't start the app yet. Resolution - `840x500`. Take care of the next file `startup.png` that shown in the center or on the right (it depends on what you did chose in `template.xml`).
    * [`startup.png`](#startuppng) - picture that shown above the "Start"/"Resume" button inside the "paper" menu. Resolution - `280x158`, but take care of that "Start"/"Resume" button in the bottom. Natively supports alpha channel, i.e. transparency.
    * [`template.xml`](#templatexml) - markup file that indicates what elements and where to place them in the "paper" menu (it can even also support links to websites, but I don't know how to do it yet, maybe this guide will appear here later).

### `icon0.png`
Let's say you did some picture for icon and named it `source_icon0.png`.

> Also, though icon in PS Vita system showed as rounded in circle, your icon0.png
> can't have alpha-channel (transparency). As far as I know, only layer supporting 
> alpha-channel is `startup.png`.

To make this picture "good" for your Vita, run these commands:
```bash
ffmpeg -i source_icon0.png -vf scale=128:128:flags=neighbor -pix_fmt ya8 middle_icon0.png
pngquant middle_icon0.png -o icon0.png
rm middle_icon0.png
```
Where:
- `:flags=neighbor` is used for pixel-perfect scaling (works really perfectly when resolution of the source image is multiple or divisor of output image, in our case `128x128`), remove it if your icon isn't related somehow to pixel-art, then you can also google other FFmpeg scaling flags.
- `-pix_fmt ya8` - make our PNG with 8-bit integer precision, it's very important step.
- `pngquant` will also index colorspace of image for us, and after that we'll get a "good" image for PS Vita.

### `pic0.png`
`pic0.png` is kinda different. It doesn't require `pngquant`, and, otherwise, you'll get an error if you run it on this pic. While other pics require 8-bit precision and little color indexing that we can get with `pngquant` (like three per image - gray, blue, white), it does require **strictly** 256 colors in index. To achieve this we can use FFmpeg's `palettegen` and `paletteuse`.

Let's imagine you have image named `source_pic0.png`.
Run these commands:
```bash
ffmpeg -i source_pic0.png -vf scale=960:544:flags=neighbor -pix_fmt ya8 middle_pic0.png
ffmpeg -i middle_pic0.png -vf palettegen palette.png
ffmpeg -i middle_pic0.png -i palette.png -filter_complex "paletteuse" -c:a copy pic0.png
rm palette.png middle_pic0.png
```

### `bg0.png`
Let's move to the `livearea/contents` directory (literally):
```bash
mkdir livearea
mkdir livearea/contents
cd livearea/contents
```

Let's imagine you have image named `source_bg0.png`.
Run these commands:
```bash
ffmpeg -i source_bg0.png -vf scale=840:500:flags=neighbor -pix_fmt ya8 middle_bg0.png
pngquant middle_bg0.png -o bg0.png
rm middle_bg0.png
```

### `startup.png`
Let's imagine you have image named `source_startup.png`.
Run these commands:
```bash
ffmpeg -i source_startup.png -vf scale=280:158:flags=neighbor -pix_fmt ya8 middle_startup.png
pngquant middle_startup.png -o startup.png
rm middle_startup.png
```

### `template.xml`
Let's fill our template file:
```xml
<?xml version="1.0" encoding="utf-8"?>

<livearea style="a1" format-ver="01.00" content-rev="1">
    <livearea-background>
        <image>bg0.png</image>
    </livearea-background>

    <gate>
        <startup-image>startup.png</startup-image>
    </gate>
</livearea>
```
Where:
- `style` - style of the "paper" menu. Can be (as far as I know):
    * `a1` - `startup.png` shown at the center.
    * `psmobile` - `startup.png` shown at the right.

## `CMakeLists.txt`
Congratulations! We've did all the pics. Now we should link them for the compilation.

Only one thing: I won't talk about `CMakeLists.txt` and compilation process too much, here are only things you need to do for LiveArea assets in the [already inited project](https://vitasdk.org/).

You need to add following lines to your `vita_create_vpk()` function at the end of the `CMakeLists.txt` file:
```
FILE sce_sys/icon0.png sce_sys/icon0.png
FILE sce_sys/pic0.png sce_sys/pic0.png
FILE sce_sys/livearea/contents/bg0.png sce_sys/livearea/contents/bg0.png
FILE sce_sys/livearea/contents/startup.png sce_sys/livearea/contents/startup.png
FILE sce_sys/livearea/contents/template.xml sce_sys/livearea/contents/template.xml
```

So this function will look like this:
```
vita_create_vpk(${CMAKE_PROJECT_NAME}.vpk ${VITA_TITLEID} ${CMAKE_PROJECT_NAME}.self
    VERSION ${VITA_VERSION}
    NAME ${VITA_APPNAME}

    FILE sce_sys/icon0.png sce_sys/icon0.png
    FILE sce_sys/pic0.png sce_sys/pic0.png
    FILE sce_sys/livearea/contents/bg0.png sce_sys/livearea/contents/bg0.png
    FILE sce_sys/livearea/contents/startup.png sce_sys/livearea/contents/startup.png
    FILE sce_sys/livearea/contents/template.xml sce_sys/livearea/contents/template.xml
)
```
> You can change `sce_sys` at the beginning to another path leading to your LiveArea assets.
> But don't change `sce_sys` at the end of the line - it's destination, it's constant.

Now try to compile it and launch on your Vita. Is everything OK? If not, you're free to [open issue](https://github.com/Hammerill/livearea-specs/issues).

## Contribution
I hope this document helped you. If you appreciate my work, please look at my game [Sand-Box2D](https://github.com/Hammerill/Sand-Box2D) (still in very early development) and, idk, leave a star there or something.

<sub>It looks like I should make auto script of this all, but will I?..</sub>

Thank you for your attention!
