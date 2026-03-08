# ECMFA 2026
This repository is to provide supplemental materials for the paper 17 to the ECMFA 2026 conference.
In this paper, we propose a LLM-based approach for metamodel-grammar co-evolution, and compare it with a rule-based methods.
Six case languages are divided into two groups, i.e., the training set, and the test set.
## 1 Directory: training_set
In the training set, the metamodels and grammars of four case languages ​​(BibTeX, EAST-ADL, SML, and Xenia) were used to develop and validate prompting strategies. For each of them, we created a dedicated subdirectory.

It's important to note that the "training" in the training set is not training in the LLM sense—the LLM itself doesn't learn anything from these four DSLs (each DSL is processed in an independent session, with no information exchange between sessions). The term "training set" refers to the researchers using these DSLs to develop and debug prompting policies; it corresponds to researcher-level learning, not model-level learning. This point is also clearly clarified in Section 3.2.

### 1.1 BibTeX
Iterative development of prompts in BibTeX.
### 1.2 EAST-ADL
Iterative development of prompts in EAST-ADL.
#### 1.2.1 Claude_Iterative_Development
This subdirectory contains a record of the prompts we used for iterative development on EAST-ADL.
#### 1.2.2 Claude_Test
The finalized prompts we developed in three small-to-medium-sized case languages ​​(BibTeX, SML, and Xenia) were tested on EAST-ADL. This subdirectory contains the test output files (note that this differs from the prompt strategy used in iterative development).
#### 1.2.3 ChatGPT_Test
We also used ChatGPT to perform LLM-based co-evolution tests on EAST-ADL.
#### 1.2.4 Gemini_Test
We also used Gemini to perform LLM-based co-evolution tests on EAST-ADL.
### 1.3 SML
Iterative development of prompts in SML.
### 1.4 Xenia
Iterative development of prompts in Xenia.
## 2 Directory: test_set
In the test set, two DSLs (DOT and Xcore) are used to verify the generalization ability of finalized prompts.
## 3 Directory: longitudinal_study
We also conducted longitudinal study of the LLM-based co-evolution approach on four versions of QVTo.

## 4 Experiment Execution Method
The experiment was executed independently for each DSL. For each DSL, the experiment proceeds in two steps. 
- In the first step, the generated grammar G1, the manually adapted grammar G1', and Prompt 1 are provided to the LLM for analysis and comparison. 
- In the second step, the generated grammar G2 and Prompt 2 are provided to the LLM, whose output yields the adapted grammar G2'. 

Taking BibTeX as an example: 
- in the first step, "MyBibTex_generated_grammar.txt" (as G1) and "MyBibTex_target_grammar.txt" (as G1') are provided together with Prompt 1; 
- in the second step, "MyBibTex_generated_grammar.txt" is provided again (this time as G2) together with Prompt 2. Note that for these six DSLs, G1 and G2 refer to the same generated grammar file; this is inherited from the experimental setup of prior work and is acknowledged as a threat to external validity in the paper (see Section 5.2).

I put the contents of gramamr and instances in .txt files to account for the possibility that readers may not have Xtext installed. Once a .txt file name contains "target", indicating that it is G1', meaning the grammar adapted from G1 (G1 is the grammar generated from the metamodel before the evolution).