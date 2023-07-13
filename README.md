# Dataset for: Design Pattern Detection on Heterogeneous Information Network

## 1 Structure of the Dataset

### a. HINs

* jhotdraw.db.dump
* junit.db.dump
* quickuml.db.dump
* tested_systems_code

### b. Detection

* cypher.md
* dp_cypher_with_methods.py
* dp_cypher_without_methods.py
* run.ipynb

### c. Results

* Evaluation Results.xlsx
* jhotdraw(with_methods).xlsx
* jhotdraw(without_methods).xlsx
* junit(with_methods).xlsx
* junit(without_methods).xlsx
* quickuml(with_methods).xlsx
* quickuml(without_methods).xlsx
* P-MARt.xls
* Xiong et.al dataset.pdf

## 2 How to Use the Data?

### View results

`c.Results/Evaluation Results.xlsx` shows the main result compared with other DPD tools.

`P-MARt.xls` is download from https://www.ptidej.net/tools/designpatterns/index_html/

> Yann-Gaël Guéhéneuc. (2007). P-MARt : Pattern-like Micro Architecture Repository. ..

`Xiong et.al dataset.pdf` is download from https://github.com/Megre/Dataset4SparT

> Xiong, R., & Li, B. (2019). Accurate Design Pattern Detection Based on Idiomatic Implementation Matching in Java Language Context. 2019 IEEE 26th International Conference on Software Analysis, Evolution and Reengineering (SANER). https://doi.org/10.1109/saner.2019.8668031

### Repeated experiment

`a.HINs` offers neo4j dump files of the tested three open systems. Neo4j-community must be download first from [Neo4j Download Center - Graph Database & Analytics](https://neo4j.com/download-center/#community), our version is `4.4.19`. And then execute the following codes to start a neo4j instanc:

```bash
cd neo4j-community/bin
./neo4j-admin load --from=xxx\Dataset4HIN-DPD-master\a.HINs\xxx.db.dump
./neo4j console
```

After neo4j instance started ,there are two ways to get results.

#### Neo4j Brower

Open http://localhost:7474/ , the default user_name and password are `neo4j` and `123456`.

Then run Cypher statements in `b.Detection/cypher.md`. 

This method is semi-automatic with a has a good visualization effect.

#### Juphter

Start JupyterLab or JupyterNotebook, create a project with files in `b. Detection`, and execute the file `run.ipynb`.

As a result, you can get the following files:

* jhotdraw(with_methods).xlsx
* jhotdraw(without_methods).xlsx
* junit(with_methods).xlsx
* junit(without_methods).xlsx
* quickuml(with_methods).xlsx
* quickuml(without_methods).xlsx

### Match new pattern

You can define your patterns and decompose them into micro-structures, and then code related Cycpher statements to match them in the HIN.