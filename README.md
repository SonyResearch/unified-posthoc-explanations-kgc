# Unifying Post-hoc Explanations of Knowledge Graph Completions

This repository contains the code supporting the homonymous paper

**Unifying Post-hoc Explanations of Knowledge Graph Completions**<br/>
Lonardi A., Badreddine S., Besold T. R., Sánchez-Martín P.

We evaluate four explainability algorithms extracting necessary post-hoc explanations for Knowledge Graph Completion (KGC):
- [Kelpie](https://dl.acm.org/doi/10.1145/3514221.3517887)
- [CRIAGE](https://aclanthology.org/N19-1337/)
- [Data Poisoning](https://www.ijcai.org/proceedings/2019/674)
- [AnyBURL Explainer](https://www.ijcai.org/proceedings/2022/391)

The first three algorithms are implemented in the [Kelpie repository](https://github.com/AndRossi/Kelpie). The AnyBURL Explainer is available downloading AnyBURL v22 from the [AnyBURL repository](https://web.informatik.uni-mannheim.de/AnyBURL/). KGC is performed by [ComplEx](https://proceedings.mlr.press/v48/trouillon16.html) on [FB15k-237](https://aclanthology.org/W15-4007/), using the code implemented in Kelpie.

Below, we provide instructions to replicate all paper results.

## Table of Contents
- [Position: Post-hoc Explanations of Knowledge Graph Completions Urge Unification and Reevaluation](#position-post-hoc-explanations-of-knowledge-graph-completions-urge-unification-and-reevaluation)
  - [Table of Contents](#table-of-contents)
  - [Getting Started](#getting-started)
  - [Training ComplEx](#training-complex)
  - [Extracting Necessary Explanations](#extracting-necessary-explanations)
    - [Sampling the Triples to Explain](#sampling-the-triples-to-explain)
    - [Running Kelpie, CRIAGE, Data Poisoning](#running-kelpie-criage-data-poisoning)
    - [Running the AnyBURL Explainer](#running-the-anyburl-explainer)
  - [Reproducing Figures and Data](#reproducing-figures-and-data)
  - [License](#license)
  - [Contact](#contact)


## Getting Started

To install the necessary dependencies, follow these steps.

1. Install [Poetry](https://python-poetry.org/) in your system.
2. Clone this repository to your local machine.
3. Install the dependencies with Poetry using
```bash 
poetry install --no-root
```

## Training ComplEx

To train ComplEx, it is possible to use the instruction detailed in Kelpie. Particularly,

1. Clone the Kelpie repository with
```bash 
git clone https://github.com/AndRossi/Kelpie
```
2. Train ComplEx using
```bash 
python scripts/complex/train.py --dataset FB15k-237 --optimizer Adagrad --dimension 1000 --batch_size 1000 --max_epochs 100 --learning_rate 0.1 --reg 5e-2
```

<ins>Note:</ins> As discussed in Appendix D of the paper, the hyperparameters suggested in Kelpie (in the command above) lead to an increase of the validation loss during training. We address this by proposing a new set of hyperparameters. To train the model with them, substitute the command in step 2 with the one below.

2. Train ComplEx using
```
python scripts/complex/train.py --dataset FB15k-237 --optimizer Adagrad --dimension 256 --batch_size 2048 --max_epochs 25 --learning_rate 0.01 --reg 0
```

<ins>Note:</ins> In these instructions, we always refer to the new set of hyperparameters. All commands can also be executed with the original hyperparameters suggested in Kelpie by substituting them into each command.

## Extracting Necessary Explanations

Upon training, it is possible to extract necessary explanations. 

### Sampling the Triples to Explain

First, the triples to explain are needed. We have randomly drawn 50 inferred triples top-ranked by ComplEx. To enable reproducibility, these triples have been stored in the ```triples-to-explain/``` folder with two formats:
1. ```triples_to_explain_kelpie_format.csv```, the Kelpie format.
2. ```triples_to_explain_anyburl_format.txt```, the AnyBURL format.
   
In this way,
1. To run Kelpie, CRIAGE, and Data Poisoning, it is sufficient to move ```triples_to_explain_kelpie_format.csv``` into the ```input_facts/``` folder of the Kelpie repository, as per Kelpie's documentation.
2. To run the AnyBURL Explainer, it is sufficient to move ```triples_to_explain_anyburl_format.txt``` in the  ```triple_to_explain/``` folder of the AnyBURL repository, as explained in [Running the AnyBURL Explainer](#running-the-anyburl-explainer).

### Running Kelpie, CRIAGE, Data Poisoning

It is now possible to run the explainability algorithms. Kelpie, CRIAGE, and Data Poisoning are all implemented within the [Kelpie repository](https://github.com/AndRossi/Kelpie). The commands to extract necessary explanations with these algorithms are as follows.

- Kelpie
```bash 
python scripts/complex/explain.py --dataset FB15k-237 --model_path stored_models/ComplEx_FB15k-237.pt --optimizer Adagrad --dimension 256 --batch_size 2048 --max_epochs 25 --learning_rate 0.01 --reg 0 --facts_to_explain_path input_facts/triples_to_explain_kelpie_format.csv --mode necessary
```

- Kelpie - K1 (explanations of length 1)
```bash 
python scripts/complex/explain.py --dataset FB15k-237 --model_path stored_models/ComplEx_FB15k-237.pt --optimizer Adagrad --dimension 256 --batch_size 2048 --max_epochs 25 --learning_rate 0.01 --reg 0 --facts_to_explain_path input_facts/triples_to_explain_kelpie_format.csv --mode necessary --baseline k1
```

- Data Poisoning
```bash 
python scripts/complex/explain.py --dataset FB15k-237 --model_path stored_models/ComplEx_FB15k-237.pt --optimizer Adagrad --dimension 256 --batch_size 2048 --max_epochs 25 --learning_rate 0.01 --reg 0 --facts_to_explain_path input_facts/triples_to_explain_kelpie_format.csv --mode necessary --baseline data_poisoning
```

- CRIAGE
```bash 
python scripts/complex/explain.py --dataset FB15k-237 --model_path stored_models/ComplEx_FB15k-237.pt --optimizer Adagrad --dimension 256 --batch_size 2048 --max_epochs 25 --learning_rate 0.01 --reg 0 --facts_to_explain_path input_facts/triples_to_explain_kelpie_format.csv --mode necessary --baseline criage
```

These commands return the explanations extracted for the triples in ```triples_to_explain_kelpie_format.csv```. To compute the rank changes of such triples, it is necessary to retrain ComplEx <ins>once after each run above</ins>. The command is
```bash 
python scripts/complex/verify_explanations.py --dataset FB15k-237 --model_path stored_models/ComplEx_FB15k-237.pt --optimizer Adagrad --dimension 256 --batch_size 2048 --max_epochs 25 --learning_rate 0.01 --reg 0 --mode necessary
```

This procedure outputs an ```output.txt``` file containing the explanations of each inferred triple, which can be used to compute the lengths of the explanations, and an ```output_end_to_end.csv``` file with the rank changes of the predictions.

<ins>Note:</ins> All three explainability algorithms here described are run with their default configuration set in the Kelpie repository.

### Running the AnyBURL Explainer

To run the AnyBURL explainer, it is possible to follow the steps detailed in the [AnyBURL repository](https://web.informatik.uni-mannheim.de/AnyBURL/). The important steps, made compatible with our setup, are as follows.

1. Download and unzip AnyBURL v22 as per [AnyBURL's documentation](https://web.informatik.uni-mannheim.de/AnyBURL/#download). The repository default level is ```src/```.
2. At default level, create a ```build/``` folder, compile the Java source code, and package the compiled file into a JAR file. This can be done by running the commands
```bash 
mkdir build
javac de/unima/ki/anyburl/*.java -d build
jar cfv AnyBURL-22.jar -C build .
rm -r build
```
1. Download the FB15k-237 dataset in a format compatible with AnyBURL. The dataset can be found as a zipped file in the AnyBURL repository. Unzip it and paster the ```data/FB15-237/``` folder at default level.
2. Create a ```triple_to_explain/``` folder at default level. Here, copy the ```triples_to_explain_anyburl_format.txt``` file and rename it to ```target.txt``` to make it compatible with the AnyBURL Explainer.
3. Run the AnyBURL Explainer using
```bash 
java -Xmx3G -cp AnyBURL-22.jar de.unima.ki.anyburl.Explain ../triples_to_explain/ ../data/FB15-237/
```

This procedure outputs necessary explanations in ```triple_to_explain/delete.txt```. The file can be reformatted as the ```output.txt``` file of Kelpie, to then retrain ComplEx upon removing AnyBURL's explanations with
```
python scripts/complex/verify_explanations.py --dataset FB15k-237 --model_path stored_models/ComplEx_FB15k-237.pt --optimizer Adagrad --dimension 256 --batch_size 2048 --max_epochs 25 --learning_rate 0.01 --reg 0 --mode necessary
```

<ins>Note:</ins> The AnyBURL explainer is run with its default configuration set in the AnyBURL repository. 

## Reproducing Figures and Data

For convenience, we stored all experiments result in the ```data/``` folder.

In ```data/complex_training/```, we pickled all data concerning ComplEx training.
In ```data/explainers-output/```, we pickled all data concerning the post-hoc explanations found by each algorithm.

These data can be conveniently processed by the Jupyter notebooks in ```notebooks/```.

Figures are saved in ```figures/```

## License
The code in this repository is released under the MIT License. Please refer to the LICENSE file for detailed licensing information.

## Contact
If you have any questions, feedback, or inquiries about the code, please reach out to the authors.<br/>
If you encounter any issues with the code or have suggestions, please open an issue on this repository.