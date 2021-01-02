---
layout: post
title:  "Getting Started with a Fancy PowerShell Prompt"
date:   2020-05-09 18:30:34 -0600
categories: tech tutorials powershell
description: "Using oh-my-posh and NerdFonts to Make a Better Prompt"
---

I started out by doing this the hard way. That didn’t work out well, and so I tried again while writing this post. I again didn’t get things right. But while I was searching for help I landed on [Scott Hanselman’s post][shanselman-prompt] about using posh-git with on-my-posh. I gave in, did things his way, and it was dead simple.

Here’s how I like my prompt to look at the moment:

![Original Prompt](/assets/Fancy-PSPrompt-old.png)

> Check out [Steve Lee’s ps profile gist][steve-lee-gist] for most of the code I used for this.

It shows me the date and time, and my present working directory. It shows the time the last command took to complete, and if I’m in a git path, the current branch. The green PS turns also red if the last command had an error.

I keep seeing the fancy prompts around and I want to get in on that too. So lets get started!

Scott suggests we use Posh-Git and Oh-My-Posh modules. Thankfully, both of the modules are in the PSGallery. To install, all you need to do is:

```powershell
Install-module posh-git
Install-module oh-my-posh
```

Once they’re installed, nothing happens. That’s expected. What we need to do is import both modules and then load a theme. The act of loading a theme runs the scripts to configure the Prompt function.

```powershell
import-module posh-git
Import-module oh-my-posh
Set-Theme Paradox
```

I cycled through all of the included themes and decided the Paradox theme is closest to what suits my preferences. You can do the same or view examples of them on the [oh-my-posh github page][github-ohmyposh].

After all that, here’s what I get:
![New Prompt](/assets/Fancy-PSPrompt-1.png)

## Powerline and Nerd Fonts

You’ll notice those blocks in the text. The font for PowerShell 7 is set to Cascadia Code, which is nice, but it doesn’t contain those extra characters & symbols that Paradox is trying to display. It turns out that those come from a set of symbols called Powerline. If you want just those additional symbols head over to the [Cascadia Code github page][github-cascadia] and grab the CascadiaPL.ttf font file. However if you want tons more symbols and icons head over to the [Nerd Fonts site][nerdfonts] and grab your favorite font. There just so happens to be one named “[Caskaydia Cove Nerd Font][nerdfonts-cascadia]” which looks a whole lot like another font I’ve heard of…

And finally after grabbing the font above, setting it as the default font in my Windows Terminal, here’s what I’ve settled on, though I may tinker with coloring the timestamp:

![Final Prompt](/assets/Fancy-PSPrompt-2.png)

I copied the Paradox theme to a new file, added the function to display the last command execution time, as well as the full date and time. Because this is during the COVID-19 work from home time period, I’ve added the day in there too in order to keep myself sane.

-Kevin

[shanselman-prompt]:  https://www.hanselman.com/blog/HowToMakeAPrettyPromptInWindowsTerminalWithPowerlineNerdFontsCascadiaCodeWSLAndOhmyposh.aspx
[github-ohmyposh]:    https://github.com/JanDeDobbeleer/oh-my-posh
[steve-lee-gist]:     https://gist.github.com/SteveL-MSFT/a208d2bd924691bae7ec7904cab0bd8e
[github-cascadia]:    https://github.com/microsoft/cascadia-code/releases
[nerdfonts]:          https://www.nerdfonts.com/
[nerdfonts-cascadia]: https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/CascadiaCode.zip
