---
layout: post
title:  "Push Your PowerShell Profile to a Github Gist"
date:   2020-12-31 08:43:00 -0600
categories: tech tutorial powershell
---

A while ago in [this post][fancy-prompt] I mentioned I adopted [Steve Lee's code][stevelee-gist] for my PowerShell Profile. Yesterday I wrote some more code to let me push my local updates back into the gist where I store it.

## Why store your profile in a gist?

It's maybe not a common need. I use & write PowerShell on my personal devices, and they stay in sync using OneDrive. I also use & write (much more) PowerShell on my work devices, and they stay in sync using OneDrive for Business. I'd really like to keep my profiles in sync between those two domains. That's where storing in Github becomes handy. And a Gist is perfect for a single file that isn't part of a larger project.

## Current state

I've made several modifications to Steve's original code to suit my needs. I still had functions built in that only worked in _Windows PowerShell_ so my profiles diverged once I made changes to work in PowerShell v7. I now have a v5 profile that uses 'latest_profile_version' and my v7 profile uses 'latest_profile7_version' to track the remote version of my profile.

Here are the base variables:

```powershell
$gistID            = "0a4222500a867075e5c69a438e0f0740"
$gistUrl           = "https://api.github.com/gists/$gistID"
$latestVersionFile = Join-Path -Path ~ -ChildPath ".latest_profile7_version"
$versionRegEx      = "# Version (?<version>\d+\.\d+\.\d+)"
```

We check the remote Gist version of the file in a thread job so we don't have to wait for it to complete while the profile loads.

```powershell
$null = Start-ThreadJob -Name "Get version of `$profile from gist" -ArgumentList $gistUrl, $latestVersionFile, $versionRegEx -ScriptBlock {
    param ($gistUrl, $latestVersionFile, $versionRegEx)

    try {
        $gist = Invoke-RestMethod $gistUrl -ErrorAction Stop

        $gistProfile = $gist.Files."profile.ps1".Content
        [version]$gistVersion = "0.0.0"
        if ($gistProfile -match $versionRegEx) {
            $gistVersion = $matches.Version
            Set-Content -Path $latestVersionFile -Value $gistVersion
        }
    }
    catch {
        # we can hit rate limit issue with GitHub since we're using anonymous
        Write-Verbose -Verbose "Was not able to access gist to check for newer version"
    }
}
```

Ths code actually comes before that thread job in my profile. It effectively checks the work done by the previous session to determine if it found a newer profile in the Gist. If that's true, it offers to install it to the local copy which would take effect on the _next_ profile load.

```powershell
if (Test-Path $latestVersionFile) {
    $latestVersion  = Get-Content $latestVersionFile
    $currentProfile = Get-Content $profile -Raw
    [version]$currentVersion = "00.00.00"
    if ($currentProfile -match $versionRegEx) {
        $currentVersion = $matches.Version
    }

    if ($latestVersion -gt $currentVersion) {
        Write-Verbose "Your version: $currentVersion" -Verbose
        Write-Verbose "New version: $latestVersion" -Verbose
        $choice = Read-Host -Prompt "Found newer profile, install? (Y)"
        if ($choice -eq "Y" -or $choice -eq "") {
            try {
                $gist = Invoke-RestMethod $gistUrl -ErrorAction Stop
                $gistProfile = $gist.Files."profile.ps1".Content
                Set-Content -Path $profile -Value $gistProfile
                Write-Verbose "Installed newer version of profile" -Verbose
                . $profile
                return
            }
            catch {
                # we can hit rate limit issue with GitHub since we're using anonymous
                Write-Verbose -Verbose "Was not able to access gist, try again next time"
            }
        }
    }
    elseif ($latestVersion -lt $currentVersion) { 
        Write-Warning "Local profile is ahead of the gist version. Consider uploading with 'Update-GistProfile'."

    }
}
```

## New code

The part that I've added is the elseif to check if the local version is _ahead_ of the remote:

```powershell
    elseif ($latestVersion -lt $currentVersion) { 
        Write-Warning "Local profile is ahead of the gist version. Consider uploading with 'Update-GistProfile'."

    }
```

The __Update-GistProfile__ function code:

```powershell
function Update-GistProfile {
    $body = @{
        gist_id     = $gistID
        description = "Powershell 7 Profile"
        files       = @{
            'profile.ps1' = @{ 
                filename = "profile.ps1"
                type     = "application/x-msdos-program"
                language = "PowerShell"
                content  = "$currentProfile"
            }
        }
    }

    Invoke-RestMethod -Method Patch -Uri $gistUrl -ContentType "application/vnd.github.v3+json" -Headers @{"Authorization" = "token $env:GithubTokenGIST"} -Body (ConvertTo-Json $body)
}
```

I discovered the sub-properties to use for files->profile.ps1 by exploring the content of $gist created earlier. This function also takes advantage the variables set earlier. The [Github Rest API documetation][github-doc] states that we must use the _patch_ method to update a gist and the full URL of it, while also using authentication. The part the documentation leaves out is a PowerShell example of using the API.

This function currently throws all the results to the screen. I might change that later on, possibly only showing a concise success/failure message. There are plenty of other ways you might format the __Invoke-RestMethod__ call as well, such as predefining the content type and header as variables, converting $body to json earlier, or even using the PowerShell [Splatting][pssplatting] method for the whole thing.

## Creating a Github Token for Gist

Creating your Github token is pretty straight forward using [their documentation][github-token]. To write to a gist, the only permission needed on the token is 'gist' and nothing else.

I've set my token as a permanent environment variable so that I don't have to store it in my profile. You can create a new environment variable with a point-and-click method by right clicking _Start -> System -> Advanced System Settings -> Environment Variables -> New.. (user variable)_ but that's not very PowerShell, is it?

[This Microsoft document][msdoc-env] explains some of how to set a permanent environment variable using the .Net method available to PowerShell.

What we need is the SetEnvironemntVariable method to create our new GithubTokenGIST variable (you could call it anything you want). I've named mine so that I might have other tokens for other purposes in the future and keep them in order.

```powershell
[Environment]::SetEnvironmentVariable("GithubTokenGIST", "1098a98jjhYourToken", 'User')
```

This will create the new variable available only to your user profile, and be aware that it _is_ clear text in your registry. Note that you may need to start a new session to load the new environment variable.

Thanks for reading!

-Kevin

[fancy-prompt]:  {% post_url 2020-05-09-Getting-Started-With-a-Fancy-PowerShell-Prompt %}
[stevelee-gist]: https://gist.github.com/SteveL-MSFT/a208d2bd924691bae7ec7904cab0bd8e
[github-doc]:    https://docs.github.com/en/free-pro-team@latest/rest/reference/gists#update-a-gist
[pssplatting]:   https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-7.1
[github-token]:  https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token#creating-a-token
[msdoc-env]:     https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables?view=powershell-7.1
