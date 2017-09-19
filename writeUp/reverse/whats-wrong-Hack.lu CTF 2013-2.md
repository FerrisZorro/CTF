 

# Hack.lu 2013 CTF Write-Up: What’s wrong with this?

> We managed to get this package of the robots servers. We managed to determine that it is some kind of compiled bytecode. But something is wrong with it. Our usual analysis failed – so we have to hand this over to you pros. We only know this: The program takes one parameter and it responds with “Yup” if you have found the secret code, with “Nope” else. We expect it should be obvious how to execute it.

The challenge provides a compressed archive _hello.tar.gz_ which contains a bunch of shared objects, an archive called _library.zip_ (actually an ELF executable with an embedded ZIP) and a Python interpreter _py_. The executable to be cracked is _hello_, which will reply “Yup” if the correct flag is provided, “Nope” otherwise.

When we extract the ZIP archive we find compiled bytecode files with the _.pyc_ extension, including ___main__hello__.pyc_, which is the main file loaded by _hello_. The Python interpreter from which these files were built has been modified, similarly to [what is done in the Dropbox client](https://www.usenix.org/sites/default/files/conference/protected-files/kholia_woot13_slides.pdf). For this reason, it is not possible to simply demarshal it in order to get insights on how it works. However, after some unsuccesful attempts with Uncompyle2, we were able to look at the bytecode of the file using [PyChrisantemum](https://code.google.com/p/pychrysanthemum/).

Two functions are defined: `rot_chr` and `encrypt_string`. The input string is encrypted and then compared with `w*0;CNU[\gwPWk}3:PWk"#&:ABu/:Hi,M`. The following picture shows the bytecode for the encryption function:

[![di-HIVD](http://secgroup.dais.unive.it/wp-content/uploads/2013/10/di-HIVD.png)](http://secgroup.dais.unive.it/wp-content/uploads/2013/10/di-HIVD.png)

By looking at the bytecode, it is possible to understand and recreate the code for the function:

```python
 def encrypt_string(s):

 new_str = []

 for (index, c) in enumerate(s):

 if index==0:

 new_str.append(rot_chr(c, 10))

 else:

 new_str.append(rot_chr(c, ord(new_str[index-1])))

 return ''.join(new_str)
```


This is easily reversible and the corresponding decryption is:


```python
 def decrypt_string(s):

 new_str = []

 for (index, c) in enumerate(s):

 if index==0:

 new_str.append(rot_chr(c, -10))

 else:

 new_str.append(rot_chr(c, -ord(s[index-1])))

 return ''.join(new_str)
```


We can use the provided modified Python interpreter _py_ to import the compiled file and get the rotation function for free. Importing `__main__hello__` returns an error if `sys.argv` is empty, hence we append a random string to fill the list of command line arguments.


```
 import sys

 sys.argv.append("useless")

 from __main__hello__ import *
```


The final step is to run `decrypt_string` and get the flag. The following is the program we used to retrieve the flag:

```python

 # Extract all files and then run this with the given ./py

 import sys

 sys.argv.append("useless") # Avoid error on import

 from __main__hello__ import *

 # Original function:

 #

 #def encrypt_string(s):

 #   new_str = []

 #   for (index, c) in enumerate(s):

 #       if index==0:

 #           new_str.append(rot_chr(c, 10))

 #       else:

 #           new_str.append(rot_chr(c, ord(new_str[index-1])))

 #   return ''.join(new_str)

 def decrypt_string(s):

 new_str = []

 for (index, c) in enumerate(s):

 if index==0:

 new_str.append(rot_chr(c, -10))

 else:

 new_str.append(rot_chr(c, -ord(s[index-1])))

 return ''.join(new_str)

 tobecracked = 'w*0;CNU[\\gwPWk}3:PWk"#&:ABu/:Hi,M'

 sol = decrypt_string(tobecracked)

 print "solution: " + sol

 print "original: " + tobecracked

 print "test:     " + encrypt_string(sol)
```


Flag: `modified_in7erpreters_are_3vil!!!`
