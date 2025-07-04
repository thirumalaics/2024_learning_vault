[https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/]()

- ASCII was able to represent every character using a number between 32 and 127
	- space was 32, A was 65
	- for the character representation, we only needed 7 bits, leaving 1 bit extra
	- computers back in the the day were using 8-bit bytes
- codes below 32 were called unprintable and were used for control characters
	- 7 made the computer beep
- because bytes have room for up to 8 bits, lots of people though that they can use 128-255 for their own purposes
	- this was the trouble
	- IBM came up with OEM character set which provided some accented chars for european langs
![[Pasted image 20240514093245.png]]


- OEM got codified in the ANSI std
- in the ansi std, everybody agreed on what to do below 128(pretty much same as ASCII)
	- there were lots of different ways to handle the characters from 128 and up, depending on where we lived
	- these different systems were called code pages
	- for israel it was code page 862, greek 737
	- some PCs had multilingual code pages that could do diff langs on the same computer
	- in asia, since alphabets had thousands of letters, which could not be fit in to 8 bits, DBCS(double byte character set) was introduced
		- messy
		- in which some letters were stored in one byte, but others took two
		- as long as the string was not moved from one computer to another
		- as soon as the internet happened, all of this fell apart
		- Unicode was invented
- Unicode
	- effort to create a single character set that included every reasonable writing system
	- myth: Unicode is 16 bit code where each char takes 16 bits and there are 65536 possible characters
		- this is wrong
	- so far we have assumed that a letter maps to some bits which we can store on disk or in memory:
		- A ->0100 0001
	- in unicode, a letter maps to a theoretical concept called ***code point***
		- how that code point is represented in mem or disk is another story
		- in unicode the letter A is a platonic ideal
		- in some languages, figuring out what a letter is can cause controversy
![[Pasted image 20240514095756.png]]

- string Hello represents five code points in unicode:
- ![[Pasted image 20240514095838.png]]
![[Pasted image 20240514100013.png]]

- big endian above v little endian below
	- big endian - stores the most significant byte of a word at the smallest memory location
	- little endian - stores the most significant byte of the word at the highest memory location