#Data Inspection:
####Inspection of `fang_et_al_genotypes.txt`
```
$ du -h fang_et_al_genotypes.txt
$ wc fang_et_al_genotypes. txt
$ column -s "," -t fang_et_al_genotypes.txt | (head -n 2; tail -n 2)| less
$ tail -n +3 fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'

```
After inspection of this file I found-

1. The file size is 11M which I found by using du -h command. 
The wc command showed 2783 lines, 2744038 words and 11051939 characters/bytes in the file.
 
2. Using the column -s command I was able to look at the header and the tail of file. I also looked at the first two lines and the last two lines. Then I piped this information to less to navigate throughout the lines.
3. While lookin at the tail, I combines the awk command to print out the number of columns. I found out there were 986 columns in this documents

#Data Processing:
### Processing Maize Files: 

```
1.	head -n 1 fang_et_al_genotypes.txt >maize_genotypes.txt | grep "ZMMIL\|ZMMLR\|ZMMMR" 
	fang_et_al_genotypes.txt >> maize_genotypes.txt
```
At first I have extracted the header of fang_at_al_genotype.txt file into a new maize_genotypes.txt file. Then I grepped 3 groups of maize from fang_at_al_genotype.txt and appended into maize_genotypes.txt file. 

```
2.	awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
```

Then by using awk command I transposed the maize_genotypes.txt file.

```
3.	sed 's/Sample_ID/SNP_ID/' transposed_maize_genotypes.txt > 
	SNPIDinheader_transposedmaize_genotypes.txt
```
I replaced ‘Sample_ID’ to ‘SNP_ID’ in the first column of transposed_maize_genotypes.txt file so that I can join this file with the snp_position.txt file later.

```
4.	head -n 1 SNPIDinheader_transposedmaize_genotypes.txt > Final_transposed_sorted_maize.txt 
5.	tail -n +2 SNPIDinheader_transposedmaize_genotypes.txt > header_missing_transposed_maize.txt 
6.	sort -k 1 header_missing_transposed_maize.txt >> Final_transposed_sorted_maize.txt
```
In the commands 4 to 6, at first, I separated the header of transposed maize genotypes file into a new file naming “Final_transposed_sorted_maize.txt” file. Then, sorted the transposed maize genotypes file without the header. Because sorting with header resulted into inserting the header row with “SNP_ID” in the middle of the text file. At last, I appended this sorted file in the Final_transposed_sorted_maize.txt file.

### Processing Teosinte Files: 
```
1.	head -n 1 fang_et_al_genotypes.txt >teosinte_genotypes.txt | grep "ZMPBA\|ZMPIL\|ZMPJA" 
	fang_et_al_genotypes.txt >> teosinte_genotypes.txt
2.	awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
3.	sed 's/Sample_ID/SNP_ID/' transposed_teosinte_genotypes.txt > 
	SNPIDinheader_transposed_teosinte_genotypes.txt
4.	head -n 1 SNPIDinheader_transposed_teosinte_genotypes.txt > 
	Final_transposed_sorted_teosinte.txt 
5.	tail -n +2 SNPIDinheader_transposed_teosinte_genotypes.txt > 
	header_missing_transposed_teosinte.txt 
6.	sort -k 1 header_missing_transposed_teosinte.txt >> Final_transposed_sorted_teosinte.txt
```
I Processed all the teosinte files from fang_at_al_genotypes.txt file as the same way I processed maize files.

### Processing SNP_position.txt file:

```
1.	cut -f 1,3,4 snp_position.txt > extracted_snp_positions.txt
```
In the first command I have extracted the columns with SNP_ID, Chromosomes and Positions that were in column number 1, 3 and 4.

```
2.	head -n 1 extracted_snp_positions.txt > Final_snp_positions.txt
3.	tail -n +2 extracted_snp_positions.txt > header_missing_snp_positions.txt 
4.	sort -k 1 header_missing_snp_positions.txt >> Final_snp_positions.txt
```
Then in commands 2 to 4, at first, I separated the header of this extracted_snp_positions.txt file to a new file naming “Final_snp_positions.txt” file. Then sorted the extracted_snp_positions.txt file without header and appended in the 
Final_snp_positions.txt file.

#### Joining respective maize and teosinte files with Final_snp_positions.txt file:

```
1.	join -1 1 -2 1 -t $'\t' Final_snp_positions.txt Final_transposed_sorted_maize.txt > 
	joined_maize_snp.txt
2.	join -1 1 -2 1 -t $'\t' Final_snp_positions.txt Final_transposed_sorted_teosinte.txt > 
	joined_teosinte_snp.txt
```
I joined each respective maize and Teosinte files with Final_snp_positions.txt file using join command. The command joined the two based on the similarities on the first column of the files. The option -t $'\t' is to use tab as a field separator.

### Sorting Maize Chromosome files:

```
grep "?" joined_maize_snp.txt > missing_data_joined_maize_snp.txt
```
At first I used grep command to grep the missing genotype data with “?” mark. (Then checked the file size and found all the SNPs have miising genotype data).

```
awk '$2 ~ /^1$/' missing_data_joined_maize_snp.txt > maize_chr1.txt
```
For extracting chromosome files, I used awk to select on the 2nd field and used (/^1$/') option to select everything that matches with that chromosome number. By the same command I created total 10 chromosome files for maize according to the respective chromosome numbers.

```
sort -k3,3n maize_chr1.txt > maize_chr1_ascending.txt
```
Then I sorted all the maize chromosome files in ascending order according to the 3rd column as SNP positions were in 3rd column.

```
sed 's/?/-/g' maize_chr1.txt | sort -k3,3nr > maize_chr1_descending.txt
```
To sort the files in descending order, according to the third column, I used sort -k3,3nr command. Before that I replaced the “?” mark with “-“ mark by sed command as mentioned in the homework instruction.

### Sorting Teosinte Chromosome files:

```
grep "?" joined_teosinte_snp.txt > missing_data_joined_teosinte_snp.txt
awk '$2 ~ /^1$/' missing_data_joined_teosinte_snp.txt > teosinte_chr1.txt
sed 's/?/-/g' teosinte_chr1.txt | sort -k3,3nr > teosinte_chr1_descending.txt     
```
