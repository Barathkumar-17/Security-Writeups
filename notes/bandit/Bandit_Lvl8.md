This level requires two new commands sort , unique ,or unique , count , grep.

We need to find the sequence which only occurs once in a file .

So for this we can use unique command

uniq -c : add thier count next to them before the sequence

uniq -u : ouputs seqs which only occurs once

uniq -d : output only duplicates

"issue is uniq checks sequence with adjacent seqs only"

we need to sort the file before uniq always so that uniq works as intended otherwise it will fail to work

command : 

sort data.txt| uniq -u

