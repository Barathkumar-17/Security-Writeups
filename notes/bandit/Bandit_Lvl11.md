This level Requires ROTR13 shift , ie each character shift by 13 places so a becomes n , both Cases.

For this we use translate command
tr

tr [input] [output] , how input character must become in output .

    input : A-Za-z 
    output: N-ZA-Mn-za-m

A to N becomes N to Z , M to Z becomes A to M

pipe output of file to tr "A-Za-z" "N-ZA-Mn-za-m"

