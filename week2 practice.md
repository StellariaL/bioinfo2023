Step 0: unzip file 1.gtf
![alt text][step 0]

Step 1:
1. print head and tail with ```cat```
   ![alt text][step 1 cat]
2. check file size with ```ls -lh``` or ```wc -l```
   ![alt text][step 1 size]
3. remove header and 0-length lines
   ![alt text][step 1 grep]
4. 2 ways to remove blank lines

   remove blank lines with ```awk```
   ![alt text][step 1 remove blank]
   remove blank lines with ```grep```
   ![alt text][step 1 remove blank 2]

Step 2:

Step 2.1: select columns with ```awk``` or ```cut```
   ![alt text][step 2.1 select columns]
   
Step 2.2: select rows with ```awk```
   ![alt text][step 2.2]
   
   
Step 3:

Step 3.1: count features with ```grep```, ```sort``` and ```uniq -c```
   ![alt text][step 3.1]
   
Step 3.2:
1. calculate length with ```awk```
   ![alt text][step 3.2 length]
2. calculate sum of lengths with ```awk```
   ![alt text][step 3.2 sum]
3. calculate mean length with ```awk```
   ![alt text][step 3.2 mean]
   
Step 3.3: extract gene names with ```split```
   ![alt text][step 3.3]


[step 0]: https://github.com/StellariaL/bioinfo2023/blob/main/step%200.png
[step 1 cat]: https://github.com/StellariaL/bioinfo2023/blob/main/step%201%20cat.png
[step 1 size]: https://github.com/StellariaL/bioinfo2023/blob/main/step%201%20size.png
[step 1 grep]: https://github.com/StellariaL/bioinfo2023/blob/main/step%201%20grep.png
[step 1 remove blank]: https://github.com/StellariaL/bioinfo2023/blob/main/step%201%20remove%20blank.png
[step 1 remove blank 2]: https://github.com/StellariaL/bioinfo2023/blob/main/step%201%20remove%20blank%202.png
[step 2.1 select columns]: https://github.com/StellariaL/bioinfo2023/blob/main/step%202.1%20select%20columns.png
[step 2.2]: https://github.com/StellariaL/bioinfo2023/blob/main/step%202.2.png
[step 3.1]: https://github.com/StellariaL/bioinfo2023/blob/main/step%203.1.png
[step 3.2 length]: https://github.com/StellariaL/bioinfo2023/blob/main/step%203.2%20length.png
[step 3.2 sum]: https://github.com/StellariaL/bioinfo2023/blob/main/step%203.2%20sum.png
[step 3.2 mean]: https://github.com/StellariaL/bioinfo2023/blob/main/step%203.2%20mean.png
[step 3.3]: https://github.com/StellariaL/bioinfo2023/blob/main/step%203.3.png
