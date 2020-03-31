---
title: "Viewing xkcd comics in RStudio"
date: 2019-10-10T18:28:13-05:00
tags: ["R"]
---

{{< figure src="https://imgs.xkcd.com/comics/t_distribution.png" >}}

I love [the webcomic xkcd](xkcd.com), and I look forward to seeing the new comic whenever. The problem is, sometimes I'm forgetful and I forget to check the website for the latest comic (or sometimes comics if I'm really behind). But luckily for me, I do use RStudio everyday. This made me realize that "hey, maybe I should make something so I can able to view xkcd in RStudio." So I did.

## But how?

The writer of xkcd, Randal Munroe, makes it really easy to get data for all of his comics. He's made an API that allows for queries for every comic he's posted on xkcd's website, and formats it in an extremely simple JSON. The JSON contains meta data for the comic's number, title, a link to the image file, and some other useful things. Using this API, I wrote an R script (and threw it in my R package stoic) that allows you to browse xkcd comics right in the RStudio plot viewer. The code for the script can be found [on my GitHub.](https://github.com/joestoica/stoic/blob/master/R/xkcd.R) This function takes either a number, which specifies the comic's unique ID number, or a boolean asking whether you want to view a random comic. It will return the most recent comic if no arguments are specified. After the function runs, it allows you to go to the previous or next comic ID, or another random comic. There are a few comics that don't work in the function (the 404 page has comic ID 404, and I don't know how to add .gif compatibility). But for the most part, you can view all the comics! If you want to try it, install my package through GitHub or just save the function to your global environment through copy/paste.
