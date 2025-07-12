why was character set needed in the first place?
- earlier, the only characters that mattered were unaccented english letters
- ascii was able to represent every character using a number between 32 and 127
	- this could be conveniently in 7 bits
	- most computers in those days were using 8-bit bytes, so not only could we store every possible ASCII char, but we had a whole bit to spare
	- in some word processors, the high bit was turned on the high bit to indicate the last letter in a word
	- codes below 32 were called unprintable as they were used for control characters
		- 7 made computer beep
- the extra one high bit allowed many to come up with their own use for the possible 128 characters that it provided
	- the IBM PC had something that came to be known as the OEM char set
		- it provided some accented characters for european languages and a bunch of line drawings
	- outside USA, all kinds of OEM character sets were dreamed up, which all used the top 128 chars for their own purpose
		- For example on some PCs the character code 130 would display as é, but on computers sold in Israel it was the Hebrew letter Gimel (![ג](https://i0.wp.com/www.joelonsoftware.com/wp-content/uploads/2003/10/gimel.png?resize=5%2C9&ssl=1)), so when Americans would send their résumés to Israel they would arrive as r![ג](https://i0.wp.com/www.joelonsoftware.com/wp-content/uploads/2003/10/gimel.png?resize=5%2C9&ssl=1)sum![ג](https://i0.wp.com/www.joelonsoftware.com/wp-content/uploads/2003/10/gimel.png?resize=5%2C9&ssl=1)s
- eventually, this OEM free for all got codified in the ANSI std
	- everyone agreed on what to do below 128, which was pretty much the same as ASCII
	- but there were lot of different ways to handle the chars from 128 and on up depending upon where one lived
	- these different systems were called code pages
		- israel DOS used a code page called 862, while greek users used 737
	- in asian languages, which had more than 1000 letters - 8 bits was not enough
		- the solution was DBCS: double byte character set
			- each character is either 1 or 2 bytes
			- the first of a two byte character comes from a specific range called a lead byte range
			- the second by has its own range 
		- some letters were stored in one byte and others took two
		- it was easy to move forward in a string, impossible backwards
		- programmers were encouraged not to use s++, s-- 
		- alternate way to navigate through a string was: AnsiNext and AnsiPrev
- 
- 
f

1031

1047