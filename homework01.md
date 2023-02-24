```shell
# count lines
wc -l test_command.gtf
# count characters
wc -m test_command.gtf

# find lines starting with 'chr_' and gene_id='YDL248W'
grep 'chr_' test_command.gtf | awk '$10="YDL268W"{print $0}'

# change 'chr_' to 'chromosome_' and print columns 1,3,4,5
sed 's/chr_/chromosome_/g' test_command.gtf | cut -f 1,3,4,5

# swap column 2 and 3, sort by 4 and 5, save results in result.gtf
cat test_command.gtf | awk '{t=$2;$2=$3;$3=t;print}' | sort -k 4 -n -k 5 -n > result.gtf

#change authorization
chmod u=rwx,go=r test_command.gtf
```
