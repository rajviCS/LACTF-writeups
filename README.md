# LACTF-writeups
lactf writeups - 2025

**web/lucky-flag**

The challenge hints: Just click the flag :)

I navigated to the website challenge - https://lucky-flag.chall.lac.tf/ and it appears that the site is filled with boxes - and all of them have the word "flag" written on them! 
I started by clicking on the boxes at the start. A pop up appears, stating that a flag cannot be found. 
This leads me to the conclusion that the flag can be obtained when one out of the many boxes, which when clicked yields the flag, is clicked.
Since there are too many boxes, automation through code needs to be used so that I have a way where all the boxes are clicked at once, rather than a human having to do it manually. So doing inspect element --> console --> and then using a javascript code yields a flag. 

Code used: 
```
document.querySelectorAll('.box').forEach((button, index) => {
    const originalAlert = window.alert;
    window.alert = function(message) {
        console.log(Button ${index + 1} output: ${message});
    };
    button.click();
    window.alert = originalAlert;
});
console.log("All buttons clicked.");
```

Going through the log outputs, the flag can be found - lactf{w4s_i7_luck_0r_ski11}

**web/I spy…**

In this challenge, I see a text box telling us to enter a token. 

1. The very first token is visible underneath the textbox: B218B51749AB9E4C669E4B33122C8AE3. 
After entering it, it says: A token in the HTML source code…

2. After a right click, click on view source. I spotted another token in the page source - <!-- Token: 66E7AEBA46293C88D484CDAB0E479268 → After entering the token, it now says: A token in the JavaScript console...

So this appears to be a sort of scavenger hunt! 

3. Right click, Inspect, console → 5D1F98BCEE51588F6A7500C4DAEF8AD6. Message says: A token in the stylesheet... Stylesheet likely refers to the css file, which will be in our sources. 

4. Right click, Inspect, Sources. Navigate to styles.css, and the /* Token: 29D3065EFED4A6F82F2116DA1784C265 */ appears at the top. Message says: A token in javascript code... Again, this javascript is probably a .js file in our sources. 

5. Navigate to thingy.js from the sources. // Token: 9D34859CA6FC9BB8A57DB4F444CDAE83. Message says: A token in a header...

6. Right click → Network → start recording the network log by reloading the website. Then, clicking on the website, I notice a “Headers” tab which seems promising. Scrolling a bit down, I notice it gives us an X-Token underneath the Response Headers: BF1E1EAA5C8FDA6D9D0395B6EA075309. Trying this out as the token, I get the next message: A token in a cookie... 
 
7. In the network tab, there is the “cookies” header where I notice two request cookies. The first is a-token and the second is stage_token. The stage token appears rather long, so the a-token is likely what the challenge requires. Upon entering it, we get the message: A token where the robots are forbidden from visiting... 

8. The robots.txt file is a text file which tells web crawlers (which are responsible for indexing websites) what sites they are and aren’t allowed to visit. In CTFs, what is of most interest to us is what robots are supposedly not supposed to visit - as that is where the interesting things exist :) Appending /robots.txt to the website url (so the url looks like this: https://i-spy.chall.lac.tf/robots.txt

9. The robots.txt file states: 
User-agent: *
Disallow: /a-magical-token.txt

10. And so of course, our next task is to navigate to what is disallowed. Upon modifying the end of the url of the site to now be https://i-spy.chall.lac.tf/a-magical-token.txt I notice the following text: Token: 3FB4C9545A6189DE5DE446D60F82B3AF

11. Entering the token above, the message: A token where Google is told what pages to visit and index… After some google search, I discover that what the hint likely refers to is an XML sitemap, which is a file listing the important pages on a website, and is used in making sure Google crawls them. Let’s navigate to this using: https://i-spy.chall.lac.tf/sitemap.xml. Aha! Token: F1C20B637F1B78A1858A3E62B66C3799. The message is: A token received when making a DELETE request to this page… 

12. A DELETE request is a kind of HTML request. This reminded me of a good tool to use to perform HTML requests, which is cURL. cURL is a command-line tool. Trying to do the cURL operation, I kept getting a cURL connection time out error (perhaps something to do with being on school wifi?) Shoutout to my teammate Komet for performing the cURL operation on this one: 
$ curl -X DELETE https://i-spy.chall.lac.tf/

The output obtained: 
You DELETED MY WEBSITE!!!!! HOW DARE YOU????? 32BFBAEB91EFF980842D9FA19477A42E
where 32BFBAEB91EFF980842D9FA19477A42E must be the token. 

13. The message now reads: A token in a TXT record at i-spy.chall.lac.tf...  

14. At this one, I was going around in a few circles to figure out what this could mean and how to find the token. Shoutout to my teammate Komet for sharing that this probably had to do with DNS and suggesting that this site could help with it: https://dnschecker.org/all-dns-records-of-domain.php 

15. After entering in the website the url i-spy.chall.lac.tf/ and after navigating to the “TXT” record type we get the final token- Token: 7227E8A26FC305B891065FE0A1D4B7D4. 

16. Finally, the flag is displayed instead of a new message: A Flag! Lactf{1_sp0773d_z_t0k3ns_4v3rywh3r3}. A pretty apt flag, I guess one might say :) 


