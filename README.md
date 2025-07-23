# AmrProfiler

**AmrProfiler** is a bioinformatics tool designed to identify antimicrobial resistance (AMR) genes, detect point mutations in core genes, and analyze mutations in rRNA genes across 18,000 bacterial species.
AmrProfiler webserver is available at https://dianalab.e-ce.uth.gr/amrprofiler


![Screenshot from 2025-02-25 15-16-37](https://github.com/user-attachments/assets/a4cbfbd8-92fb-4057-896d-34e34dd22c7e)

AmrProfiler relies on the RefSeq database (downloaded on 01/06/2025). It is implemented in Python 3.9.5 and uses the `NcbiblastxCommandline` utility from the Biopython library (v1.79), along with the BLAST+ suite (v2.9.0).


## Installation & Setup

To run AmrProfiler locally, you must first download the required databases and files from Zenodo (https://zenodo.org/records/15674467). Extract the downloaded folders inside the `amrprofiler` directory from this repo.

After this, install the dependencies using `conda` and activate the newly created environment

```bash
conda env create -f ./amrprofiler-main/environment.yml
conda activate amrprofiler
```

This is highly recommended as it will guarantee that `python v3.9.5` is installed properly.

## Running AmrProfiler

### Default Execution

To run AmrProfiler with default parameters, use **Python v3.9.5** and execute the following command. 
Newer versions require modifications to handle _str.replace_ with the letters "lcl|" differently. 
To run amrprofiler.py, you need the name of the strain, the selected species and also the folder saved the amrprofiler databases and scripts. The results will be saved in the running folder. 

```sh
python /home/argis/Desktop/pasteur/amr_server/amrprofiler-main/amrprofiler.py "Lar4933_contigs.fasta" "Staphylococcus aureus" /home/argis/Desktop/pasteur/amr_server/amrprofiler-main/
```

### Run each AmrProfiler's modules separately in python

### Tool 1: AMR Gene Identification

This tool identifies antimicrobial resistance genes from a given query sequence.

```python
import sys
import importlib.util

# Load module
module_path = '/home/argis/Desktop/pasteur/amr_server/amrprofiler-main/find_amr_genes_module.py'
spec = importlib.util.spec_from_file_location("find_amr_genes_module", module_path)
find_amr_genes_module = importlib.util.module_from_spec(spec)
sys.modules["find_amr_genes_module"] = find_amr_genes_module
spec.loader.exec_module(find_amr_genes_module)

# Parameters
query_file = "Lar4933_contigs.fasta"
Database_of_Genes = "Resfinder+ReferenceGeneCatalog"
db_path = "/home/argis/Desktop/pasteur/amr_server/amrprofiler-main/databases/all/amrFinder_ResFinder" if Database_of_Genes == "Resfinder+ReferenceGeneCatalog" else "/home/argis/Desktop/pasteur/amr_server/amrprofiler-main/databases/all/all_amr"
num_threads = 4
protein_annotation_file = "/home/argis/Desktop/pasteur/amr_server/amrprofiler-main/databases/genes_annotation_databases.csv"
protein_start_filter = 50
identity_threshold = 70
coverage_threshold = 70

# Run analysis
blast_results_df = find_amr_genes_module.run_blastx_to_dataframe(query_file, db_path, num_threads)
final_results = find_amr_genes_module.process_blast_results(protein_annotation_file, blast_results_df, query_file, protein_start_filter, identity_threshold, coverage_threshold)

# Save results
blast_results_df.to_csv("blast_results.csv", sep="\t", index=False, float_format="%.6f")
final_results.to_csv("final_results_tool1.csv", sep="\t", index=False, float_format="%.6f")
```

### Tool 2: Core Gene Mutation Analysis

This tool detects point mutations in core genes for a given bacterial species.

```python
# Load module
module_path = '/home/argis/Desktop/pasteur/amr_server/amrprofiler-main/core_genes_finder.py'
spec = importlib.util.spec_from_file_location("core_genes_finder", module_path)
core_genes_finder = importlib.util.module_from_spec(spec)
sys.modules["core_genes_finder"] = core_genes_finder
spec.loader.exec_module(core_genes_finder)

# Parameters
query_file = "Lar4933_contigs.fasta"
species = "Staphylococcus aureus"
db_path_folder = "/home/argis/Desktop/pasteur/amr_server/amrprofiler-main/refseq/"
threads = 4
difference_number = 2
difference_from_start = 20
identity_threshold = 90
coverage_threshold = 90

# Run analysis
df4 = core_genes_finder.get_species_row(species, db_path_folder)
df4.to_csv("core_genes_of_this_species.csv", index=False, float_format="%.6f")
blast_results_core = core_genes_finder.run_blastx_for_references(query_file, df4, db_path_folder)
blast_results_core.to_csv("blast_results_core.csv", sep="\t", index=False, float_format="%.6f")

result_df = core_genes_finder.process_blast_results(species, blast_results_core, query_file, df4, difference_number, db_path_folder, difference_from_start, identity_threshold, coverage_threshold)
result_df.to_csv("mutations_results.csv", sep="\t", index=False, float_format="%.6f")
```

### Tool 3: rRNA Mutation Analysis

This tool identifies mutations in rRNA genes for a given bacterial species.

```python
# Load module
module_path = '/home/argis/Desktop/pasteur/amr_server/amrprofiler-main/rRNA_genes_finder_NEW.py'
spec = importlib.util.spec_from_file_location("rRNA_genes_finder_NEW", module_path)
rRNA_genes_finder_NEW = importlib.util.module_from_spec(spec)
sys.modules["rRNA_genes_finder_NEW"] = rRNA_genes_finder_NEW
spec.loader.exec_module(rRNA_genes_finder_NEW)

# Parameters
query_file = "Lar4933_contigs.fasta"
species = "Staphylococcus aureus"
db_path_folder = "/home/argis/Desktop/pasteur/amr_server/amrprofiler-main/db/"
difference_number_rRNA = 5

# Run analysis
df5 = rRNA_genes_finder_NEW.get_species_row(species, db_path_folder)
df5.to_csv("rRNA_genes_of_this_species.csv", sep="\t", index=False, float_format="%.6f")
blast_results_rRNA = rRNA_genes_finder_NEW.run_blastn_for_references(query_file, df5, db_path_folder)
rRNA_dif = rRNA_genes_finder_NEW.find_rRNA_differences(species, df5, blast_results_rRNA, query_file, db_path_folder, difference_number_rRNA, 5)
rRNA_dif.to_csv("mutations_rRNA_results.csv", sep="\t", index=False, float_format="%.6f")
```

## 📖 Citations

If you have used **ResFinder + Reference Gene Catalog** as reference databases, please cite:

- **AMRProfiler**:
  - Skoulakis, A., Kostoula, V., Daniilidis, K., Gkountinoudis, C.-G., Petinaki, E., & Hatzigeorgiou, A. G. (2025).
    **AmrProfiler: a comprehensive tool for identifying antimicrobial resistance genes and mutations across species.** *Nucleic Acids Res., gkaf400. doi: 10.1093/nar/gkaf400*
- **ResFinder**:  
  Zankari E, Hasman H, Cosentino S, et al. **Identification of acquired antimicrobial resistance genes.** J Antimicrob Chemother. 2012;67(11):2640-2644. doi:10.1093/jac/dks261
- **AMRFinderPlus**:  
  - Feldgarden M, Brover V, Gonzalez-Escalona N, Frye JG, Haendiges J, Haft DH, Hoffmann M, Pettengill JB, Prasad AB, Tillman GE, Tyson GH, Klimke W. (2019).  
    **AMRFinderPlus and the Reference Gene Catalog facilitate examination of the genomic links among antimicrobial resistance, stress response, and virulence.** *Antimicrobial Agents and Chemotherapy, 63(7):e00483-19.*  
- **BLAST+**:  
  - Camacho C, Coulouris G, Avagyan V, Ma N, Papadopoulos J, Bealer K, Madden TL. (2009).  
    **BLAST+: Architecture and Applications.** *BMC Bioinformatics, 10(1):421.*  

---

If you have used **ResFinder + Reference Gene Catalog + CARD** or **Tool 2 and/or Tool 3**, please cite:

- **AMRProfiler**: 
  - Skoulakis, A., Kostoula, V., Daniilidis, K., Gkountinoudis, C.-G., Petinaki, E., & Hatzigeorgiou, A. G. (2025).
    **AmrProfiler: a comprehensive tool for identifying antimicrobial resistance genes and mutations across species.** *Nucleic Acids Res., gkaf400. doi: 10.1093/nar/gkaf400*
- **CARD**:  
  - Alcock BP, Raphenya AR, Lau TTY, Tsang KK, Bouchard M, Edalatmand A, Huynh W, Nguyen AL, Cheng AA, Liu S, Min SY, Miroshnichenko A, Tran HK, Werfalli RE, Nasir JA, Oloni M, Speicher DJ, Florescu A, Singh B, Faltyn M, Hernandez KA, Sharma AN, Bordeleau E, Pawlowski AC, Zubyk HL, Dooley D, Griffiths E, Maguire F, Winsor GL, Beiko RG, Brinkman FSL, Hsiao WWL, Domselaar GV, McArthur AG. (2023).  
    **CARD 2023: Expanded Curation, Support for Machine Learning, and Resistome Prediction at the Comprehensive Antibiotic Resistance Database.** *Nucleic Acids Research, 51, D690-D699.*  
- **ResFinder**:  
  - Zankari E, Hasman H, Cosentino S, et al.
    **Identification of acquired antimicrobial resistance genes.** J Antimicrob Chemother. 2012;67(11):2640-2644. doi:10.1093/jac/dks261
- **AMRFinderPlus**:  
  - Feldgarden M, Brover V, Gonzalez-Escalona N, Frye JG, Haendiges J, Haft DH, Hoffmann M, Pettengill JB, Prasad AB, Tillman GE, Tyson GH, Klimke W. (2019).  
    **AMRFinderPlus and the Reference Gene Catalog facilitate examination of the genomic links among antimicrobial resistance, stress response, and virulence.** *Antimicrobial Agents and Chemotherapy, 63(7):e00483-19.*  
- **BLAST+**:  
  - Camacho C, Coulouris G, Avagyan V, Ma N, Papadopoulos J, Bealer K, Madden TL. (2009).  
    **BLAST+: Architecture and Applications.** *BMC Bioinformatics, 10(1):421.*  


## Dependencies

- Python 3.9.5
- pandas 2.2+
- BLAST+ : 2.9.0+
- Biopython : 1.84+
- All the installed packages are available in the installed_packages.txt

  
## License and Third-Party Database Usage

This software is licensed under the **MIT License**. You are free to use, modify, and distribute this software under the terms of the license. See the [LICENSE](Licence) file for full details.

### Third-Party Databases Notice
This software integrates with external antimicrobial resistance (AMR) databases, including but not limited to:

- **CARD (Comprehensive Antibiotic Resistance Database)** – Available for **non-commercial academic research** under its terms of use ([CARD Terms](https://card.mcmaster.ca/about)). Users are responsible for ensuring compliance with these terms.
- **Reference Gene Catalog (NCBI)** – **Public domain**, freely available for unrestricted use.
- **ResFinder & PointFinder** – Licensed under the **Apache License 2.0**, allowing free use, modification, and distribution.

Users must **obtain appropriate permissions** if redistributing or using **CARD-derived data** in **commercial settings**. The authors of this software **do not provide any rights or licenses** for third-party databases.

For more details, refer to the [LICENSE](Licence) file.

