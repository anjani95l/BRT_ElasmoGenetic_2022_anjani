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

# 2) Basecalling with Guppy on Windows

This will run guppy base caller and dump the outputs into the fastq file

input_path <- paste0('"', Location, "/fast5\"")  # In double quotes for cmd.exe

save_path  <- paste0('"', Location, "/fastq\"")

Construct the command to basecall the FAST5 data

args_basecall <- paste0(

  '"C:/Program Files/OxfordNanopore/ont-guppy/bin/guppy_basecaller.exe"',
  
  " --input_path ", input_path,
  
  " --save_path ", save_path,
  
  " -x auto",                   # auto-detect GPU or CPU
  
  " --flowcell FLO-FLG114",     # Adjust to your flow cell
  
  " --kit SQK-RBK114-24"        # Adjust to your barcoding kit
)

Execute the basecalling

system("cmd.exe", input = args_basecall, invisible = FALSE)

# 3) Concatenate files

Concatenate all 'pass' reads into a single file

We'll create a new folder "passed_reads" and merge into "Merged.fastq".

dir.create(file.path(Location, "passed_reads"), showWarnings = FALSE)

Use Windows 'type' command to merge all fastq files in pass/
args_cat <- paste0(
  'type "', Location_win, '\\fastq\\pass\\*.fastq" > "',
  Location_win, '\\passed_reads\\Merged.fastq"'
)

# Run the concatenation
system("cmd.exe", input = args_cat)

# make a windows compatible file path"
Location_win<-str_replace_all(Location,"/","\\\\")
Location_linux<-str_sub(Location,4,str_length(Location))


# 4) Adaptor trimming and demultiplexing
demultiplex our data according to barcode and trim barcodes of sequencing reads. The barcode kit is specified in the script, in this case: SQK-RBK114-24

args_barcode <- paste0(

  '"C:/Program Files/OxfordNanopore/ont-guppy/bin/guppy_barcoder.exe"',
  
  ' -i "', Location_win, '\\passed_reads"',
  
  ' -s "', Location_win, '\\demultiplexed"',
  
  ' --barcode_kits SQK-RBK114-24',
  
  ' --records_per_fastq 0',
  
  ' -x auto'
)

system("cmd.exe", input = args_barcode, invisible = FALSE)


# 5) Filtering

filter our fastq files for quality and read length. This step is optional as NGSpeciesID performs well without filtered reads.
Gunzip step

  Note: Windows does not have 'gunzip' natively; you must have it installed  and accessible in your PATH. Otherwise, adapt to 7-Zip or another tool.
  
   gunzip_cmd <- paste0('gunzip "', TARGET_DIR_WIN, '\\', f, '\\*.*"')
   
  If your files are specifically named *.fastq.gz, then do:
  
  gunzip_cmd <- paste0('gunzip "', TARGET_DIR_WIN, '\\', f, '\\*.fastq.gz"')

  Execute gunzip (ignore warnings if files are already unzipped)
  
  system("cmd.exe", input = gunzip_cmd, invisible = FALSE)

  We apply -q 7 (quality filter), -l 100 (min length 100), and --maxlength 750
  
  using input redirection (<) and output redirection (>) just like your Bash script.
  
  nanoFilt_cmd <- paste0(
  
    '"', nanoFilt_path, '" ', 
    
    '-q 7 -l 100 --maxlength 750 ',
    
    '< "', TARGET_DIR_WIN, '\\', f, '\\*" ',
    
    '> "', TARGET_DIR_WIN, '\\', f, '\\', f, '_filtered.fastq"'
    
  )

  Execute NanoFilt
  system("cmd.exe", input = nanoFilt_cmd, invisible = FALSE)


# 6) NGSpeciesID

Define directories (adjust these paths for your setup on Windows)
###############################################################################
TARGET_DIR <- "C:/Users/genetics/Desktop/minION_species_ID-master/data/out/demultiplexed"
OUT_DIR    <- "C:/Users/genetics/Desktop/minION_species_ID-master/data/out/ngspecies_id"
PRIMER_DIR <- "C:/Users/genetics/Desktop/demultiplexed/Primer2.txt"  # Not directly used below

# Convert to backslashes for Windows commands
TARGET_DIR_WIN <- str_replace_all(TARGET_DIR, "/", "\\\\")
OUT_DIR_WIN    <- str_replace_all(OUT_DIR,  "/", "\\\\")

###############################################################################
# 2) Define barcodes
###############################################################################
barcodes <- sprintf("barcode%02d", 1:24) 
# This generates "barcode01", "barcode02", ..., "barcode24"

###############################################################################
# 3) Create output directories for each barcode
###############################################################################
for (bc in barcodes) {
  dir.create(file.path(OUT_DIR, bc), showWarnings = FALSE, recursive = TRUE)
}

###############################################################################
# 4) Path to NGSpeciesID
###############################################################################
# If NGSpeciesID is already on your PATH (e.g., you started R in a conda prompt),
# you can just use "NGSpeciesID".
# Otherwise, specify the full path, e.g.:
# ngspeciesid_exe <- "C:/Users/anjani/miniconda3/envs/NGSpeciesID/Scripts/NGSpeciesID.exe"
ngspeciesid_exe <- "NGSpeciesID"  # assume it's in PATH

###############################################################################
# 5) Run NGSpeciesID for each barcode
###############################################################################
for (bc in barcodes) {
  # Show which FASTQ files we are processing (mimics `echo $TARGET_DIR/$i/*`)
  message("Processing: ", file.path(TARGET_DIR, bc, "*"))
  
  # Construct the command, pointing to the filtered FASTQ
  fastq_pattern <- paste0('"', file.path(TARGET_DIR_WIN, bc, "*_filtered.fastq"), '"')
  outfolder     <- paste0('"', file.path(OUT_DIR_WIN, bc), '"')
  
  args_NGSpeciesID <- paste0(
    '"', ngspeciesid_exe, '" ',
    "--ont ",
    "--fastq ", fastq_pattern, " ",
    "--outfolder ", outfolder, " ",
    "--consensus ",
    "--m 655 ",
    "--s 100 ",
    "--racon ",
    "--racon_iter 3 ",
    "--abundance_ratio 0.05 "
    # add other flags as needed
  )

  # Run NGSpeciesID through cmd.exe
  system("cmd.exe", input = args_NGSpeciesID, invisible = FALSE)
}
