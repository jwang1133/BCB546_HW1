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

##Subsetting data
**1.Before transposing the data, I need to subset the data for the maize group and the teosinte group. And both of those two files need to have the header from file** fang\_et\_al\_genotypes.txt

- **Following code were used to accomplish this part**

```
head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt
```

\##this line of code write the **header line** to the file **maize\_genotypes.txt**,so that after we transpose the file, we can have the **SNP\_ID** in the file.

```
grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt >> maize_genotypes.txt
```

```
head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt
```

```
grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt >> teosinte_genotypes.txt
```

**2.Then I count the number of rows and columns in the new file to see whether it match the expected number of lines within each group (maize:1573, teosinte:975)**

```
wc -l maize_genotypes.txt  teosinte_genotypes.txt
```

```
awk -F "\t" '{print NF; exit}'  maize_genotypes.txt 
```

```
awk -F "\t" '{print NF; exit}' teosinte_genotypes.txt
```



