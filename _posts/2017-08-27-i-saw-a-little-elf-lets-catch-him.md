---
layout: post
title: "I saw a little elf, let's catch him"
author: "jantwisted"
categories: elf, python
tags: [tech, fun]
image: elf-meme.jpg
---


It was a bored weekend, untill Mr. Nobody, (a friend LOL) gave a challenge which he was unable to figure out. It is available in `ringzer0team.com` website, which is a popular website for such challenges. The challenge is to get a secret word out of a huge payload which is encoded into unknown format. The name of the challenge is `I saw a little elf`.

This is going to be a long post, just hold your breath and let me explain how I figured it out. First, I did a google search (I'm smart), and yes, some people have already solved it, however they have less information on what the problem really is. Then I started to understand the problem with my own and here it is. (I have removed 90% of message body to make it small and clear)

* URL : https://ringzer0team.com/challenges/15

```
		You have 3 seconds to get the secret word.
   Send the answer back using https://ringzer0team.com/challenges/15/[your_string]

		----- BEGIN Elf Message -----
   UTJkQ01HRlhOWEJZZDBFeFRHcEpkVTFzT1VSUmEyeE5VakJDUVdOSFZteGlTRTFCV2xkNGFWbFdVbXhpYlRselVUQXhWV050VmpCak1teHVXbGhLWmxSV1VrcFlkMEptV0RCU1QxSldPVVJVVmxKbVdIZENlbHB........etc

		----- End Elf Message -----

		----- BEGIN Checksum -----
		44fde7bfd0a94e2352f4616bd0670ffd
		----- END Checksum -----
```
*  Figure out the encoded method.

The string length was 37164, but when I refresh, it gives another string with a different length. However,  they have some similar attributes, such as the lengths are multiple of 4 and every character is in the set of `[ A-Z, a-z, 0-9, +, / ]`, except the padding at the end. So the conclusion is, the encoded method should be base64. However, it didn't give a useful message, even after the decode process. But why?

*  Why they have given a checksum ?

A checksum is used to verify a file, message or anything related to hashing. This checksum has 32 characters, which is md5. Then, I used md5 to generate a hash for the encoded value, and it gave something different. The expected value would be the same as the checksum. I tried a few times, and suddenly it gave the expected result. I ran it again but no luck this time. So what I was missing ? The iteration. I call the script in a loop until I get what I needed, and it gave the result several times, but not always. Now I have the answer for the 1st step. When the checksum matches, the encoded string gets something meaningful.

* Elf files

Now I have the message, but what is it ? yes, it's an elf file (think about the name), which stands for **Executable and Linking File**. Great, I wrote the string to a file and used `readelf` command to read it (I wrote it as a byte string). It showed the complete details of the file. But I'm yet to figure out the hidden message.

```Shell
readelf -a myelf.elf
```
* Execute Elf file

I renamed my elf file to .out file. Then changed the permissions and executed. Whoa, it printed the answer which I was looking for. :)

```Shell
chmod +x myelf.out && ./myelf.out
```

* The Time

Time is the common enemy for everything. It gives 3 seconds to catch the Elf. When I write an elf file and execute, 90% of the attempts, it failed to accomplish the task within the given time. Instead of executing the file, I opened it and search for the answer (in this case I printed the answer), and it was there, but not that easy to read, because it was in a byte string.

* Vim to rescue

In vim, you can get the details of your current position in the buffer by using **g ctrl+g** in command mode.(other text editors might have similar methods). So, I was able to get the exact location of the hidden word. Now, in the script, without even writing the byte string to a file, I was able to capture the answer by defining a character range. In this method, it has almost 100% accuracy on providing the correct answer on time.

Code junk : https://github.com/jantwisted/wwwcraw-elf

```python
#!/usr/bin/python3

# Author : jantwisted (janith@member.fsf.org)

import requests
import hashlib
import re
import base64
import sys, os


cookie='hccfqq57351dvfv97cc4bdrq25'

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:55.0) Gecko/20100101 Firefox/55.0',
    'Host': 'ringzer0team.com',
    'Referer': 'https://ringzer0team.com/login',
    'Cookie': 'PHPSESSID='+cookie+';'
}

def tryit():
  challenge_url = 'https://ringzer0team.com/challenges/15/'
  challenge = requests.get(challenge_url, headers=headers)
  a = re.search('BEGIN Elf Message -----<br />((.|\n)*?)<br', challenge.text)
  b = re.search('BEGIN Checksum -----<br />((.|\n)*?)<br', challenge.text)
  data = a.group(1).strip()
  checksum = b.group(1).strip()
#  print("length:",len(data))
  data  = base64.b64decode(data)
  data = data[::-1]
  _data = hashlib.md5(data).hexdigest()
  print("Hash data:",_data)
  print("Checksum:",checksum)
  if _data == checksum:
    result = (data[1510:1514]).decode('iso-8859-2')+(data[1518:1520]).decode('iso-8859-2') 
    print("Secret Word:",result)
    result= result.strip('\n')
    new_url = challenge_url + result 
    result = requests.get(new_url, headers=headers)
    print("Challenge flag:",re.findall(r'FLAG-\w*', result.text)[0])
    sys.exit(4)

  sys.exit(5)
 

if __name__ == '__main__':
tryit()
```

Happy Hacking ! :)