**rev/javascryption**

This challenge has a website mentioned in the description, so let’s navigate to it. There is a textbox with a space to enter the flag and check if it is correct. My first hunch is to just right click and click on “view page source.” Upon viewing the page source, I noticed a script, <script src="cabin.js"></script>. I right clicked on cabin.js and clicked to open it in a new tab. This yields more code, and then I noticed something interesting:

```
function checkFlag(flag) {
    const step1 = btoa(flag);
    const step2 = step1.split("").reverse().join("");
    const step3 = step2.replaceAll("Z", "[OLD_DATA]");
    const step4 = encodeURIComponent(step3);
    const step5 = btoa(step4);
    return step5 === "JTNEJTNEUWZsSlglNUJPTERfREFUQSU1RG85MWNzeFdZMzlWZXNwbmVwSjMlNUJPTERfREFUQSU1RGY5bWI3JTVCT0xEX0RBVEElNURHZGpGR2I=";
}
```

This suggests that several operations were done on the flag, and then an output of "JTNEJTNEUWZsSlglNUJPTERfREFUQSU1RG85MWNzeFdZMzlWZXNwbmVwSjMlNUJPTERfREFUQSU1RGY5bWI3JTVCT0xEX0RBVEElNURHZGpGR2I=" was obtained. The flag can therefore be reverse engineering through doing the opposite of these steps (i.e., decoding instead of encoding) in the reverse order. 

1. Upon a quick google search, it is apparent that the btoa() method encodes the input in base-64. Therefore, using an online base64 calculator tool, I can decode an input of  "JTNEJTNEUWZsSlglNUJPTERfREFUQSU1RG85MWNzeFdZMzlWZXNwbmVwSjMlNUJPTERfREFUQSU1RGY5bWI3JTVCT0xEX0RBVEElNURHZGpGR2I=" 

The base64 calculator I used was this one, but any should work: https://emn178.github.io/online-tools/base64_decode.html 

2. The result obtained from doing base64 decoding: %3D%3DQflJX%5BOLD_DATA%5Do91csxWY39VespnepJ3%5BOLD_DATA%5Df9mb7%5BOLD_DATA%5DGdjFGb 

3. Next, I see that the step was to encodeURIComponent(step3) so I need an online URL decoder. The one I used was this, but any can work: https://www.urldecoder.org/ 
As input, I put %3D%3DQflJX%5BOLD_DATA%5Do91csxWY39VespnepJ3%5BOLD_DATA%5Df9mb7%5BOLD_DATA%5DGdjFGb and get the output of ==QflJX[OLD_DATA]o91csxWY39VespnepJ3[OLD_DATA]f9mb7[OLD_DATA]GdjFGb 

4. Seeing that the step was for step2.replaceAll("Z", "[OLD_DATA]"); I know that I should now replace all instances of [OLD_DATA] in the string with the letter Z. The input of ==QflJX[OLD_DATA]o91csxWY39VespnepJ3[OLD_DATA]f9mb7[OLD_DATA]GdjFGb yields output: ==QflJXZo91csxWY39VespnepJ3Zf9mb7ZGdjFGb 

