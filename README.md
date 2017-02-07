## BCB546X\_Spring2017\_UNIX Assignment##
## Jinyu Wang

----------
 

##Inspect data  





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

##Subset data
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
##Transpose data

**1.Next I need to transpose the data so that the SNP\_ID could be in one column and can be used as a common column to join two files**

```
awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
```

```
awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
```

**2. Then I inspected rows of those two transposed files to make sure it is transposed correctly**

```
wc -l transposed_maize_genotypes.txt transposed_teosinte_genotypes.txt
```

##Sort and join data

**1.Next I remove the first three lines of file** transposed\_maize\_genotypes.txt  and transposed\_teosinte\_genotypes.txt. **As they will cause problem for data sorting and they are not necessary for data joining.**

```
tail -n +4 transposed_maize_genotypes.txt > clean_transposed_maize_genotypes.txt
```

```
tail -n +4 transposed_teosinte_genotypes.txt > clean_transposed_teosinte_genotypes.txt
```

**2. Then I extract the first, third and fourth column of** snp\_position.txt **file. They are respectly for SNP_ID, Chromosome and position**

```
cut -f1,3,4 snp_position.txt | tail -n +2 > clean_snp_position.txt
```

I also inspect the extracted files make sure it is correct

```
wc -l clean_snp_position.txt
```

**3. After this, I sort the files** clean\_transposed\_maize\_genotypes.txt, clean\_transposed\_teosinte\_genotypes.txt and clean\_snp\_position.txt **based on the first column SNP_ID.**

```
sort -k1,1 clean_transposed_maize_genotypes.txt > clean_transposed_maize_genotypes_sorted.txt
```

```
sort -k1,1 clean_transposed_teosinte_genotypes.txt > clean_transposed_teosinte_genotypes_sorted.txt
```

```
sort -c -k1,1 clean_snp_position.txt
echo $?
0
```
\##This code check that the clean\_snp\_position.txt file has already sorted based on the first column

**4. join the data** clean\_snp\_position.txt **respectly with** clean\_transposed\_maize\_genotypes\_sorted.txt  and clean\_transposed\_teosinte\_genotypes\_sorted.txt** **based on the common column SNP_ID.**

```
join -t $'\t' -1 1 -2 1 clean_snp_position.txt clean_transposed_teosinte_genotypes_sorted.txt > teosinte_genotypes_joined.txt
```

```
join -t $'\t' -1 1 -2 1 clean_snp_position.txt clean_transposed_maize_genotypes_sorted.txt > maize_genotypes_joined.txt
```

Also inspect the files to make sure the joined files are correct

```
head maize_genotypes_joined.txt
```

```
awk -F "\t" '{print NF; exit}' maize_genotypes_joined.txt
```

```
wc -l maize_genotypes_joined.txt
```

##Subset and sort data for each chromosome for each group
**1. For the maize group**

- **The following line of code was used to subset and sort data for each chromosome accordingly. Here I only paste the code for chromsome1 and chromosome2. To do the process for other chromosme, just need to change the number accordingly.**

```
awk '$2==1' maize_genotypes_joined.txt | sort -k3,3n | tee maize_chr1.txt | sed 's/?/-/g' | sort -k3,3nr > maize_chr1_rev.txt
``` 

For the above code: 1. before the first pipe, it is trying to match by column. When the second column equals 1 it send the output to the second program; 2. It sort the output from first program based on the third column (chromosome_position increasing order), and then send the output to the third program; 3. Create a intermediate file which contains the chromosome1 information and has been sorted based on chromosome_position increasing order; 4. Replace the "?" with "-" from the output of second program and then send it to the fourth program; 5, Sort the output from third program based on chromosme_position decreasing order and then write the output to file named maize\_chr1\_rev.txt.

**2. For the teosinte group**

```
awk '$2==1' teosinte_genotypes_joined.txt | sort -k3,3n | tee teosinte_chr1.txt | sed 's/?/-/g' | sort -k3,3nr > teosinte_chr1_rev.txt

```

**3. Inspect the subsetted files**

```
wc -l *_chr*.txt
```

```
du -h *_chr*.txt
```

