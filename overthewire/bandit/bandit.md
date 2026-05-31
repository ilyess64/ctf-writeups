# OverTheWire Bandit CTF Writeups
=======
                # OVERTHEWIRE BANDIT CTF WRITEUPS
>>>>>>> e252ef6 (add overthewire bandit writeups for levels 9-14)

[ MISSION LOG: LEVEL 0 ]
----------------------------------------------------------------------
* Objective     : Log into the game server and find the first password.
* Concept       : Remote Logging & Basic Navigation
* Strategy      : Logged in using the default username and password on port 
                  2220, looked at the files in the folder, and opened the 
                  readme file.

> COMMANDS:
  ssh bandit0@bandit.labs.overthewire.org -p 2220
  ls
  cat readme


[ MISSION LOG: LEVEL 1 ]
----------------------------------------------------------------------
* Objective     : Read the password from a file named "-" (a single dash).
* Concept       : Input Streams vs. File Paths
* Strategy      : Typing a plain dash makes Linux think it is waiting for 
                  keyboard input. Forced the terminal to see it as a normal 
                  file by adding the dot and slash path in front of it.

> COMMANDS:
  cat ./-


[ MISSION LOG: LEVEL 2 ]
----------------------------------------------------------------------
* Objective     : Open a file that has spaces in its name.
* Concept       : Line Spaces & Text Quoting
* Strategy      : The terminal treats spaces as breaks between different 
                  instructions. Solved this by putting the whole name inside 
                  double quotes so Linux read it as one single filename.

> COMMANDS:
  cat "./spaces in this filename"


[ MISSION LOG: LEVEL 3 ]
----------------------------------------------------------------------
* Objective     : Find and open a hidden file inside a folder.
* Concept       : Hidden Files & Detailed Listing
* Strategy      : Files starting with a dot are hidden by default. Switched 
                  into the folder and used the "-la" switches to force the 
                  terminal to show absolutely everything.

> COMMANDS:
  cd inhere
  ls -la
  cat ./.hidden


[ MISSION LOG: LEVEL 4 ]
----------------------------------------------------------------------
* Objective     : Find the only readable text file inside a group of binaries.
* Concept       : Wildcard Matching & Content Viewing
* Strategy      : Used the star symbol (*) to open all the "file0" files at the 
                  same time, then scanned the text on screen to find the one 
                  password that made sense.

> COMMANDS:
  cd inhere
  cat ./file0*


[ MISSION LOG: LEVEL 5 ]
----------------------------------------------------------------------
* Objective     : Find a 1033-byte, un-executable text file deep in nested folders.
* Concept       : File Searching & Size Filtering
* Strategy      : Used the find tool with exact settings: looking for regular 
                  files (-type f), matching the exact size (-size 1033c), making 
                  sure I can read it (-readable), and skipping programs (! -executable).

> COMMANDS:
  cd inhere/
  find . -type f -size 1033c ! -executable -readable
  cat ./maybehere07/.file2


[ MISSION LOG: LEVEL 6 ]
----------------------------------------------------------------------
* Objective     : Search the whole system for a 33-byte file owned by user bandit7 and group bandit6.
* Concept       : System Searching & Error Hiding
* Strategy      : Started a search from the very top folder (/). Added the owner 
                  and size rules, then sent all the annoying "Permission denied" 
                  errors into a black hole to keep the screen clean.

> COMMANDS:
  find / -type f -size 33c -group bandit6 -user bandit7 2>/dev/null


[ MISSION LOG: LEVEL 7 ]
----------------------------------------------------------------------
* Objective     : Extract the password line sitting right next to a specific word.
* Concept       : Text Searching & Keyword Matching
* Strategy      : Used the grep tool to scan through the massive data file and 
                  instantly pull out only the line that contained the word "millionth".

> COMMANDS:
  grep "millionth" data.txt


[ MISSION LOG: LEVEL 8 ]
----------------------------------------------------------------------
* Objective     : Find the only single line of text that does not repeat.
* Concept       : Data Sorting & Duplicate Counting
* Strategy      : The unique checking tool only counts identical lines if they are 
                  right next to each other. Sorted the file first, then passed it 
                  to the uniq tool to find the line with a count of 1.

> COMMANDS:
  sort data.txt | uniq -c

[ MISSION LOG: LEVEL 9 ]
----------------------------------------------------------------------
* Objective     : Find the password inside a file filled with data garbage.
* Concept       : Binary Text Extraction
* Strategy      : The file was mostly unreadable machine code, but it had a 
                  few human-readable lines preceded by equal signs. Used the 
                  strings tool to filter out the noise and grab the password.

> COMMANDS:
  strings data.txt | grep "=="


[ MISSION LOG: LEVEL 10 ]
----------------------------------------------------------------------
* Objective     : Decode a file containing a Base64 encoded password.
* Concept       : Data Encoding Formats
* Strategy      : Recognized the data format as Base64 text. Used the built-in 
                  decode flag (-d) with the base64 command to instantly 
                  translate it back into plain text.

> COMMANDS:
  base64 -d data.txt


[ MISSION LOG: LEVEL 11 ]
----------------------------------------------------------------------
* Objective     : Decrypt a password file where all letters are shifted.
* Concept       : Substitution Ciphers (ROT13)
* Strategy      : The text was scrambled using a classic ROT13 cipher (where 
                  each letter is moved 13 spots forward). Used an online 
                  cipher decoder site to quickly reverse the shift.

> COMMANDS:
  cat data.txt (then pasted into online ROT13 decoder)


[ MISSION LOG: LEVEL 12 ]
----------------------------------------------------------------------
* Objective     : Recover a text file buried inside multiple layers of compression.
* Concept       : File Signatures & Nested Decompression
* Strategy      : Moved to a temporary folder in /tmp to have write access. Reversed 
                  the hex dump text back into a binary file using xxd. Then, used 
                  the file tool repeatedly to discover each layer's type, renaming 
                  and unpacking them step-by-step (Gzip -> Bzip2 -> Gzip -> Tar 
                  -> Tar -> Bzip2 -> Tar -> Gzip) until reaching plaintext ASCII.

> COMMANDS:
  mkdir /tmp/something && cp data.txt /tmp/something && cd /tmp/something
  xxd -r data.txt > name
  file name
  mv name name.gz && gzip -d name.gz
  file name (bzip2) -> bunzip2 name
  (Repeated file, mv, and extraction tools: gzip -d, bunzip2, tar -xf)
  cat name


[ MISSION LOG: LEVEL 13 ]
----------------------------------------------------------------------
* Objective     : Log into the next level using a private SSH security key.
* Concept       : Private Key Authentication & Server Blocks
* Strategy      : The game server explicitly blocks internal connections to 
                  localhost to save resources. Copied the text inside 
                  sshkey.private, exited back to the local WSL2 terminal, 
                  and saved it as bandit14.key. Fixed the open file permissions 
                  using chmod 600, and logged directly into level 14 from the 
                  local machine.

> COMMANDS:
  cat sshkey.private (copy text)
  exit
  nano bandit14.key (paste text)
  chmod 600 bandit14.key
  ssh -i bandit14.key bandit14@bandit.labs.overthewire.org -p 2220


[ MISSION LOG: LEVEL 14 ]
----------------------------------------------------------------------
* Objective     : Submit the current level password to a specific port to get 
                  the next flag.
* Concept       : Network Ports & Raw Data Transfer
* Strategy      : Printed the retrieved password file for Level 14 and used 
                  the netcat (nc) tool to send it directly to port 30000 on the 
                  local network loopback interface.

> COMMANDS:
  cat /etc/bandit_pass/bandit14 | nc localhost 30000
