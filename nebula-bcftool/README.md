# Searching entire nebula.org variant calling file for pathogenic variants
https://jiggermast.blogspot.com/2024/09/searching-nebulaorg-variant-calling.html
## Enrich my VCF file using bcftool with pathological information CLNSIG from ClinVar VCF reference file

https://samtools.github.io/bcftools/bcftools.html#annotate
# use bcftool supplied in docker image to avoid local installation
docker run -it -v /Users/william/workspaces/genetics/vep_data:/data mcfonsecalab/variantutils  

# download VCF file from nebula to localhost
MYNEBULAVCF.mm2.sortdup.bqsr.hc.vcf.gz

# index VCF if not downloaded from nebula
tabix -p vcf MYNEBULAVCF.mm2.sortdup.bqsr.hc.vcf

# download ClinVar VCF reference
wget ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh38/clinvar.vcf.gz

# decompress the clinvar.vcf.gz and copy it to a new file
bgzip -d clinvar.vcf.gz
cp clinvar.vcf clinvar.vcf.txt

# add "chr" to the chromosome number (CHROM) in the decompressed ClinVar file so that it will joined with the nebula VCF on CHROM and POS fields 
awk '{if($1 ~ /^[0-9XYM]/) print "chr"$1substr($0, length($1) + 1); else print $0}' clinvar.vcf.txt > clinvar-adjusted.vcf.txt

# compress the adjusted file and index it
bgzip clinvar-adjusted.vcf.txt
tabix -p vcf clinvar-adjusted.vcf.txt.gz

# annotate file
bcftools annotate -a clinvar-adjusted.vcf.txt.gz -c CHROM,POS,INFO/CLNSIG -o MYNEBULAVCF-annotated.vcf -O z HHTLNHYJ4.mm2.sortdup.bqsr.hc.vcf.gz

# add gene name to VCF enriched file
wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_42/gencode.v42.annotation.gtf.gz

# decompress
bgzip -d gencode.v42.annotation.gtf.gz

#convert to BED file format
cat gencode.v42.annotation.gtf | awk '$3 == "gene"' | awk 'BEGIN{OFS="\t"} {print $1, $4-1, $5, $14}' > gencode_genes.bed

# add information to INFO field from gencode_genes.bed
bcftools annotate -a gencode_genes.bed -h <(echo '##INFO=<ID=GENE,Number=1,Type=String,Description="Gene name">') -c CHROM,FROM,TO,GENE -o HHTLNHYJ4-annotated_with_genes.vcf -O z HHTLNHYJ4-annotated.vcf 


# filter pathogenic variants with grep
grep Pathogenic MYNEBULAVCF-annotated_with_genes.vcf  > MYNEBULAVCF-pathogenic-filtered.vcf

# get just gene name
grep Patho  MYNEBULAVCF-annotated_with_genes.vcf | sed  -n 's/.*GENE="\([^"]*\)".*/\1/p' 

# sort, filter unique and make a comma separated list
sort -u risk-genes.txt |  tr '\n' ','

# gene list can be used in the gene.io.bio website
PERM1, ABCA4, EHBP1, WNT10A, AIMP1, ENPP1, SBDS, PRSS1, ADRA2A, ASXL3

#####################
## findings
# WNT10A
Schöpf–Schulz–Passarge syndrome
https://www.ncbi.nlm.nih.gov/clinvar/variation/4461/

# ADRA2A
https://www.ncbi.nlm.nih.gov/clinvar/variation/2691730/

# AIMP1 (****)
https://www.ncbi.nlm.nih.gov/clinvar/variation/590774/

# ASXL3 -
33,578,219 - 33,751,195
             33,743,122

             remaining nucleotides after mutation: 8073 / 3 = 2,691 amino acids

total nucleotides: 172,976 / 3   57,658

2,691/57,658 = 4.67 % defective

# AMP1
106,315,544 - 106,349,456
total nucleotides  = 33,912

# Other
#
docker run -t -i -v /Users/william/workspaces/genetics/vep_data:/data ensemblorg/ensembl-vep
docker run -t -i -v /Users/william/workspaces/genetics/vep_data:/data mcfonsecalab/variantutils  

# run in docker image
gene-iobio-flagged-variants.vcf
vep --cache --dir_cache /data/cache -i /data/HHTLNHYJ4.vcf --plugin AlphaMissense,file=/data/AlphaMissense_hg38.tsv.gz,transcript_match=1 --force_overwrite
vep --cache --dir_cache /data/cache -i /data/gene-iobio-flagged-variants.vcf --plugin AlphaMissense,file=/data/AlphaMissense_hg38.tsv.gz,transcript_match=1 --force_overwrite


# run check
~/.vep$ vep --cache --dir_cache /data/cache --fork 4  -i HHTLNHYJ4.mm2.sortdup.bqsr.hc.vcf --output_file annotated_output.vcf --force_overwrite  --vcf


