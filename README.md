# Autopocket

## Introduction

Auto-pocket is a tool designed to automate the addition and removal of articles into your personal pocket account. It was designed primarily for use with Kobo ereaders to provide a daily selection of news articles to your reader for a newspaper like experience. 

Using the config file, you can setup a series of RSS news feeds that shall be added to your pocket account and automatically removed after a configurable amount of time to avoid your account getting out of hand. 

All articles uploaded are tagged with an auto-pocket tag and a further tag of your choice to provide a level of seperation. 

Lastly the program accepts input of a URL from the command line. This can be useful for integration with other applications such as newsbeuter.


## Installation

requests and feedparser packages are required: I have tested this against requests (2.10.0) and feedparser (5.1.3)

```
sudo pip3 install requests feedparser


clone the repository

git clone https://github.com/Dave-Vallance/Auto-Pocket.git


python3 Auto-Pocket

or 

chmod +x Auto-Pocket

./Auto-Pocket

On first time installation you will be requested to sign in and authorize the application. Once authorized, the 


```

## Configuration file

A basic configuration is provided. Articles will be be automatically deleted after the amount of days sepecified for _delete_after_ tag. Note that articles marked as favorite or archived will not be deleted from pocket ever. Even ones added by autopocket. This is by design.

```
[General]
#General Configuration
#The main tag used to identify Auto-Pocket content
Auto-Pocket-Tag = Auto-Pocket
#Tag to be given to articles uploaded via command line
cli_tag = CLI-Upload
#Delete cli uploaded articles older than 1 day
cli_delete_after = 1


[Engadget]
url = http://www.engadget.com/rss.xml
#Delete articles older than 1 day
delete_after = 1 
# Add all articles from the past 1 day
days_to_add = 1
# RSS feed special tag. 
tag = engadget 
```

## Usage

```
usage: Auto-Pocket [-h] [-u URL]

Auto Pocket, automatically adds articles to pocket from given feeds in the
config file

optional arguments:
  -h, --help         show this help message and exit
  -u URL, --url URL  Allows uploading a single article from the command line.
                     Usefull for integration with other apps e.g newsbeuter

```


## Integration with Newsbeauter

To integrate with newsbeauter you must first create a macro in the ~/.newsbeuter/config directory

```
macro p set browser "/[Path-to-Autopocket]/Auto-Pocket/Auto-Pocket --url %u"; open-in-browser ; set browser "firefox %u
```

**Note:** Replace firefox with your web browser of choice. 
