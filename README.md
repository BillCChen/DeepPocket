# DeepPocket: Ligand Binding Site Detection and Segmentation using 3D Convolutional Neural Networks

DeepPocket is a 3D convolutional Neural Network framework for ligand binding site detection and segmentation from protein structures. This is the official open source repository for the following paper:

Aggarwal, Rishal; Gupta, Akash; Chelur, Vineeth; Jawahar, C. V.; Priyakumar, U. Deva (2021): DeepPocket: Ligand Binding Site Detection and Segmentation using 3D Convolutional Neural Networks. ACS JCIM. 2021 [Link](https://pubs.acs.org/doi/10.1021/acs.jcim.1c00799) 

If you want to use this project for development, we recommend going through [libmolgrid](https://github.com/gnina/libmolgrid) first. To use DeepPocket for predicting binding sites on an input protein skip to "Predicting Binding Sites" section. 

## Requirements

[Fpocket](https://github.com/Discngine/fpocket), [Pytorch](https://pytorch.org/), [libmolgrid](https://github.com/gnina/libmolgrid), [Biopython](https://biopython.org/) and other frequently used python packages

To reproduce the substructure benchmark [Prody](https://prody.csb.pitt.edu/) and [Rdkit](https://www.rdkit.org/) are also required.

## Dataset Preprocessing

PDB files are first parsed to remove hetero atoms, then converted to "gninatypes" files and finally collected into a "molcache2" file for quicker input and model training with libmolgrid. "gninatypes" and "molcache2" files are binary files that store an efficient representation of the input protein to be used for gridding the molecule. They are prepared for faster input with libmolgrid for quicker training of the CNN models.

cavity6.mol2 files that are provided by scPDB and generated by volsite for other datasets are used as is, the "data_dir" argument in training scripts have to be pointed to the parent directory they are present.

".types" files contain training data points prepared, the first column is the class label, the next three columns are pocket center cordinates (x,y,z) and the final columns contain molecule files required for that datapoint. All the molecule files specified in the types files must be present in either the molcache or in the "data_dir". 

Prepared types, molcache and saved model checkpoints can be downloaded [here](https://iiitaphyd-my.sharepoint.com/:f:/g/personal/rishal_aggarwal_research_iiit_ac_in/EoJSrvuiKPlAluOJLjTzfpcBT2fVRdq8Sr4BMmil0_tvHw?e=Kj7reS](https://pitt-my.sharepoint.com/:u:/g/personal/ria43_pitt_edu/EUihPbBXe8VNtHhHNBuaNk8B3nNVtTZQ5c2ofxnmSoHLxw?e=9m9bxQ). SC6K can be downloaded from the same link (SC6K.tar.gz). COACH420 and HOLO4k are publicly available [here](https://github.com/rdk/p2rank-datasets)

Edit - I now provide the ligands used in the publication for Coach420 and Holo4k in the same onedrive link above. The number of ligands is not the same as the original manuscript possibly due to differences in whether a molecule file is percieved as valid by different versions of openbabel. For Coach420 I now provide 361 molecules as compared to 359 in the manuscript, and for Holo4k I provide 4309 as compared to 4288 mentioned earlier.   

I know this may be very cryptic, therefore I have written down simple steps in the last section of the README that one can use to prepare a new dataset for training. 

## Predicting Binding Site

"predict.py" is a simple script that can be used for predicting binding sites from a .pdb file. It follows 6 steps namely:
1) Hetero atom removal (clean_pdb)
2) fpocket run
3) Parsing fpocket output for candidate centers (get_centers)
4) Creating gninatypes and types file for CNN input (types_and_gninatyper)
5) Rerank types input according to CNN score (rank_pockets)
6) Segment shape of top ranked pockets (segment_pockets)

Example usage of predict.py:

    python predict.py -p protein.pdb -c first_model_fold1_best_test_auc_85001.pth.tar -s seg0_best_test_IOU_91.pth.tar -r 3

Description of each argument given in script.

If the name of the input file is protein.pdb, then fpocket creates a protein_out/pockets directory. The CNN ranked pockets will be given in the bary_centers_ranked.types file in that directory. The CNN confidence scores will be provided in the *_confidence.txt file

If you asked for segmented pockets ("-r") the script will output ".dx" files that can be visualised in pymol. It will also output "pocket*.pdb" files that contain predicted binding site residues. If no binding site residues are predicted that particular pocket*.pdb will not be created.

## Training Classifier

We use [wandb](https://wandb.ai/site) to track training performance. It's free and easy to use. If you want to avoid using wandb, simply comment out all lines that contain "wandb" in the training script.

Example usage of train.py:

    python train.py -m model.py --train_types scPDB_train0.types --test_types scPDB_test0.types -i 200000 --train_recmolcache scPDB_new.molcache2 --test_recmolcache scPDB_new.molcache2 -r val0 -o /model_saves/val9 --base_lr 0.001 --solver Adam 

Description of each argument given in script.

## Training segmentation

Example usage of train_segmentation.py:

    python train_segmentation.py --train_types seg_scPDB_train9.types --test_types seg_scPDB_test9.types -d data/ --train_recmolcache scPDB_new.molcache2 --test_recmolcache scPDB_new.molcache2 -b 8 -o model_saves/seg9 -e 200 -r seg9
    
Description of each argument in script

## Preparing Data

I have written down steps below that pertain to preparing training data for a dataset like PDBbind, but could easily be adopted for other datasets by making approprate changes of file paths and file names in the scripts.

Steps for preparing training data:

1) remove hetero atoms (clean_pdb.py)
2) run fpocket through structures (fpocket -f *_protein.pdb)
3) get candidate pocket centers for all structures (get_centers.py)
4) create .gninatypes files for all structure (gninatype() in types_and_gninatyper.py)
5) make train and test types (make_types.py)
6) create molcache file for training (create_molcache2.py)

Example usage of create_molcache2:

	python create_molcache2.py -c 4 --recmolcache scPDB_new.molcache2 -d data/scPDB/  scPDB_train0.types scPDB_test0.types

## Substructure Benchmark

To reproduce our results on the substructure benchmark, run the following command:
     
        python subpockets_benchmark_all.py --test_types refined4414_predict.types --model_weights refined_best_test_IOU_88.pth.tar -d ./data/ --test_recmolcache refined4414.molcache2

"refined4414_predict.types" file contains fpocket candidate centers closest to ligand for protein-ligand complexes in the refined dataset.
The data directory should contain clean (no water) .pdb files and ligand sdf files.

## Citation

If you find this useful please cite the paper mentioned above.

	@article{doi:10.1021/acs.jcim.1c00799,
    author = {Aggarwal, Rishal and Gupta, Akash and Chelur, Vineeth and Jawahar, C. V. and Priyakumar, U. Deva},
    title = {DeepPocket: Ligand Binding Site Detection and Segmentation using 3D Convolutional Neural Networks},
    journal = {Journal of Chemical Information and Modeling},
    volume = {0},
    number = {0},
    pages = {null},
    year = {0},
    doi = {10.1021/acs.jcim.1c00799},
    note ={PMID: 34374539},

    URL = { 
        https://doi.org/10.1021/acs.jcim.1c00799
    
    },
    eprint = { 
        https://doi.org/10.1021/acs.jcim.1c00799
    
    }

    }