5. Next, step1.split("").reverse().join("") suggests to reverse the string. The tool used: https://onlinestringtools.com/reverse-string with an input of ==QflJXZo91csxWY39VespnepJ3Zf9mb7ZGdjFGb yields an output of bGFjdGZ7bm9fZ3JpenpseV93YWxsc19oZXJlfQ==

6. Finally, btoa(flag) means that I should just do base64 decoding, input of bGFjdGZ7bm9fZ3JpenpseV93YWxsc19oZXJlfQ== gives output of the flag: 

lactf{no_grizzly_walls_here}

**crypto/Extremely Convenient (with team)**

In this challenge, I worked with my teammates. Upon connecting to the challenge as well as inspecting the source code file provided in the challenge, we realized that the code encrypts the flag from a file using AES in ECB mode and displays this encrypted flag in hex. Then, the user has an option to attempt to decrypt a provided ciphertext but there’s a catch - only if the provided ciphertext is not the encrypted flag does it decrypt it and prints the result. If the provided ciphertext is the flag, it refuses to decrypt the flag. We tried several options, such as starting from the basic idea of doing buffer overflow and seeing if it was susceptible to this. It didn’t appear to work. Then, shoutout to my teammate Cookie, who thought of changing only the last letter in the encrypted flag in hex, to check what the output gives us. It turned out to be a partial flag: 
b"b'lactf{seems_it_was_extremely_convenient_to_get_t\\xf4\\xbd\\x17\\x97#\\x8cW\\xfei\\xbf\\xe17/iB.'\n"
After tinkering with some more letters: 
b"b'`b\\x0e2\\x1d\\xdf\\x813\\xe1\\xaf\\xce\\x92%\\x0ft$as_extremely_convenient_to_get_the_flag_too_heh}'\n"

Then we get: 
Lactf{seems_it_was_extremely_convenient_to_get_the_flag_too_heh}
Finally get the flag! 
 

**web/chessbased (with team)**

In this challenge, I worked with my teammates. One of my suggestions to the team was we could try seeing if it was an XSS challenge. I had watched a youtube video of the walkthrough on some of the web challenges from LA CTF 2024, and the formatting of the site for this challenge appeared to be similar to an XSS challenge from last year - especially the admin bot made it seem that way to me. I shared the video with my team mates, and after trying to do the same technique done in the video in the group, we found no luck. 

Then, we looked through the code: 

```
app.get('/render', (req, res) => {
  const id = req.query.id;
  const op = lookup.get(id);
  res.send(`
    <p>${op?.name}</p>
    <p>${op?.moves}</p>
  `);
});

app.post('/search', (req, res) => {
  if (req.headers.referer !== challdomain) {
    res.send('only challenge is allowed to make search requests');
    return;
  }
  const q = req.body.q ?? 'n/a';
  const hasPremium = req.cookies.adminpw === adminpw;
  for (const op of openings) {
    if (op.premium && !hasPremium) continue;
    if (op.moves.includes(q) || op.name.includes(q)) {
      return res.redirect(`/render?id=${encodeURIComponent(op.name)}`);
    }
  }
  return res.send('lmao nothing');
});

```

Looking at this line -  const hasPremium = req.cookies.adminpw === adminpw - made me wonder if perhaps it’s a cookie tampering challenge? I suggested this to my teammates too. They had not done a cookie tampering challenge yet, so I shared how there are browser extensions where it’s possible to play around with the cookie values. But after a half hour, we still had no luck. 

Then, shoutout to my team mate Cookie, who was thinking of trying to navigate through the website and thought of just visiting /render?id=admin to see if anything would pop up. It just displayed: 
undefined
undefined 

But it didn’t lead to any errors. Next, Cookie tried to visit https://chessbased.chall.lac.tf/render?id=flag just to see what kind of output would show up. Imagine the surprise when that yielded the flag! 

flag

lactf{t00_b4s3d_4t_ch3ss_f3_kf2}

At the time, we wondered if this was a fluke, but the ctf platform accepted our flag. We wondered if this is how the other teams had solved it too, and decided we would definitely wait for the writeups to see if this was an unintended vulnerability or the intended solution. But either way, we weren’t complaining! :) 

**Thanks for reading!**
