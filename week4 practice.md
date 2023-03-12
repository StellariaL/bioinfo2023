code:
```bash
#!/bin/bash
# Get filepath.
echo -n "please input a directory path:"

read path
n=0
# Determine whether the input is a directory. If not, ask for new input. There are 5 chances to correct the input.
while true
do
	if [ -d $path ];then
		break;
	elif [ "$n" -eq "5" ];then
	       echo "too many wrong inputs"
       exit 0
else
	((n++))
	echo -n "not directory, please input again:"
	read path
fi
done

# List contents under the given directory.
cur=`ls ${path}`
# Determine whether each content is a file or a directory.
for val in $cur
do
abs=${path}/${val}
	if [ -f $abs ];then
	echo "FILE: $val" >> filename.txt
elif [ -d $abs ];then
	echo "DIR: $val" >> dirname.txt
else
	echo "${val} is not file or directory"
fi
done

exit 0
```

output:
filename.txt:
```
FILE: a1.txt
FILE: a.txt
FILE: b1.txt
FILE: bam_wig.sh
FILE: b.filter_random.pl
FILE: c1.txt
FILE: chrom.size
FILE: c.txt
FILE: d1.txt
FILE: dir.txt
FILE: e1.txt
FILE: f1.txt
FILE: human_geneExp.txt
FILE: if.sh
FILE: image
FILE: insitiue.txt
FILE: mouse_geneExp.txt
FILE: name.txt
FILE: number.sh
FILE: out.bw
FILE: random.sh
FILE: read.sh
FILE: test3.sh
FILE: test4.sh
FILE: test.sh
FILE: test.txt
FILE: wigToBigWig
```

dirname.txt:
```
DIR: a-docker
DIR: app
DIR: backup
DIR: bin
DIR: biosoft
DIR: c1-RBPanno
DIR: datatable
DIR: db
DIR: download
DIR: e-annotation
DIR: exRNA
DIR: genome
DIR: git
DIR: highcharts
DIR: home
DIR: hub29
DIR: ibme
DIR: l-lwl
DIR: map2
DIR: mljs
DIR: module
DIR: mogproject
DIR: node_modules
DIR: perl5
DIR: postar2
DIR: postar_app
DIR: postar.docker
DIR: RBP_map
DIR: rout
DIR: script
DIR: script_backup
DIR: software
DIR: tcga
DIR: test
DIR: tmp
DIR: tmp_script
DIR: var
DIR: x-rbp
```
