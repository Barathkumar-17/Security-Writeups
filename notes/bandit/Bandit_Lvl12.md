This level requires finding the actual file type and decompressing the file based on it .

Each file's name's extension is just fake of what it really is .

a .gz compressed file can be data.txt .

So first it involved finding what type it was compressed to using the file command

file "filename"

convert file name to match the respective extension type

then decompressing using its respective type

For .gz:

gunzip filename.gz

For .bz2

bunzip2 filename.bz2

For tar , which is like a compressed folder similar to ...

so decompressing it will give u a new file

for tar:

tar -xf filename

do until we get a file with the property ASCII , which means human readable , so we can get the password from it.

