# Tutorial: Predicting Protein Complex Structures with AlphaPulldown and AlphaLink

This tutorial guides you through the process of predicting protein complex structures using **AlphaPulldown** and **AlphaLink**, tools that extend AlphaFold's capabilities for multimeric protein complexes and incorporate experimental cross-linking data, respectively. **AlphaPulldown** supports multiple modeling backends including **AlphaFold** and **AlphaLink**.

## Prerequisites

- Access to a Linux terminal with AlphaPulldown and AlphaLink installed.
- Basic familiarity with command-line operations.
- ChimeraX installed on your local machine for visualization.

---

## Overview

We will:

1. **Generate features** for individual protein chains using AlphaPulldown.
2. **Configure and run AlphaPulldown** to predict protein complexes.
3. **Prepare cross-linking data** and use AlphaLink for structure prediction.
4. **Run standard AlphaFold predictions** for comparison.
5. **Visualize and compare models** in ChimeraX.
6. **Utilize multimeric templates** to guide AlphaFold modeling.

---

## Step 1: Set Up the Working Environment

### Copy Tutorial Data to Your Home Directory

First, copy the tutorial data to your home directory:

```bash
cp -r /scratch/denbi/k8s/sunday_tutorial_copy/ /scratch/denbi/k8s/<your_directory>
cd /scratch/denbi/k8s/<your_directory>
```

---

## Step 2: Generate Features for Individual Chains

To predict protein complexes, we need to generate features for each individual protein chain.

### Proteins Used in This Tutorial

We will use three subunits of the **DNA-directed RNA polymerase III**:

