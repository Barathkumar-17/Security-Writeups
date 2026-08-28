For level 5 had to use find command 
1033 bytes file , is not executeable
1. find . -size "1033c"
    // . is for all directories under this 
    //1033c is file size , c is for telling it is a byte size 
2. find ! -executeable
    // returns paths of file which is not executeable
3. find -type f
    //returns only files not links or directories 
    // l returns links 
    // d returns directories
