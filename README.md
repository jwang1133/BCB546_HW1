## BCB546X\_Spring2017\_UNIX Assignment##
## Jinyu Wang

----------
 

##Inspect data  





**1.Using the following code to inspect the size, row number and column number for both of the files**

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


- **fang\_et\_al\_genotypes.txt** (size 11M, #row 2783, #column 986)
- **snp\_position.txt** (size 84k, #row 984, #column 986)
 
**2.Inspect both of the files to check the header and determine based on which column/filed to join those two files**

```
 head fang_et_al_genotypes.txt
```

```
 head snp_position.txt
```

- From the above two line code I could see that those two files containing different number of column headers and the no common column for join.
- Determined that I need to transpose the data before join the files.

**3.check the number of lines within each group in file** fang\_et\_al\_genotypes.txt

- This is accomplished using the following code	

```
 cut -f 3 fang_et_al_genotypes.txt | uniq -c
```

By doing this I know how many lines with each group, and this will make it easier to check 
the extracted files in next step.

##Subset data
**1.Before transposing the data, I need to subset the data for the maize group and the teosinte group. And both of those two files need to have the header from file** fang\_et\_al\_genotypes.txt

- **Following code were used to accomplish this part**

```
head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt
```

\##this line of code write the **header line** to the file **maize\_genotypes.txt**,so that after we transpose the file, we can have the **SNP\_ID** in the file.

- **Next I subset the data for the maize group and the teosinte group**

```
grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt >> maize_genotypes.txt
```

```
head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt
```

```
grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt >> teosinte_genotypes.txt
```

**2.Then I count the number of rows and columns in the new files to see whether they match the expected number of lines within each group (maize:1573, teosinte:975)**

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

**2. Then I extract the first, third and fourth column of** snp\_position.txt **file. They are respectly for SNP_ID, Chromosome and Position**

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
awk '$2==1' maize_genotypes_joined.txt | sed 's/-/?/g'| sort -k3,3n | tee maize_chr1.txt | sed 's/?/-/g' | sort -k3,3nr > maize_chr1_rev.txt
``` 

For the above code: 1. before the first pipe, it is trying to match by column. When the second column equals 1 it send the output to the second program;2. 3. It sort the output from second program based on the third column (chromosome_position increasing order), and then send the output to the fourth program; 4. Create a intermediate file which contains the chromosome1 information and has been sorted based on chromosome_position increasing order; 5. Replace the "?" with "-" from the output of third program and then send it to the fifth program; 6, Sort the output from fourth program based on chromosme_position decreasing order and then write the output to file named maize\_chr1\_rev.txt.

**2. For the teosinte group (the code is the same as before)**

```
awk '$2==1' teosinte_genotypes_joined.txt | sed 's/-/?/g'| sort -k3,3n | tee teosinte_chr1.txt | sed 's/?/-/g' | sort -k3,3nr > teosinte_chr1_rev.txt

```

**3. Inspect the subset files to make sure they are correct and also check the file size for each of them**

```
wc -l *_chr*.txt
```

```
du -h *_chr*.txt
```

##Add the header line to the subset files
**1. create the header line for the maize group subset files and the teosinte subset files**

```
head -n 1 transposed_maize_genotypes.txt | cut -f2-1574 > maize_file_sample_header.txt
```
\##cut the corresponding columns from the transposed\_maize\_genotypes.txt to get only the sample_ID and write it to file maize\_file\_sample\_header.txt

```
head -n 1 transposed_teosinte_genotypes.txt | cut -f2-976 > teosinte_file_sample_header.txt
```

```
head -n 1 snp_position.txt | cut -f1,3,4  > snp_file_header.txt
```
\##cut the corresponding columns from the snp\_position.txt to get the column head of SNP_ID, Chromosome, Position and write it to file snp\_file\_header.txt

```
paste snp_file_header.txt maize_file_sample_header.txt > maize_file_header.txt
```

```
paste snp_file_header.txt teosinte_file_sample_header.txt > teosinte_file_header.txt
```


**2. Inspect the created header files and make sure they are correct**

```
awk -F "\t" '{print NF; exit}' maize_file_header.txt
```

```
awk -F "\t" '{print NF; exit}' teosinte_file_header.txt
```

**3. cat the subset files with the header files with for loop**

```
for ((i=1; i<=10; i+=1)); do cat maize_file_header.txt  "maize_chr"$i".txt" > "maize_genotypes_chr"$i".txt"; done
```

```
for ((i=1; i<=10; i+=1)); do cat maize_file_header.txt  "maize_chr"$i"_rev.txt" > "maize_genotypes_chr"$i"_rev.txt"; done
```

```
 for ((i=1; i<=10; i+=1)); do cat teosinte_file_header.txt  "teosinte_chr"$i".txt" > "teosinte_genotypes_chr"$i".txt"; done
```

```
for ((i=1; i<=10; i+=1)); do cat teosinte_file_header.txt  "teosinte_chr"$i"_rev.txt" > "teosinte_genotypes_chr"$i"_rev.txt"; done
```

**4. Inspect the new created files and make sure they are correct**

```
wc -l *_genotypes_chr*.txt 
```

```
awk -F "\t" '{print NF; exit}' maize_genotypes_chr1.txt
```

```
awk -F "\t" '{print NF; exit}' teosinte_genotypes_chr1.txt
```
