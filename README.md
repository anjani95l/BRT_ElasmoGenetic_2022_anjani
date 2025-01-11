# BRT_ElasmoGenetic_2022_anjani
# Species ID workflow with ONT amplicon data Using R
This pipeline is designed for processing and analyzing Oxford Nanopore Technologies (ONT) MinION DNA-barcoding sequence data for species identification. Assumes barcoded reads sequenced on a Flongle flow cell.

# 1) Define directories

 Example Windows paths (update to match your environment):
fast5_dir  <- "C:/Users/user1/Desktop/minION_workflow/data/fast5"
out_dir    <- "C:/Users/user1/Desktop/minION_workflow/data/basecalled

# Input files
make a windows compatible file path

Location_win<-str_replace_all(Location,"/","\\\\")
Location_linux<-str_sub(Location,4,str_length(Location))

# Basecalling 
This will run guppy base caller and dump the outputs into the fastq file

input_path<-paste0('"',Location,'/fast5"')
save_path<-paste0('"',Location,'/fastq"')
args_basecall<-paste0('"C:/Program Files/OxfordNanopore/ont-guppy/bin/guppy_basecaller.exe"',' --input_path ',input_path,' --save_path ',save_path,' -x auto --flowcell FLO-FLG114 --kit SQK-RBK114-24')
system('cmd.exe',input = args_basecall,invisible = F)
#location of the fast5 files eg C:/Anjanisequencingtrial/Elasmo01/1to26/20240404_1534_MN40942_ATH520_4c3c63c3
Location<-"C:/Anjanisequencingtrial/C065_20240706/C065GENOMESKINMMING/20240706_1821_MN40942_ATR438_35422216/"
library(tidyverse)


# Concatenate files
catting the output

args_cat= paste0('type "',Location_win,'\\fastq\\pass\\*.fastq" > "',Location_win,'\\passed_reads\\Merged.fastq"')
dir.create(paste0(Location,"/passed_reads"))
system('cmd.exe', input=args_cat)
#location of the fast5 files eg C:/Anjanisequencingtrial/Elasmo01/1to26/20240404_1534_MN40942_ATH520_4c3c63c3
Location<-"C:/Anjanisequencingtrial/C059Genome_Skimming/c05920240707/20240707_1530_MN40942_ATR415_5edbe28d/"
library(tidyverse)

make a windows compatible file path"
Location_win<-str_replace_all(Location,"/","\\\\")
Location_linux<-str_sub(Location,4,str_length(Location))


# Adaptor trimming and demultiplexing
demultiplex our data according to barcode and trim barcodes of sequencing reads. The barcode kit is specified in the script, in this case: SQK-RBK114-24

args_barcode<-paste0('"C:/Program Files/OxfordNanopore/ont-guppy/bin/guppy_barcoder.exe" -i "',Location_win,'\\passed_reads" -s "',Location_win,'\\demultiplexed" --barcode_kits SQK-RBK114-24 --records_per_fastq 0 -x auto')
system('cmd.exe', input=args_barcode)

# Filtering
filter our fastq files for quality and read length. This step is optional as NGSpeciesID performs well without filtered reads.


# NGSpeciesID
