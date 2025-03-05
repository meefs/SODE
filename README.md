# DLCoT


## How to reproduce
To run the code, you need to be able to utlize LLM for inference. Most steps generate files in the chatml format, where the data with the key "input" serves as the LLM's input. In the code, we assume that after each step, a new field named "result" is added to the original file, containing the LLM's output.

In the paper, we employ Qwen-2.5-72B-Instruct as the model for steps 1, 2, 3, and 5, while GPT-4o is used for step 4.

### Step 1 Extract the pattern of the long chain-of-thought answer

Run inference for the file `data/20250209_deepseek_r1_math_26K.jsonl` we provide. 

### Step 2 Further decompose the solution part and the verification part

#### step 2.1

```python
python pattern_extraction.py --part=basic_structure
```
This will generate three files:

`R1_split_part1.jsonl`: The basic structre of long CoTs generated by R1

`R1_solution_explore.jsonl`: The prompt created for further solution decomposition.

`R1_verify_explore.jsonl`: The prompt created for further verification decomposition.

You need to run inference for `R1_solution_explore.jsonl` and `R1_verify_explore.jsonl`

#### step 2.2
```python
python pattern_extraction.py --part=solution_verification_explore
```
This will generate a file named 
`R1_split_part2_filtered.jsonl`. This is the intermediate version of long CoT decomposition. You need to run inference for it.


#### step 2.3
```python
python solution_clustering.py --status=complete_answer
```
This step improves the division between the solution section and the verification section, making the final solution more complete. A file named `R1_split_part2_filtered_complete_answer.jsonl` will be generated. You need to run inference for it.

### Step 3 Solution labeling
```python
python solution_classification.py
```
This will generate a file named `R1_split_part2_filtered_solution_classification.jsonl`: solution(approach) labeling

### Step 4 Solution clustering
#### step 4.1
```python
python solution_clustering.py --status=clustering
```
This will generate a file named `R1_split_part2_filtered_clustering.jsonl`: input of solution(approach) clustering 

`R1_split_part2_filtered_complete_answer_output.json`: The final result of decomposition of the long CoT

You need to run LLM inference for `R1_split_part2_filtered_clustering.jsonl`


#### step 4.2
```python
python solution_clustering.py --status=clustering_extract
```
This will generate a file named `R1_split_part2_filtered_need_filter.json`, which adds clustering information to the final result.

### Step 5 Chain-of-thought optimization

There are two seperate strategies we design for solution optimization, redundancy reduction and incorrectness reduction.

#### step 5.1 Solution optimization by redundancy reduction
```python
python redundancy_deduction.py
```
This will generate files:

`R1_split_part2_filtered_delete_multi_1_need_simplize.jsonl`

`R1_split_part2_filtered_delete_multi_2_need_simplize.jsonl`

`R1_split_part2_filtered_delete_multi_all_need_simplize.jsonl`

#### step 5.2 Solution optimization by incorrectness reduction (optional)
```python
python incorrectness_deduction.py
```
This step will generate a file named `R1_split_part2_ablation_keep_one_global_delete_all_need_simplize.jsonl`

There are more settings in the code, but they do not yield good performace. You could try on your own data if you are interested.



By running LLM inference using files in step 5, you will be able to get the optimized training data.