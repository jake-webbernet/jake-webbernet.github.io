Today I got my raspberry pi hooked up in my office. I wanted a way to be able to play some music at 9:15am each day to remind everyone about our stand up meetings.

### OS
My first job was to install a fresh operating system onto the SD card for the Pi. I initially downloaded `Ubuntu mate` but it turned out it wasn't compatible with my Pi. It wouldn't boot pass the bios.

Instead, I downloaded NOOBs, which is pretty much the officially supported OS for the Pi.  I used the `SD Formatter` program from the 'SD association' to format the SD first and then just simply copied the files (no irregular flashing required)

### Booted

After booting the first thing I wanted to do was get SSH going as I'm not really interested in using the Gui. To get this going I simply opened the terminal and used `raspi-config`. This opened a menu where I could enable SSH, really easy.

Next I needed to get it on my network. I have a Pi 2 which means it doesn't have any onboard wireless capabilites, only ethernet. I've always found this quite limiting and it's one of the reasons I think I really didn't get good use out of the product before.

My solution was buying a tiny USB Wifi Dongle, actually designed for the Pi from ebay for $9 delivered. I plugged this in, and my Pi automatically restarted (weird) and then the wifi was available. I was suprised there was no other setup required.

By the end of the stage I had a network connection and SSH setup. I can now unplug the screen as I can interface from my main laptop.

### SSH

I was wondering about what IP I would use to actually access the Pi. It turns out I could just access it via `raspberrypi.local` which is really cool

```
ssh raspberrypi.local -l pi
Default username is `pi`
Default password is `raspberry`
```

## Playing Music

Some quick googling suggested I needed `mpg123` to play music.

`sudo apt-get install mpg123`

I installed and downloaded a random mp3 from the net 

`wget link-to-file.mp3 -O file.mp3`

And I can play it with

`mpg123 file.mp3`

## Setting up Audio

I needed to configure audio to come out of the 3.5mm jack on the Pi. I read somewhere that HDMI was acutally the default audio output.

I could change this by again running `raspi-config` and going to advanced and audio settings.

Lastly, i used `$ alsamixer` to adjust the volume on the Pi. 

## Shutdown

`sudo shutdown -h now` 

And just wait for the `act` to stop blinking on the board before unplugging

## Running Scripts on a timer

So I needed to run the `mpg123` command at a specified time each morning. I decided to use cron for it.

`crontab -e` Opens Vim
 
and added this to the file.
 
`15 23 * * * mpg123 music.mp3`

This means at 23:15 UTC each day the command will run

Easy as...Pi


