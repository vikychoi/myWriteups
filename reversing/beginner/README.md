**Beginner**

Dust off the cobwebs, let's reverse!

**Description**

This is a simple crackme that utilizes intel intrinsic instruction to hide the flag.

This writeup serves as a document on how I solved this challenge mainly using ltrace.

**Solution**

Using IDA, the binary takes in user input, `v5`, and performs shuffling, addition, xor, eventually resulting `s2`. The value `s2` is compared with v5. If `v5` and `s2` are equal, then it results in the SUCCESS message

![img](https://media.discordapp.net/attachments/743500937926017084/746593064960786452/unknown.png)



In essence, this is what it does.

![img](https://media.discordapp.net/attachments/743500937926017084/746592120109924422/unknown.png)



Sadly I am inexperienced with ANGR and Z3, so instead I have to rely on ltrace to solve it.



Based on some documentation of _mm_shuffle_epi8 _

https://software.intel.com/sites/landingpage/IntrinsicsGuide/#text=_mm_shuffle_epi8&expand=5153

I noticed each byte of the input is mapped to another index of the input string for the final check.

This implies if we know the correct value of a byte of the input, we could know the correct byte of another byte. Hence the EXPECTED_PREFIX constant can be used to slowly breaking the flag one by one.



Summarizing what I know about the flag:

1. length = 16
2. starts with "CTF{", ends with "}"



Here is how I do it with ltrace. 

Feed the binary with these two input:

`CTF{aaaaaaaaaa}`
`CTF{bbbbbbbbbb}`

Here is the result

![image-20200829145615910](C:\Users\vikyc\Documents\myWriteups\googlectf2020\reversing\beginner\image-20200829145615910.png)

The input is compared with:

```Cx\273z\203Z\034Df0`@,S=```

```Cy\244z\202[\033Df0aA#P<```

Comparing the two string, some of the characters are the same, `Df0` at index 9

We now know the the 3 byte of the flag starting index 9.

**Repeating the above procedure:**

Feed

`CTF{aaaDf0aaaa}` | `CTF{bbbDf0bbbb}`

Get

```CxF{\203ZMDf0`M,S=```|```CyF{\202[MDf0aM#P<```



Feed

`CTF{aaMDf0aMaa}`|`CTF{bbMDf0bMbb}`

Get

```CTF{\2036MDf0`M,S=```|```CTF{\2026MDf0aM#P<```



Feed

`CTF{a6MDf0aMaa}`|`CTF{b6MDf0bMbb}`

Get

``CTF{n1MDf0`M,S=``|`CTF{n1MDf0aM#P<`



Feed

`CTF{n1MDf0aMaa}`|`CTF{n1MDf0bMbb}`

Get

`CTF{S1MDf0]M,S=`|`CTF{S1MDf0]M#P<`



Feed

`CTF{S1MDf0aMaa}`|`CTF{S1MDf0bMbb}`

Get

`CTF{S1MDf0rM,S=`|`CTF{S1MDf0rM#P<`

...

Get flag:

`CTF{S1MDf0rM3!}`

