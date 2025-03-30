#### 1. 请阐述bowtie中利用了 BWT 的什么性质提高了运算速度？并通过哪些策略优化了对内存的需求？
- 提高运算速度：BWT的LF-Mapping性质
  - 在Burrows-Wheeler Matrix (BWM)中，同一个字符在最后一列中的出现的rank（与自己相同的字母排序）与在第一列中出现的rank相同。
  - 利用这个性质，可以通过“walk-left”算法不断回溯字符串中字符前面的字符。这种映射方式能将长查询序列的匹配过程转化为高效的线性时间搜索。
  - 在实际应用中，根据输入序列进行渐进式的mapping，不断更新top和bot缩小搜索范围
- 内存优化策略：
  - BWT的压缩特性：在形成BWM时，sort往往使相同的字母拼在一起，能够减少内存占用。
  - milestone技术：为了解决存储整个后缀数组(SA)所需的巨大内存，Bowtie采用了“里程碑”策略。它只存储每32行一个样本，而不需要将整个后缀数组载入内存。
  - checkpoint和累积计数：为了提高查询时计算字符出现次数(rank)的效率，Bowtie在BWT矩阵中间定期设置检查点，利用预计算的累计计数来加速字符的排名查找，从而减少内存访问和计算时间。

#### 2. 用bowtie将 THA2.fa mapping 到 BowtieIndex/YeastGenome 上，得到 THA2.sam，统计mapping到不同染色体上的reads数量(即统计每条染色体都map上了多少条reads)。

```bash
bowtie -v 2 -m 10 --best --strata BowtieIndex/YeastGenome -f THA2.fa -S THA2.sam
less THA2.sam
cat THA2.sam | awk '$1 !~ /^@/ {print $3}' | sort | uniq -c
```
     92 *
     18 chrI
     51 chrII
     15 chrIII
    194 chrIV
     25 chrIX
     12 chrmt
     33 chrV
     17 chrVI
    125 chrVII
     68 chrVIII
     71 chrX
     56 chrXI
    169 chrXII
     67 chrXIII
     58 chrXIV
    101 chrXV
     78 chrXVI

#### 3. 查阅资料，回答以下问题:
- **3.1 什么是sam/bam文件中的"CIGAR string"? 它包含了什么信息?**
  - CIGAR string储存在SAM/BAM文件的第6列。它用于描述查询序列和参考序列之间的比对操作，由数字和字母组成，数字表示碱基数目，字母表示比对操作，如匹配、插入、删除等。常见的有：M (Match)，I (Insertion)，D (Deletion)，N (Skip)，S (Soft Clipping)，H (Hard Clipping)，X (Mismatch)等 。例如：3S5M
    <figure  style="text-align: center;">
    <img src="./hw4.1.jpg" style="width: 100%; height: auto;">
    </figure>

- **3.2 "soft clip"的含义是什么，在CIGAR string中如何表示？**
  - 用于描述那些一条序列上，在序列两端，比对不上的碱基序列；于是这些序列不用于比对，但是在BAM/SAM文件中的reads上还是存在的；在CIGAR string中用nS表示，其中n是软剪切的碱基数；H (Hard Clipping)与之相对，指完全删除两端比对不上的序列，不在reads中出现。

- **3.3 什么是reads的mapping quality? 它反映了什么样的信息?**
  - Mapping quality（MAPQ）储存在SAM/BAM文件的第五列。

- **3.4 仅根据sam/bam文件的信息，能否推断出read mapping到的区域对应的参考基因组序列?**

#### 4. 安装bwa软件，从UCSC Genome Browser下载Yeast (S. cerevisiae, sacCer3)基因组序列。使用bwa对Yeast基因组sacCer3.fa建立索引，并利用bwa将THA2.fa，mapping到Yeast参考基因组上，并进一步转化输出得到THA2-bwa.sam文件。

**bwa安装**
```bash
## download bwa-master
cd ~/mapping
wget https://github.com/lh3/bwa/archive/refs/heads/master.zip
unzip bwa-master.zip 

## define bwa
cd ~/mapping/bwa-master
make
alias bwa='~/mapping/bwa-master/bwa'
source ~/.bashrc
```

**genome下载**
<figure  style="text-align: center;">
<img src="./hw4.2.jpg" style="width: 80%; height: auto;">
</figure>
<figure  style="text-align: center;">
<img src="./hw4.3.jpg" style="width: 80%; height: auto;">
</figure>

**建立索引&mapping**
```bash
cd ~/mapping
mkdir bwa_index
cd bwa_index
bwa index ../sacCer3.fa
bwa mem ~/mapping/bwa_index/sacCer3.fa THA2.fa > THA2-bwa.sam
cat THA2-bwa.sam | awk '$1 !~ /^@/ {print}' | head -5
```
    1251-506        4       *       0       0       *       *       0       0       TTACTATTCTATTATTATACTTTATTG     *   AS:i:0   XS:i:0
    1252-505        4       *       0       0       *       *       0       0       TCACCACATAACTCTAAAATAAG *       AS:i:0       XS:i:0
    1253-504        4       *       0       0       *       *       0       0       ATATATATATATATATATACATAATATAATTTAACG*AS:i:0  XS:i:0
    1254-504        4       *       0       0       *       *       0       0       TTTTATACTATTATCTTTTTAATTAATG    *   AS:i:0   XS:i:0
    1255-503        4       *       0       0       *       *       0       0       ACCAATACATAACAAACCCTTTTG        *   AS:i:0   XS:i:0

