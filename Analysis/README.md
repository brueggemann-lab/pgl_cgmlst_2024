# Analysis

## BUSCO completeness
```
# Calculate completeness stats for each assembly
busco -f -m genome -o assembly.busco.out -i assembly.fa -c 1 -l busco_downloads/lineages/lactobacillales_odb10/
```
## Profile clustering
```
# Run MSTclust on sets of profiles, varying -e depending on dataset
java -jar MSTclust.jar -i  5050profiles.txt -o mst_cluster.5050.input.out.d -l 1 -p 2-1223 -r 2- -e 0.90 -L 1222

# Convert output to a matrix with csv format
cut -f1 -d ' ' mst_cluster.5050.input.out.d | datamash transpose | sed $'s/^/\t/g' | cat -mst_cluster.5050.input.out.d| tr -s ' ' | tr '\t' ',' | sed 's/ /,/g' | sed '1 s/^/ID/' > mstclust.5050.d.csv
```
## Maximum likelihood phylogenies
```
# Using nucleotide sequences for each sample:allele combination downloaded from PubMLST, convert from unaligned XMFA to FASTA
grep -v "+" export_PubMLST.fasta

# Split into fasta file for each loci 
parallel -j8 "fastaqual_select.pl -f export_PubMLST.fasta -regexp {} > loci/{}.fasta" :::: spne.list

# Subset per-loci fasta files to only contain the relevant isolates (defined in the shortlist.txt file) for each loci (defined in spne.list), then pipeline to MAFFT to align
parallel -j8 "cat loci/{}.fasta | fastaqual_select.pl -f - -i shortlist.txt | mafft --thread 1 --auto - > alignments/{}.aln.out" :::: spne.list

# Convert to XMFA format for ClonalFrameML, with a custom spacer file
echo "=" > space.txt
ls | grep re | sort | sed 's/^/cat /g' | sed 's/$/ space.txt/g' | tr '\n' ' ' > test.sh
./test.sh > out.reformatted.d.xmfa

# Merge invidual FASTA files into supermatrix
python concat_aln.py -a alignments/*.aln.out -u out.18k.concat -d ":"

# Run FastTree
FastTreeMP -gtr -nt -log logfile < out.18k.concat > out.18k.tree

# Run ClonalFrameML
ClonalFrameML out.18k.tree out.reformatted.d.xmfa cframe_18k_d -xmfa_file true -show_progress true -output_filtered true

```
## Mandrake clustering
```
# Repeat allele concatenation and alignment as above on 5000 samples
parallel -j8 "fastaqual_select.pl -f export_PubMLST.fasta -regexp {} > loci/{}.fasta" :::: spne.list
parallel -j8 "cat loci/{}.fasta | fastaqual_select.pl -f - -i 5k_list.txt | mafft --thread 1 --auto - > alignments/{}.aln.out" :::: spne.list
python concat_aln.py -a alignments/*.aln.out -u out.5k.concat -d ":"

# Run Mandrake
mandrake --alignment out.5k.concat --output def_knn5k
```
# GPSC clustering 
```
poppunk_assign --db GPS_v8_ref --external-clustering GPS_v8_external_clusters.csv --query fastalist.ls --output GPSC_result_v8 --threads 32
```
