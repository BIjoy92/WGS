### SRA 파일 사용  
SRAtoolkit download  
https://github.com/ncbi/sra-tools/wiki/01.-Downloading-SRA-Toolkit  

`tar -xzvf sratoolkit.tar.gz`  
환경변수 설정 - sratoolkit의 bin 파일이 있는 곳을 전체에서 사용할 수 있도록 해준다.  

SRR 데이터의 크기가 20GB 이상이면 fastq-dump로 받기가 힘들다. 따라서 이 방법을 사용  
`prefetch --max-size 30G SRR8454589`  

받은 파일이 fastq paired-end 파일이므로 fasterq-dum --split-files를 이용하여 파일을 나눠준다.  
`fasterq-dump --split-files SRR8454589.sra`  


