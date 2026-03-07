# ECMFA 2026
This repository is to provide supplemental materials for the paper 17 to the ECMFA 2026 conference.
In this paper, we propose a LLM-based approach for metamodel-grammar co-evolution, and compare it with a rule-based methods.
Six case languages are divided into two groups, i.e., the training set, and the test set.
## Directory: training_set
In the training set, the metamodels and grammars of four case languages ​​(BibTeX, EAST-ADL, SML, and Xenia) were used to develop and validate prompting strategies. For each of them, we created a dedicated subdirectory.

It's important to note that the "training" in the training set is not training in the LLM sense—the LLM itself doesn't learn anything from these four DSLs (each DSL is processed in an independent session, with no information exchange between sessions). The term "training set" refers to the researchers using these DSLs to develop and debug prompting policies; it corresponds to researcher-level learning, not model-level learning. This point is also clearly clarified in Section 3.2.

### BibTeX
Iterative development of prompts in BibTeX.
### EAST-ADL
Iterative development of prompts in EAST-ADL.
#### Claude_Iterative_Development
This subdirectory contains a record of the prompts we used for iterative development on EAST-ADL.
#### Claude_Test
The finalized prompts we developed in three small-to-medium-sized case languages ​​(BibTeX, SML, and Xenia) were tested on EAST-ADL. This subdirectory contains the test output files (note that this differs from the prompt strategy used in iterative development).
#### ChatGPT_Test
We also used ChatGPT to perform LLM-based co-evolution tests on EAST-ADL.
#### Gemini_Test
We also used Gemini to perform LLM-based co-evolution tests on EAST-ADL.
### SML
Iterative development of prompts in SML.
### Xenia
Iterative development of prompts in Xenia.
## Directory: test_set
In the test set, two DSLs (DOT and Xcore) are used to verify the generalization ability of finalized prompts.
## Directory: longitudinal_study
We also conducted longitudinal study of the LLM-based co-evolution approach on four versions of QVTo.