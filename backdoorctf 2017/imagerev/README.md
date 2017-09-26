**CTF: backdoorctf 2017**
--
**Challenge:** `imagerev`
**Tags:**  `crypto` 
**Given files:** [encrypt.py][encrypt], [encrypted.txt][encrypted]

By reading [encrypt.py][encrypt] we can identify what input becomes an output.

Actually [encrypt.py][encrypt] takes through Image.getdata() a linear list of every pixel on image. 

**Ex:** `[(97,98,99),(254,52,63),(562,85,45),......]`

In `data_encrypted()` each pixel is transformed in a 3-ascii letter word.

**Ex:** `(97,98,99) => 'abc'`

Then in `encryption()` an input of 'abc' will give up an 8-tuple of 8-char hex number.
According to [encrypted.txt][encrypted] each "8-tuple of 8-char hex number" are repeting itself which that means it's probably the same pixel each time.

**Ex:** `['709e80c8','8487a241','1e1ee4df','b9f22a86','1492d20c','4765150c','0c794abd','0f81477c']`

Exploit should be to use this regularity to determine the more current pixel as the background of our solution and others pixels as the flag text.

Using python we can count how many "8-tuple of 8-char hex number" (pixels) there is and try each format (height, widht) with height between 1 to 100 (because of letter size).

(Note that if you try to encrypt images you can see that encryption is creating a huge amount of data which means flag must be a very tiny image.) 

Here is the python [exploit.py][exploit] for more indications.

```python
from PIL import Image

def get_hex_list(filename):
	f = open(filename, 'r')
	crypted_data = f.read()
	word_list = list()
	data_list = list()
	count = 0
	while count < len(crypted_data):
		word_list.append(crypted_data[count:count+8])
		if len(word_list) == 8:
			data_list.append(word_list)
			word_list = list();
		count += 8
	return data_list

def resolve(input_list):
	formated_list = list()
	for item in input_list:
		if item == input_list[0]:
			formated_list.append((0,0,0))
		else:
			formated_list.append((255,255,255))
	for i in range(1,100):
		if len(formated_list) % i == 0:
			size = (len(formated_list) / i, i)
			im = Image.new("RGB",size,"white")
			im.putdata(formated_list)
			title = './flag' + str(i) + '.png'
			im.save(title)

hex_data_list = get_hex_list('./encrypted.txt')
resolve(hex_data_list)
```

Finally we get images like those (you just have to find the right one with the good ratio [w:h]) :
![9 pixel height flag generated](/backdoorctf%202017/imagerev/flag9.png)
![81 pixel height flag generated](/backdoorctf%202017/imagerev/flag81.png)
![91 pixel height flag generated](/backdoorctf%202017/imagerev/flag91.png)

As [backdoor][backdoor_site] asked there is not the right flag here. Try to find it by yourself.

[encrypt]:/backdoorctf%202017/imagerev/encrypt.py
[encrypted]:/backdoorctf%202017/imagerev/encrypted.txt
[exploit]:/backdoorctf%202017/imagerev/exploit.py
[backdoor_site]:https://backdoor.sdslabs.co/
