## BCB546X\_Spring2017\_UNIX Assignment##
## Jinyu Wang

----------
 

##Inspecting data  





**1.Using the following code to inspect the size, row number and column number of both of the files**

```
 du -h fang_et_al_genotypes.txt snp_position.txt
```
   
```
wc -l fang_et_al_genotypes.txt snp_position.txt
```

```
awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt snp_position.txt
```

**From the above code:**


- **fang\_et\_al\_genotypes.txt** (size 11M, #rows 2783, #columns 986)
- **snp\_position.txt** (size 84k, #rows 984, #columns 986)
 
**2.Inspect both of the files to check the header and determine based on which column/filed to join those two files**

```
 head fang_et_al_genotypes.txt
```

```
 head snp_position.txt
```

- From the above two line code I could see that those two files containing different number of column headers and the no common column for join.
- Determined that there is need to transpose the data before join the files.

**3.check the number of lines within each group in file** fang\_et\_al\_genotypes.txt

- This is accomplished using the following code	

```
 cut -f 3 fang_et_al_genotypes.txt | uniq -c
```

By doing this I know how many lines with each group, and this will make it easier to check 
the extracted files.






