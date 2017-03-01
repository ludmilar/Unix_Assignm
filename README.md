


> Written with [StackEdit](https://stackedit.io/).

##__Data Inspection__

1. `cd UNIX_Assignment`
2. `file fang_et_al_genotypes.txt` ASCII text, with very long lines
3.  `ls -lh`  
 * fang_et_al_genotypes.txt  - size 11M
 * snp_position.txt - size 81K
4. `wc fang_et_al_genotypes.txt snp_position.txt ` will count words:
2783 2744038 11051939 fang_et_al_genotypes.txt
     984   13198   82763 snp_position.txt
5. `wc -l fang_et_al_genotypes.txt  snp_position.txt` just lines: 
*2783 in fang_et_al_genotypes.txt
*984 in snp_position.txt
6. Do those files have any empty lines? `grep -c "[^ \\n\\t]" snp_position.txt fang_et_al_genotypes.txt` will exclude empty spaces:
Got same numbers -> no empty lines.
7. How many columns? `awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt`
* 986 - in fang_et_al_genotypes.txt
* 15  - in snp_position.txt
8. To get idea of what column and raws are there I used `cut -f1-15 fang_et_al_genotypes.txt | head -n 5`. It will show first 15 columns and 5 raws
9.  To make more clear idea will print 4 column, only 2 lines: `awk '{ print $1 "\t" $3 "\t" $4 "\t" $6 }' fang_et_al_genotypes.txt | column -t | head -n 3`  Got result:

> Sample_ID    Group  abph1.20  ae1.3 SL-15        TRIPS  ?/?       T/T
> SL-16        TRIPS  ?/?       T/T

## Editing
1 . to take only maize: including header:
2. `head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt `
then `grep -E 'ZMMIL|ZMMLR|ZMMMR' fang_et_al_genotypes.txt >> maize_genotypes.txt`

3 same for teosinte `head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt `
then `grep -E 'ZMPBA|ZMPIL|ZMPJA' fang_et_al_genotypes.txt >> teosinte_genotypes.txt`

4. now transpose: `awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt`
`awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt`

Did transpose worked? `wc -l transposed_maize_genotypes.txt transposed_teosinte_genotypes.txt
` - both shown 986. 
Worked!
5. We need only 3 columns from file snp_position.txt, so I created new file with only 3 column:
`cut -f 1,3,4 snp_position.txt > snp_position_3.txt` 
In addition I also change name of I column in genotypes `sed 's/Sample_ID/SNP_ID/' transposed_maize_genotypes.txt > new_transposed_maize_genotypes.txt`

# Join

1. Combine 2 files into new one: `join -t $'\t' -1 1 -2 1 snp_position_3.txt new_transposed_maize_genotypes.txt > maize_all_results.txt`
2. `bioawk -c hdr '$Chromosome==2 {print $0}' maize_all_results.txt > maize_chr2.txt` 
Hear I look for chr2 in column "chromosome" and return entire line.
3. Sort in increase order:
`sort -k3  teosinte_chr1.txt > teosinte_chr1_sortIN.txt`
4. To replace "?/?" for "-/-" needed to run: `sed 's/?/-/g' teosinte_chr1_sortRev.txt >teosinte_chr1_sortRev_replaced.txt` 
5. Put all files into new dir: `mkdir results-$(date +%F)` then `mv maize_chr?.txt results-2017-02-07`
# loop to create file by chromozons:

'for i in {1..10}; do awk '$2=='$i'' maize_genotypes_joined.txt | sed 's/?/-/g'| sort -k3,3n | tee maize_chr$i.txt | sed 's/-/?/g' | sort -k3,3nr > maize_chr"$i"_rev.txt; done'