- **P32349**: [UniProt Entry](https://www.uniprot.org/uniprotkb/P32349/entry)
- **P32910**: [UniProt Entry](https://www.uniprot.org/uniprotkb/P32910/entry)
- **P17890**: [UniProt Entry](https://www.uniprot.org/uniprotkb/P17890/entry)

### Create a FASTA File for P17890

1. **Copy the sequence** from the [UniProt entry for P17890](https://www.uniprot.org/uniprotkb/P17890/entry).

2. **Create a FASTA file** named `P17890.fasta` using a text editor (e.g., `gedit`, `vim`):

   ```bash
   vim P17890.fasta
   ```

3. **Paste the following content** into the file:

   ```text
   >P17890
   MSSYRGGSRGGGSNYMSNLPFGLGYGDVGKNHITEFPSIPLPINGPITNKERSLAVKYIN
   FGKTVKDGPFYTGSMSLIIDQQENSKSGKRKPNIILDEDDTNDGIERYSDKYLKKRKIGI
   SIDDHPYNLNLFPNELYNVMGINKKKLLAISKFNNADDVFTGTGLQDENIGLSMLAKLKE
   LAEDVDDASTGDGAAKGSKTGEGEDDDLADDDFEEDEDEEDDDDYNAEKYFNNGDDDDYG
   DEEDPNEEAAF
   ```

4. **Repeat for P32349.fasta and P32910.fasta**, below their sequences for convenience:
   ```text
   >P32349
   MDELLGEALSAENQTGESTVESEKLVTPEDVMTISSLEQRTLNPDLFLYKELVKAHLGE
   RAASVIGMLVALGRLSVRELVEKIDGMDVDSVKTTLVSLTQLRCVKYLQETAISGKKTTY
   YYYNEEGIHILLYSGLIIDEIITQMRVNDEEEHKQLVAEIVQNVISLGSLTVEDYLSSVT
   SDSMKYTISSLFVQLCEMGYLIQISKLHYTPIEDLWQFLYEKHYKNIPRNSPLSDLKKRS
   QAKMNAKTDFAKIINKPNELSQILTVDPKTSLRIVKPTVSLTINLDRFMKGRRSKQLINL
   AKTRVGSVTAQVYKIALRLTEQKSPKIRDPLTQTGLLQDLEEAKSFQDEAELVEEKTPGL
   TFNAIDLARHLPAELDLRGSLLSRKPSDNKKRSGSNAAASLPSKKLKTEDGFVIPALPAA
   VSKSLQESGDTQEEDEEEEDLDADTEDPHSASLINSHLKILASSNFPFLNETKPGVYYVP
   YSKLMPVLKSSVYEYVIASTLGPSAMRLSRCIRDNKLVSEKIINSTALMKEKDIRSTLAS
   LIRYNSVEIQEVPRTADRSASRAVFLFRCKETHSYNFMRQNLEWNMANLLFKKEKLKQEN
   STLLKKANRDDVKGRENELLLPSELNQLKMVNERELNVFARLSRLLSLWEVFQMA

   ```
   ```text
   >P32910
   MSGMIENGLQLSDNAKTLHSQMMSKGIGALFTQQELQKQMGIGSLTDLMSIVQELLDKNL
   IKLVKQNDELKFQGVLESEAQKKATMSAEEALVYSYIEASGREGIWSKTIKARTNLHQHV
   VLKCLKSLESQRYVKSVKSVKFPTRKIYMLYSLQPSVDITGGPWFTDGELDIEFINSLLT
   IVWRFISENTFPNGFKNFENGPKKNVFYAPNVKNYSTTQEILEFITAAQVANVELTPSNI
   RSLCEVLVYDDKLEKVTHDCYRVTLESILQMNQGEGEPEAGNKALEDEEEFSIFNYFKMF
   PASKHDKEVVYFDEWTI

   ```

### Generate Features Using AlphaPulldown

Run the following command to generate features for P17890:

```bash
create_individual_features.py \
  --fasta_paths=P17890.fasta \
  --data_dir=/scratch/AlphaFold_DBs/2.3.2/ \
  --output_dir=features_mmseqs2/ \
  --max_template_date=2050-01-01 \
  --use_mmseqs2
```
Repeat for P32349 and P32910
**Explanation of arguments**:

- `--fasta_paths`: Path to the input FASTA file(s).
- `--data_dir`: Directory containing AlphaFold databases.
- `--output_dir`: Directory to store the generated features.
- `--max_template_date`: Cutoff date for template structures (set far in the future to include all templates).
- `--use_mmseqs2`: Use MMseqs2 for fast multiple sequence alignment.

**Note**: Since the sequence is short and we're using MMseqs2, this should complete in about **1 minute**.

### Verify the Generated Features

List the contents of the features directory:

```bash
ls features_mmseqs2/
```

You should see:

```bash
P17890.a3m
P17890_env
P17890_feature_metadata_<date>.json
P17890.pkl
...
```

---

## Step 3: Configure AlphaPulldown for Complex Prediction

We will now set up AlphaPulldown to predict interactions between the protein subunits.

### Create a Protein Pair List

Create a file named `custom.txt` containing the protein pairs to model in gedit or in vim:

```bash
vim custom.txt
```

Add the following lines:

```text
P32349;P32910
P32349;P17890
P32910;P17890
```

**Explanation**:

- Each line represents a protein pair to model, with protein IDs separated by a `;`.

---

## Step 4: Run AlphaPulldown

Run AlphaPulldown to predict the structures of the protein complexes:

```bash
run_multimer_jobs.py \
  --mode=custom \
  --num_predictions_per_model=1 \
  --model_names=model_2_multimer_v3 \
  --output_path=predictions/ \
  --data_dir=/scratch/AlphaFold_DBs/2.3.2/ \
  --protein_lists=custom.txt \
  --monomer_objects_dir=features_mmseqs2/
```

**Explanation of arguments**:

- `--mode=custom`: Use a custom list of protein pairs.
- `--num_predictions_per_model=1`: Generate one prediction per model.
- `--model_names=model_2_multimer_v3`: Specify the AlphaFold multimer model version.
- `--output_path=predictions/`: Directory to store predictions.
- `--data_dir`: AlphaFold databases directory.
- `--protein_lists=custom.txt`: File containing protein pairs.
- `--monomer_objects_dir`: Directory with pre-calculated features for individual proteins.

**Note**: Each protein pair should take **1-2 minutes** to process.

### Verify the Predictions

List the contents of the predictions directory:

```bash
ls predictions/
```

You should see:

```bash
P32349_and_P17890
P32349_and_P32910
P32910_and_P17890
```

Check the contents of one of the prediction directories:

```bash
ls predictions/P32349_and_P17890/
```

Expected output:

```bash
confidence_model_2_multimer_v3_pred_0.json
pae_plot_ranked_0.png
ranked_0.pdb
ranking_debug.json
timings.json
pae_model_2_multimer_v3_pred_0.json
ranked_0.cif
ranked_0.zip
result_model_2_multimer_v3_pred_0.pkl
unrelaxed_model_2_multimer_v3_pred_0.pdb
```

---

## Step 5: Analyze the Results

### Check Model Scores

The `ranking_debug.json` file contains scores that help assess the quality of the models.

To extract the scores:

```bash
grep -A 1 'iptm+ptm' predictions/*/ranking_debug.json
```

**Explanation**:

- `grep -A 1 'iptm+ptm'`: Search for the line containing 'iptm+ptm' and display that line along with the following line (`-A 1`).
- This will show the interface predicted TM-score plus the predicted TM-score, which are indicators of model confidence.

### Visualize the Models in ChimeraX

To visualize the predicted complex:

1. **Download the model** `ranked_0.pdb` from `predictions/P32349_and_P17890/` to your local machine.

2. **Open ChimeraX**.

3. **Load the model**:
   Either open via dialog `Open -> File...` or using ChimeraX command line:

   ```bash
   open /path/to/output/directory/ranked_0.pdb
   color bfactor palette alphafold
   alphafold pae #1 palette bluered file predictions/P32349_and_P17890/pae_model_2_multimer_v3_pred_0.json
   ```
4. **Analyse PAE plot**:
   Do two chains interact according to the PAE plot? How to find the interface?
---

## Step 6: Running AlphaLink with Cross-Linking Data

We will now use **AlphaLink** to incorporate experimental cross-linking data into the protein complex prediction.

**Open the AlphaLink terminal**

### Prepare Input Data for AlphaLink

1. **Change directory to the AlphaLink input data**:

   ```bash
   cd alphalink/
   ls
   ```

   You should see:

   ```bash
   create_features.sh  csv2pb.py  custom.txt          features_mmseqs2  H1142.csv  H1142.fasta           predict.sh  results
   ```

2. **Examine the cross-linking data**:
Open `H1142.csv` in gedit or vim. The `H1142.csv` file contains cross-linking data between proteins A and B.

### Generate Cross-Link File Compatible with AlphaLink

Convert the CSV file to a pickle file:

```bash
generate_crosslink_pickle.py --csv H1142.csv --output crosslinks.pkl.gz
```

**Explanation**:

- This script converts cross-linking data into a format that AlphaLink can use.

### Run AlphaLink Prediction

Run the structure prediction with AlphaLink:

```bash
run_structure_prediction.py \
--output_directory predictions_new/ \
--num_cycle 3 \
--num_predictions_per_model 1 \
--data_directory /scratch/AlphaFold_DBs/2.3.2/alphalink_weights/AlphaLink-Multimer_SDA_v3.pt \
--features_directory features_mmseqs2/ \
--crosslinks crosslinks.pkl.gz \
--fold_backend alphalink \
--use_ap_style \
--use_gpu_relax \
--input A+B
```

**Alternative**:

- You can run the prediction using the provided script:

  ```bash
  ./predict.sh
  ```

### Convert Cross-Links to Pseudobonds for Visualization

Generate a pseudobonds file for ChimeraX:

```bash
python csv2pb.py H1142.csv H1142.pb
```

---

## Step 7: Visualize the AlphaLink Model in ChimeraX
1. **Open ChimeraX**.
Open in ChimeraX:
- `predictions/A+B/ranked_0.pdb` (AlphaLink model)
- `H1142.pb` (pseudobonds file)

> [!NOTE]
> In case of troubles with remote ChimeraX, you can download the files using web interface at [https://bard-external.embl.de/](https://bard-external.embl.de/)

2. **Assess Cross-Link Satisfaction**:

   - The pseudobonds represent cross-links.
   - Check if the distances between linked residues satisfy the cross-linking constraints.

---

## Step 8: Running Standard AlphaFold for Comparison

We will now run a standard AlphaFold prediction without cross-linking data to compare with the AlphaLink model.

### Switch to AlphaFold Terminal

1. *Close the current terminal*.

2. **Open a new terminal** configured for AlphaFold:

   - Click the appropriate icon in the taskbar.

### Run AlphaFold Prediction

1. **Navigate to the input directory**:

   ```bash
   cd alphalink/
   ```

2. **Run the prediction**:

   ```bash
   run_structure_prediction.py \
     --input A+B \
     --output_directory predictions_af/ \
     --features_directory features_mmseqs2/ \
     --data_directory /scratch/AlphaFold_DBs/2.3.2/
   ```

### Visualize the AlphaFold Model

1. Open **new** ChimeraX window.

3. Load the AlphaFold model from `predictions_af/A+B/ranked_0.pdb`.

4. Load the pseudobonds file.

---

### Analyze Cross-Link Satisfaction

1. **Check cross-link distances**:

   - Hover mouse to check individual distances


### Assess the Impact of Cross-Linking Data

- **AlphaLink Model**:

  - Should show better satisfaction of cross-linking restraints.

- **AlphaFold Model**:

  - May have more violations due to the absence of cross-linking data.

---

## Step 10: Summary

In this tutorial, we've:

- **Generated features** for individual protein chains.
- **Predicted protein complexes** using AlphaPulldown.
- **Incorporated cross-linking data** using AlphaLink.
- **Ran standard AlphaFold predictions** for comparison.
- **Visualized and compared models** in ChimeraX.


# Modeling using multimeric templates

AlphaFold2 typically uses only monomeric templates, disregarding chain orientations in template files. However, multimeric templates can be useful in cases such as:

- refining a rough integrative model of your complex from another program,
- extending a cryo-EM subcomplex structure with additional subunits or domains, or
- enforcing a specific conformation for modeling.

In the example below, we:

1. Build an initial model using AlphaFold3.  
1. Fit the model to an EM map, identify misfit, and manually adjust it, improving the fit but introducing steric clashes.  
1. Use AlphaFold2 with a multimeric template to resolve clashes, aiming to produce a model that fits the map, resolves clashes, and remodels loops.

> [!NOTE]
> This tutorial teaches a "quick mode" of the multimeric mode, which is simpler to run but only works for cases where the template is very similar in sequence to the complex being modeled. For more complex cases, you would need to run the full multimeric mode as described in the manual: https://github.com/KosinskiLab/AlphaPulldown?tab=readme-ov-file#14-run-with-custom-templates-truemultimer. That mode also contains additional features like automated removal of low pLDDT regions and clashing regions in the input template prior to modeling.

## Open the EM map and initial models in ChimeraX

1. Open ChimeraX
1. Load the ChimeraX session file that we prepared for you:

   ```text
   /scratch/denbi/k8s/sunday_tutorial/multimeric_templates/multimeric_templates.cxs
   ```

The session contains the following models:

1. The map of the Pol 3 complex in the elongating conformation.  
1. The same map, filtered with Gaussian and with transparency adjusted to make it easier to visualize the models and their fits.
1. The AlphaFold3 model of the Pol 3 subcomplex.
1. The AlphaFold3 model of the Pol 3 subcomplex, manually adjusted to fit the map. To generate this model, we selected parts of the structure and fitted them individually to the map. We also removed some of the regions extending outside the density, hoping AlphaFold2 would remodel them using the fitted template. This is just an example—a model like this could also come from automated flexible fitting or integrative modeling.

Which model fits better? Which parts we have adjusted?

## Calculate clashes

1. Select the initial model with the following command:
   
   ```bash
   select #3
   ```
1. In ChimeraX, click Menu -> Tools -> Structure Analysis -> Clashes
1. In "Interaction parameters:
   1. Select "Limit by selection" and "with bond ends selected"
   1. Select all "Include" checkboxes
1. In "Treatment of results", select "Select atoms"
1. Click "Apply"
1. How many clashes do you see? (Check the number in the log panel)
1. Now, select the manually adjusted model (#4) and measure clashes again. How many clashes do you see now?

> [!NOTE]
>  some clashes were present already in the initial model - this is correct, AlphaFold3 models often have some clashes!

## Run AlphaFold2 with the multimeric template using AlphaPulldown

1. Create and enter directory for this project in your directory:

   ```bash
   cd /scratch/denbi/k8s/<your directory>
   mkdir multimeric_templates
   cd multimeric_templates
   ```

1. In this directory, create a file named `Pol3.txt` containing the list of protein names to model (here, Uniprot IDs):

   ```text
   P32349;P32910;P17890;P47076;P35718;P04051;P22276;P20434
   ```

1. Create a text file named `description.csv` that maps the proteins to chains in the template:

   ```text
   P32349,AlphaFold3_optimized.cif,A
   P32910,AlphaFold3_optimized.cif,B
   P17890,AlphaFold3_optimized.cif,C
   P47076,AlphaFold3_optimized.cif,D
   P35718,AlphaFold3_optimized.cif,E
   P04051,AlphaFold3_optimized.cif,F
   P22276,AlphaFold3_optimized.cif,G
   P20434,AlphaFold3_optimized.cif,H
   ```

1. As the next step, normally, you would calculate MSA and template features using the `create_individual_features.py` script. However, this would take too long for this tutorial and we have already calculated the features for the proteins in the following directory.
   
   ```bash
   /scratch/denbi/k8s/sunday_tutorial/multimeric_templates/Pol3_features
   ```

1. Run the modeling:

   ```bash
   run_multimer_jobs.py \
   --mode=custom \
   --protein_lists=Pol3.txt \
   --monomer_objects_dir=/scratch/denbi/k8s/sunday_tutorial/multimeric_templates/Pol3_features \
   --output_path=./ \
   --num_cycle=1 \
   --data_dir=/scratch/AlphaFold_DBs/2.3.2/ \
   --num_predictions_per_model=1 \
   --model_names=model_2_multimer_v3 \
   --multimeric_template \
   --description_file=description.csv \
   --path_to_mmt=./
   ```

   This will take around 20 minutes on Nvidia H100 GPU card. Normally, you would run this on a cluster and with at least --num_cycle=3 and without --model_names flag.
   
   Explanations:
   ```bash
   run_multimer_jobs.py \
   --mode=custom \ # Use the custom model of AlphaPulldown
   --protein_lists=Pol3.txt \ # Specification of the protein complex
   --monomer_objects_dir=/scratch/denbi/k8s/sunday_tutorial_copy/multimeric_templates/Pol3_features \ # Directory with pre-calculated features for individual proteins
   --output_path=./ \ # Output directory, here "." means the current directory
   --num_cycle=1 \ # Number of cycles, normally at least 3 (25 in AlphaFold2)
   --data_dir=/scratch/AlphaFold_DBs/2.3.2/ \ # Directory with AlphaFold databases
   --num_predictions_per_model=1 \ # Number of predictions per AlphaFold2 model type
   --model_names=model_2_multimer_v3 \ # AlphaFold2 model type
   --multimeric_template \ # Turns on the use of multimeric templates
   --description_file=description.csv \ # File with the mapping of proteins to chains in the multimeric template
   --path_to_mmt=./ # Path to the directory with the multimeric template, here "." means the current directory
   ```

> [!NOTE]
>  AlphaFold2 would sometimes not listen the template, especially for smaller complexes and for cases with large Multiple Sequence Alignmnets (MSAs). Thus, normally you would first run the command above (```--num_cycle=1``` and ```--num_predictions_per_model=1```) with an additional ```--msa_depth_scan``` flag to try different MSA depths and then run the command without this flag with the best MSA depth using ```--msa_depth=<value>``` flag and with more recycles or num_predictions_per_model.

## Compare the models

Open the models in ChimeraX and compare them to the template.cif structure. Calculate clashes and see how the new model resolves them.

Hint: You can use the
```bash
mm <model id> to <model id>
```
command to superimpose the models. Are the clashes resolved? How does the new model fit the map?

> [!NOTE]
> This model has pLDDT scores in the B-factor column as the usual AlphaFold models. However, when the multimeric template is identical in sequence to the complex being modeled, like in our case, the pLDDT scores in the regions aligning to the template are often overestimated and should not be used for model evaluation. The model should be evaluated using the map, the clashes, and other information and used more as a starting point for further modeling and validation.
---

# Additional Resources

- **AlphaFold GitHub Repository**: [https://github.com/deepmind/alphafold](https://github.com/deepmind/alphafold)
- **AlphaPulldown Documentation**: [AlphaPulldown Documentation](https://github.com/KosinskiLab/AlphaPulldown?tab=readme-ov-file#alphapulldown-version-200)
- **AlphaLink Publication**: [https://www.nature.com/articles/s41467-024-51771-2](https://www.nature.com/articles/s41467-024-51771-2)
- **ChimeraX Tutorials**: [https://www.cgl.ucsf.edu/chimerax/tutorials.html](https://www.cgl.ucsf.edu/chimerax/tutorials.html)

---
