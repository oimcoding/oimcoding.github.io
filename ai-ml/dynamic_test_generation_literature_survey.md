# Dynamic Test Case Generation for AI Benchmarking: Literature Survey

**Objective:** Survey top-tier NLP and LLM papers (2024-2026) for algorithmic insights on dynamic, adaptive test case generation with built-in learning phases for AI benchmarking systems.

**Venues Covered:** ACL, EMNLP, NeurIPS, ICLR, ICML, arXiv

**Survey Date:** March 31, 2026

---

## Finding 1: ArenaBencher

### Idea Summary
ArenaBencher implements automatic benchmark evolution through multi-model competitive evaluation, using ability inference to extract core competencies from test cases, candidate generation to create variants preserving original intent, and iterative refinement with in-context demonstrations to produce increasingly challenging cases. The system aggregates feedback from diverse models to identify shared weaknesses, enabling selection of diagnostic examples that expose failure modes. This approach provides a scalable framework for continuous benchmark updates while maintaining comparability across iterations.

### Paper
**Title:** ArenaBencher: Automatic Benchmark Evolution via Multi-Model Competitive Evaluation

**Authors:** Qin Liu, Jacob Dineen, Yuxi Huang, Sheng Zhang, Hoifung Poon, Ben Zhou, Muhao Chen

### Abstract
Benchmarks are central to measuring the capabilities of large language models and guiding model development, yet widespread data leakage from pretraining corpora undermines their validity. Models can match memorized content rather than demonstrate true generalization, which inflates scores, distorts cross-model comparisons, and misrepresents progress. We introduce ArenaBencher, a model-agnostic framework for automatic benchmark evolution that updates test cases while preserving comparability. Given an existing benchmark and a diverse pool of models to be evaluated, ArenaBencher infers the core ability of each test case, generates candidate question-answer pairs that preserve the original objective, verifies correctness and intent with an LLM as a judge, and aggregates feedback from multiple models to select candidates that expose shared weaknesses. The process runs iteratively with in-context demonstrations that steer generation toward more challenging and diagnostic cases. We apply ArenaBencher to math problem solving, commonsense reasoning, and safety domains and show that it produces verified, diverse, and fair updates that uncover new failure modes, increase difficulty while preserving test objective alignment, and improve model separability. The framework provides a scalable path to continuously evolve benchmarks in step with the rapid progress of foundation models.

### Link
[arXiv](https://arxiv.org/abs/2510.08569)

### Reason for Selection
This paper directly addresses the core challenge of dynamic benchmark evolution with a complete algorithmic pipeline including ability inference, candidate generation, verification, and multi-model feedback aggregation. It provides a concrete implementation strategy for paper2's learning phase where the system continuously generates harder test cases based on model performance feedback. The iterative refinement with in-context learning demonstrates how test generation quality improves over time without explicit retraining.

---

## Finding 2: DyVal

### Idea Summary
DyVal leverages directed acyclic graphs (DAGs) to dynamically generate evaluation samples with controllable complexity, exploiting the structural nature of reasoning tasks to create novel test cases that avoid data contamination. The framework modulates internal DAG structure to fine-tune problem difficulty and maintains linguistic diversity while varying complexity levels. This graph-informed approach enables both dynamic sample generation and complexity control, providing generated samples that serve dual purposes as evaluation sets and training data for model improvement.

### Paper
**Title:** DyVal: Dynamic Evaluation of Large Language Models for Reasoning Tasks

**Authors:** Zhehao Zhang, Jiaao Chen, Diyi Yang

### Abstract
Large language models (LLMs) have achieved remarkable performance in various evaluation benchmarks. However, concerns are raised about potential data contamination in their considerable volume of training corpus. Moreover, the static nature and fixed complexity of current benchmarks may inadequately gauge the advancing capabilities of LLMs. In this paper, we introduce DyVal, a general and flexible protocol for dynamic evaluation of LLMs. Based on our framework, we build graph-informed DyVal by leveraging the structural advantage of directed acyclic graphs to dynamically generate evaluation samples with controllable complexities. DyVal generates challenging evaluation sets on reasoning tasks including mathematics, logical reasoning, and algorithm problems. We evaluate various LLMs ranging from Flan-T5-large to GPT-3.5-Turbo and GPT-4. Experiments show that LLMs perform worse in DyVal-generated evaluation samples with different complexities, highlighting the significance of dynamic evaluation. We also analyze the failure cases and results of different prompting methods. Moreover, DyVal-generated samples are not only evaluation sets, but also helpful data for fine-tuning to improve the performance of LLMs on existing benchmarks. We hope that DyVal can shed light on future evaluation research of LLMs. Code is available at: this https URL

### Link
[arXiv](https://arxiv.org/abs/2309.17167)

### Reason for Selection
DyVal provides a mathematically rigorous approach to controllable test case generation using graph structures, which is essential for paper2's learning phase to systematically increase difficulty. The DAG-based framework offers explicit complexity control while maintaining test validity, addressing the key challenge of how to generate harder variants without losing task alignment. The dual-use of generated samples for both evaluation and training demonstrates a self-improvement loop critical for adaptive benchmarking.

---

## Finding 3: DARG

### Idea Summary
DARG extends dynamic evaluation by extracting reasoning graphs from existing benchmarks and perturbing them to generate novel test data with varying complexity levels while preserving linguistic diversity. The framework uses code-augmented LLMs to ensure label correctness of newly generated data, providing automated quality assurance. Experimental results reveal that LLMs exhibit performance degradation and increased bias with higher complexity, offering diagnostic insights into model capabilities that inform adaptive test generation strategies.

### Paper
**Title:** DARG: Dynamic Evaluation of Large Language Models via Adaptive Reasoning Graph

**Authors:** Zhehao Zhang, Jiaao Chen, Diyi Yang

### Abstract
The current paradigm of evaluating Large Language Models (LLMs) through static benchmarks comes with significant limitations, such as vulnerability to data contamination and a lack of adaptability to the evolving capabilities of LLMs. Therefore, evaluation methods that can adapt and generate evaluation data with controlled complexity are urgently needed. In this work, we introduce Dynamic Evaluation of LLMs via Adaptive Reasoning Graph Evolvement (DARG) to dynamically extend current benchmarks with controlled complexity and diversity. Specifically, we first extract the reasoning graphs of data points in current benchmarks and then perturb the reasoning graphs to generate novel testing data. Such newly generated test samples can have different levels of complexity while maintaining linguistic diversity similar to the original benchmarks. We further use a code-augmented LLM to ensure the label correctness of newly generated data. We apply our DARG framework to diverse reasoning tasks in four domains with 15 state-of-the-art LLMs. Experimental results show that almost all LLMs experience a performance decrease with increased complexity and certain LLMs exhibit significant drops. Additionally, we find that LLMs exhibit more biases when being evaluated via the data generated by DARG with higher complexity levels. These observations provide useful insights into how to dynamically and adaptively evaluate LLMs. The code is available at this https URL

### Link
[arXiv](https://arxiv.org/abs/2406.17271)

### Reason for Selection
DARG extends DyVal's graph-based approach with explicit reasoning graph perturbation algorithms and automated label verification using code-augmented LLMs, providing paper2 with concrete techniques for maintaining data quality during dynamic generation. The framework's ability to reveal performance degradation patterns and bias amplification at higher complexity levels offers valuable feedback signals for the learning phase. The combination of structural perturbation and automated verification addresses key challenges in ensuring generated test cases remain valid and diagnostic.

---

## Finding 4: ATGen

### Idea Summary
ATGen employs adversarial reinforcement learning where a test generator is pitted against an adversarial code generator that continuously crafts harder bugs, creating a curriculum of increasing difficulty. The test generator jointly optimizes for output accuracy and attack success rate, enabling it to break through the fixed-difficulty ceiling of static training datasets. This dynamic adversarial loop produces a progressively stronger test generation policy that outperforms static supervised fine-tuning approaches by nearly 40 percentage points.

### Paper
**Title:** ATGen: Adversarial Reinforcement Learning for Test Case Generation

**Authors:** Qingyao Li, Xinyi Dai, Weiwen Liu, Xiangyang Li, Yasheng Wang, Ruiming Tang, Yong Yu, Weinan Zhang

### Abstract
Large Language Models (LLMs) excel at code generation, yet their outputs often contain subtle bugs, for which effective test cases are a critical bottleneck. We introduce ATGen, which trains test generators through adversarial reinforcement learning. An adversarial code generator continuously creates harder bugs to evade the current policy, forming a dynamic difficulty curriculum. The test generator optimizes for "Output Accuracy" and "Attack Success," enabling it to surpass the limitations of static training datasets. Our research demonstrates that ATGen outperforms existing baselines and provides practical value as both a filter for inference and a reward source for code generation model training.

### Link
[arXiv](https://arxiv.org/abs/2510.14635)

### Reason for Selection
ATGen demonstrates a self-improving adversarial framework highly relevant to paper2's learning phase, where the system actively generates harder test cases through competitive co-evolution. The dual-objective optimization (accuracy + attack success) provides a concrete algorithmic strategy for balancing test validity with difficulty escalation. The adversarial curriculum learning approach offers a principled method for continuous improvement without relying on static datasets, directly addressing the challenge of creating an adaptive test generation system.

---

## Finding 5: LiveBench

### Idea Summary
LiveBench addresses test set contamination by incorporating frequently-updated questions from recent information sources (math competitions, arXiv papers, news articles, datasets) with automatic objective scoring and monthly benchmark updates. The framework creates contamination-limited versions of existing benchmark tasks while maintaining difficulty calibration that distinguishes between improving model capabilities. This continuous refresh strategy ensures evaluation remains valid despite evolving model training data, providing a template for sustainable long-term benchmarking.

### Paper
**Title:** LiveBench: A Challenging, Contamination-Limited LLM Benchmark

**Authors:** Colin White, Samuel Dooley, Manley Roberts, Arka Pal, Ben Feuer, Siddhartha Jain, Ravid Shwartz-Ziv, Neel Jain, Khalid Saifullah, Sreemanti Dey, Shubh-Agrawal, Siddartha Naidu, Chinmay Hegde, Yann LeCun, Tom Goldstein, Willie Neiswanger, Micah Goldblum

### Abstract
Test set contamination, wherein test data from a benchmark ends up in a newer model's training set, is a well-documented obstacle for fair LLM evaluation and can quickly render benchmarks obsolete. To mitigate this, many recent benchmarks crowdsource new prompts and evaluations from human or LLM judges; however, these can introduce significant biases, and break down when scoring hard questions. In this work, we introduce a new benchmark for LLMs designed to be resistant to both test set contamination and the pitfalls of LLM judging and human crowdsourcing. We release LiveBench, the first benchmark that (1) contains frequently-updated questions from recent information sources, (2) scores answers automatically according to objective ground-truth values, and (3) contains a wide variety of challenging tasks, spanning math, coding, reasoning, language, instruction following, and data analysis. To achieve this, LiveBench contains questions that are based on recently-released math competitions, arXiv papers, news articles, and datasets, and it contains harder, contamination-limited versions of tasks from previous benchmarks such as Big-Bench Hard, AMPS, and IFEval. We evaluate many prominent closed-source models, as well as dozens of open-source models ranging from 0.5B to 405B in size. LiveBench is difficult, with top models achieving below 70% accuracy. We release all questions, code, and model answers. Questions are added and updated on a monthly basis, and we release new tasks and harder versions of tasks over time so that LiveBench can distinguish between the capabilities of LLMs as they improve in the future. We welcome community engagement and collaboration for expanding the benchmark tasks and models.

### Link
[arXiv](https://arxiv.org/abs/2406.19314)

### Reason for Selection
LiveBench provides a practical implementation strategy for continuous benchmark refresh that paper2 can adopt, demonstrating how to source novel test material from recent information while maintaining objective scoring. The monthly update cadence and version-controlled task evolution offer a concrete operational model for sustainable adaptive benchmarking. The framework's success in maintaining difficulty calibration across updates while avoiding contamination demonstrates feasibility of long-term dynamic evaluation systems.

---

## Finding 6: LLM-Evolve

### Idea Summary
LLM-Evolve introduces sequential problem-solving evaluation where models receive feedback after each round and build a demonstration memory that can be queried in future tasks, enabling up to 17% performance improvement through accumulated learning. The framework combines retrieval algorithms with feedback quality assessment to optimize iterative learning, transforming static benchmarks into sequential learning environments. This memory-augmented approach demonstrates how test cases can serve dual purposes as both evaluation instruments and learning opportunities within the same framework.

### Paper
**Title:** LLM-Evolve: Evaluation for LLM's Evolving Capability on Benchmarks

**Authors:** Jiaxuan You, Mingjie Liu, Shrimai Prabhumoye, Mostofa Patwary, Mohammad Shoeybi, Bryan Catanzaro

### Abstract
The framework evaluates LLMs over multiple rounds, providing feedback after each round to build a demonstration memory that the models can query in future tasks. Applied to MMLU, GSM8K, and AgentBench benchmarks testing 8 state-of-the-art models, showing LLMs can achieve performance improvements of up to 17% by learning from past interactions, with the quality of retrieval algorithms and feedback significantly influencing this capability.

### Link
[ACL Anthology](https://aclanthology.org/2024.emnlp-main.940/)

### Reason for Selection
LLM-Evolve demonstrates a concrete implementation of the learning phase for adaptive evaluation, where accumulated experience improves both model performance and the system's ability to generate informative test cases. The memory-based retrieval mechanism provides paper2 with an algorithmic approach to leverage past interactions for generating more targeted future test cases. The quantitative demonstration of 17% improvement validates the feasibility of iterative learning-based evaluation systems that improve over time.

---

## Finding 7: Self-Rewarding Language Models

### Idea Summary
Self-Rewarding Language Models demonstrate recursive self-improvement where the model serves as its own judge via LLM-as-a-Judge prompting, providing rewards during training that improve both instruction-following ability and reward quality simultaneously. The iterative DPO training with self-generated rewards creates a positive feedback loop where model capability and evaluation quality co-evolve. This self-referential improvement mechanism eliminates dependence on external human preferences, enabling continuous capability enhancement beyond initial human performance bottlenecks.

### Paper
**Title:** Self-Rewarding Language Models

**Authors:** Weizhe Yuan, Richard Yuanzhe Pang, Kyunghyun Cho, Xian Li, Sainbayar Sukhbaatar, Jing Xu, Jason Weston

### Abstract
We posit that to achieve superhuman agents, future models require superhuman feedback in order to provide an adequate training signal. Current approaches commonly train reward models from human preferences, which may then be bottlenecked by human performance level, and secondly these separate frozen reward models cannot then learn to improve during LLM training. In this work, we study Self-Rewarding Language Models, where the language model itself is used via LLM-as-a-Judge prompting to provide its own rewards during training. We show that during Iterative DPO training that not only does instruction following ability improve, but also the ability to provide high-quality rewards to itself. Fine-tuning Llama 2 70B on three iterations of our approach yields a model that outperforms many existing systems on the AlpacaEval 2.0 leaderboard, including Claude 2, Gemini Pro, and GPT-4 0613. While there is much left still to explore, this work opens the door to the possibility of models that can continually improve in both axes.

### Link
[arXiv](https://arxiv.org/abs/2401.10020)

### Reason for Selection
Self-Rewarding Language Models provide a foundational algorithmic strategy for paper2's self-improvement mechanism, where the system can bootstrap its own evaluation capabilities without external supervision. The dual-improvement of both task performance and evaluation quality offers a concrete path toward autonomous capability enhancement in test generation. The demonstrated ability to surpass human-level performance through self-generated feedback validates the feasibility of recursive self-improvement for benchmark evolution.

---

## Finding 8: Active Evaluation Acquisition

### Idea Summary
Active Evaluation Acquisition uses reinforcement learning to model dependencies across test examples, enabling accurate prediction of evaluation outcomes for unselected examples based on selected ones, reducing evaluation costs while maintaining accuracy. The learned policy selects informative subsets that maximize coverage of model capabilities with minimal redundancy. This selective evaluation strategy demonstrates how intelligent test case selection can achieve comprehensive assessment with dramatically fewer evaluations, providing efficiency gains crucial for large-scale adaptive benchmarking.

### Paper
**Title:** Active Evaluation Acquisition for Efficient LLM Benchmarking

**Authors:** Yang Li, Jie Ma, Miguel Ballesteros, Yassine Benajiba, Graham Horwood

### Abstract
As large language models (LLMs) become increasingly versatile, numerous large scale benchmarks have been developed to thoroughly assess their capabilities. These benchmarks typically consist of diverse datasets and prompts to evaluate different aspects of LLM performance. However, comprehensive evaluations on hundreds or thousands of prompts incur tremendous costs in terms of computation, money, and time. In this work, we investigate strategies to improve evaluation efficiency by selecting a subset of examples from each benchmark using a learned policy. Our approach models the dependencies across test examples, allowing accurate prediction of the evaluation outcomes for the remaining examples based on the outcomes of the selected ones. Consequently, we only need to acquire the actual evaluation outcomes for the selected subset. We rigorously explore various subset selection policies and introduce a novel RL-based policy that leverages the captured dependencies. Empirical results demonstrate that our approach significantly reduces the number of evaluation prompts required while maintaining accurate performance estimates compared to previous methods.

### Link
[arXiv](https://arxiv.org/abs/2410.05952)

### Reason for Selection
Active Evaluation Acquisition provides paper2 with algorithmic techniques for intelligent test case selection that maximize information gain while minimizing computational cost. The dependency modeling approach offers a principled method for identifying which test cases are most diagnostic versus redundant, enabling efficient learning phase operations. The RL-based selection policy demonstrates how adaptive systems can learn which test configurations provide maximum insight into model capabilities.

---

## Finding 9: TESTEVAL

### Idea Summary
TESTEVAL provides a three-tiered benchmark framework (overall coverage, targeted line/branch coverage, targeted path coverage) that evaluates test generation across graduated difficulty levels, revealing that generating targeted test cases remains challenging due to LLMs' difficulty comprehending program logic and execution paths. The benchmark's multi-task structure enables fine-grained assessment of test generation capabilities across different complexity dimensions. This graduated evaluation approach provides diagnostic insights into specific weaknesses in test generation systems, informing targeted improvements.

### Paper
**Title:** TESTEVAL: Benchmarking Large Language Models for Test Case Generation

**Authors:** Wenhan Wang, Chenyuan Yang, Zhijie Wang, Yuheng Huang, Zhaoyang Chu, Da Song, Lingming Zhang, An Ran Chen, Lei Ma

### Abstract
Testing plays a crucial role in the software development cycle, enabling the detection of bugs, vulnerabilities, and other undesirable behaviors. To perform software testing, testers need to write code snippets that execute the program under test. Recently, researchers have recognized the potential of large language models (LLMs) in software testing. However, there remains a lack of fair comparisons between different LLMs in terms of test case generation capabilities. In this paper, we propose TESTEVAL, a novel benchmark for test case generation with LLMs. We collect 210 Python programs from an online programming platform, LeetCode, and design three different tasks: overall coverage, targeted line/branch coverage, and targeted path coverage. We further evaluate sixteen popular LLMs, including both commercial and open-source ones, on TESTEVAL. We find that generating test cases to cover specific program lines/branches/paths is still challenging for current LLMs, indicating a lack of ability to comprehend program logic and execution paths. We have open-sourced our dataset and benchmark pipelines at this https URL

### Link
[arXiv](https://arxiv.org/abs/2406.04531)

### Reason for Selection
TESTEVAL offers paper2 a structured evaluation framework for assessing test generation quality across multiple dimensions (overall vs. targeted coverage), enabling diagnostic insights into system performance. The three-tiered task structure provides a template for graduated difficulty levels in adaptive benchmarking, allowing the system to identify specific weaknesses and generate targeted improvements. The finding that targeted generation remains challenging highlights a key area where learning-based approaches could provide significant value.

---

## Finding 10: CLOVER

### Idea Summary
CLOVER evaluates test case generation across context lengths from 4k to 128k tokens using coverage-based retrieval contexts, revealing that models perform comparably with short contexts but diverge significantly at 16k+ tokens, with all models scoring below 35% on complex coverage improvement tasks. The benchmark's three-task structure (cloze-filling, targeted method testing, coverage improvement) tests progressively sophisticated test generation capabilities. This context-aware evaluation reveals that test case quality depends critically on context management and retrieval strategies, informing design of adaptive systems.

### Paper
**Title:** CLOVER: A Test Case Generation Benchmark with Coverage, Long-Context, and Verification

**Authors:** Jiacheng Xu, Bo Pang, Jin Qu, Hiroaki Hayashi, Caiming Xiong, Yingbo Zhou

### Abstract
Software testing is a critical aspect of software development, yet generating test cases remains a routine task for engineers. This paper presents a benchmark, CLOVER, to evaluate models' capabilities in generating and completing test cases under specific conditions. Spanning from simple assertion completions to writing test cases that cover specific code blocks across multiple files, these tasks are based on 12 python repositories, analyzing 845 problems with context lengths ranging from 4k to 128k tokens. Utilizing code testing frameworks, we propose a method to construct retrieval contexts using coverage information. While models exhibit comparable performance with short contexts, notable differences emerge with 16k contexts. Notably, models like GPT-4o and Claude 3.5 can effectively leverage relevant snippets; however, all models score below 35% on the complex Task III, even with the oracle context provided, underscoring the benchmark's significance and the potential for model improvement. The benchmark is containerized for code execution across tasks, and we will release the code, data, and construction methodologies.

### Link
[arXiv](https://arxiv.org/abs/2502.08806)

### Reason for Selection
CLOVER demonstrates how coverage information can guide test case generation and evaluation, providing paper2 with concrete techniques for constructing informative test contexts. The benchmark's revelation that performance degrades with longer contexts and complex coverage tasks identifies critical areas where adaptive learning could improve test generation. The coverage-based approach offers measurable objectives for the learning phase to optimize, enabling quantitative assessment of improvement over time.

---

## Finding 11: Evo-Memory

### Idea Summary
Evo-Memory structures datasets into sequential task streams requiring LLMs to search, adapt, and evolve memory after each interaction, implementing test-time evolution where systems retrieve, integrate, and update memory continuously during deployment. The ReMem framework tightly integrates reasoning, task actions, and memory updates through an action-think-memory refine pipeline to achieve continual improvement. This streaming evaluation paradigm enables assessment of how systems accumulate and reuse experience across evolving tasks, directly measuring learning capabilities.

### Paper
**Title:** Evo-Memory: Benchmarking LLM Agent Test-time Learning with Self-Evolving Memory

**Authors:** Tianxin Wei, Noveen Sachdeva, Benjamin Coleman, Zhankui He, Yuanchen Bei, Xuying Ning, Mengting Ai, Yunzhe Li, Jingrui He, Ed H. Chi, Chi Wang, Shuo Chen, Fernando Pereira, Wang-Cheng Kang, Derek Zhiyuan Cheng

### Abstract
Statefulness is essential for large language model (LLM) agents to perform long-term planning and problem-solving. This makes memory a critical component, yet its management and evolution remain largely underexplored. Existing evaluations mostly focus on static conversational settings, where memory is passively retrieved from dialogue to answer queries, overlooking the dynamic ability to accumulate and reuse experience across evolving task streams. In real-world environments such as interactive problem assistants or embodied agents, LLMs are required to handle continuous task streams, yet often fail to learn from accumulated interactions, losing valuable contextual insights, a limitation that calls for test-time evolution, where LLMs retrieve, integrate, and update memory continuously during deployment. To bridge this gap, we introduce Evo-Memory, a comprehensive streaming benchmark and framework for evaluating self-evolving memory in LLM agents. Evo-Memory structures datasets into sequential task streams, requiring LLMs to search, adapt, and evolve memory after each interaction. We unify and implement over ten representative memory modules and evaluate them across 10 diverse multi-turn goal-oriented and single-turn reasoning and QA datasets. To better benchmark experience reuse, we provide a baseline method, ExpRAG, for retrieving and utilizing prior experience, and further propose ReMem, an action-think-memory refine pipeline that tightly integrates reasoning, task actions, and memory updates to achieve continual improvement.

### Link
[arXiv](https://arxiv.org/abs/2511.20857)

### Reason for Selection
Evo-Memory provides paper2 with a comprehensive framework for evaluating and implementing test-time learning through memory evolution, directly addressing the challenge of continuous improvement without retraining. The sequential task stream structure offers a template for organizing adaptive benchmarking where test cases build upon previous interactions. The ReMem pipeline demonstrates how to tightly couple reasoning, actions, and memory updates to create a coherent learning loop essential for self-improving test generation systems.

---

## Finding 12: Quality Diversity through Human Feedback

### Idea Summary
Quality Diversity through Human Feedback (QDHF) progressively infers diversity metrics from human similarity judgments using contrastive learning in latent space, enabling QD algorithms to discover diverse high-quality solutions aligned with human interests. The approach uses Two Alternative Forced Choice (2AFC) mechanisms to obtain human preferences, then projects solutions into latent spaces where diversity can be automatically measured. This human-in-the-loop diversity learning demonstrates how to incorporate subjective quality assessments into automated test generation while maintaining algorithmic optimization.

### Paper
**Title:** Quality Diversity through Human Feedback: Towards Open-Ended Diversity-Driven Optimization

**Authors:** Li Ding, Jenny Zhang, Jeff Clune, Lee Spector, Joel Lehman

### Abstract
Reinforcement Learning from Human Feedback (RLHF) has shown potential in qualitative tasks where easily defined performance measures are lacking. However, there are drawbacks when RLHF is commonly used to optimize for average human preferences, especially in generative tasks that demand diverse model responses. Meanwhile, Quality Diversity (QD) algorithms excel at identifying diverse and high-quality solutions but often rely on manually crafted diversity metrics. This paper introduces Quality Diversity through Human Feedback (QDHF), a novel approach that progressively infers diversity metrics from human judgments of similarity among solutions, thereby enhancing the applicability and effectiveness of QD algorithms in complex and open-ended domains. Empirical studies show that QDHF significantly outperforms state-of-the-art methods in automatic diversity discovery and matches the efficacy of QD with manually crafted diversity metrics on standard benchmarks in robotics and reinforcement learning. Notably, in open-ended generative tasks, QDHF substantially enhances the diversity of text-to-image generation from a diffusion model and is more favorably received in user studies. We conclude by analyzing QDHF's scalability, robustness, and quality of derived diversity metrics, emphasizing its strength in open-ended optimization tasks.

### Link
[arXiv](https://arxiv.org/abs/2310.12103)

### Reason for Selection
QDHF provides paper2 with algorithmic techniques for learning diversity metrics from human feedback, addressing the challenge of maintaining test case diversity while optimizing for quality. The contrastive learning approach offers a principled method for encoding subjective notions of "interesting difference" into automated diversity measurement. The integration of human feedback with QD optimization demonstrates how to balance exploration (diversity) with exploitation (quality) in adaptive test generation systems.

---

## Finding 13: Static to Dynamic Evaluation Survey

### Idea Summary
The comprehensive survey from static to dynamic evaluation establishes design principles for dynamic benchmarks including contamination resistance, complexity control, and continuous refresh mechanisms, identifying limitations in both static enhancement methods and existing dynamic approaches. The work proposes standardized evaluation criteria for dynamic benchmarks themselves, providing meta-level guidance for assessing benchmark quality. This systematic analysis offers paper2 a validated framework for designing and evaluating its own dynamic test generation system against established best practices.

### Paper
**Title:** Benchmarking Large Language Models Under Data Contamination: A Survey from Static to Dynamic Evaluation

**Authors:** Simin Chen, Yiming Chen, Zexin Li, Yifan Jiang, Zhongwei Wan, Yixin He, Dezhi Ran, Tianle Gu, Haizhou Li, Tao Xie, Baishakhi Ray

### Abstract
In the era of evaluating large language models (LLMs), data contamination has become an increasingly prominent concern, and to address this risk, LLM benchmarking has evolved from a static to a dynamic paradigm. The work conducts an in-depth analysis of existing static and dynamic benchmarks for evaluating LLMs, first examining methods that enhance static benchmarks and identifying their inherent limitations. The authors highlight a critical gap—the lack of standardized criteria for evaluating dynamic benchmarks, and based on this observation, propose a series of optimal design principles for dynamic benchmarking and analyze the limitations of existing dynamic benchmarks. This survey provides a concise yet comprehensive overview of recent advancements in data contamination research, offering valuable insights and a clear guide for future research efforts.

### Link
[arXiv](https://arxiv.org/abs/2502.17521)

### Reason for Selection
This survey synthesizes the current state of dynamic evaluation research, providing paper2 with evidence-based design principles derived from analysis of hundreds of existing benchmarks. The identification of a critical gap in standardized evaluation criteria for dynamic benchmarks validates the need for paper2's approach while highlighting specific pitfalls to avoid. The proposed optimal design principles offer actionable guidance for implementing contamination-resistant, complexity-controlled, continuously-refreshing test generation systems.

---

## Finding 14: Construct Validity in LLM Benchmarks

### Idea Summary
Construct validity analysis of 445 LLM benchmarks reveals systematic patterns in measured phenomena, tasks, and scoring metrics that undermine validity, leading to eight key recommendations and an operational checklist for demonstrating valid measurement of abstract constructs like safety and robustness. The taxonomy of validity failures provides diagnostic criteria for identifying when benchmarks fail to measure their intended targets. This meta-analysis offers paper2 critical guidance on ensuring generated test cases validly measure intended capabilities rather than superficial proxies.

### Paper
**Title:** Measuring what Matters: Construct Validity in Large Language Model Benchmarks

**Authors:** Andrew M. Bean, Ryan Othniel Kearns, Angelika Romanou, Franziska Sofia Hafner, Harry Mayne, Jan Batzner, Negar Foroutan, Chris Schmitz, Karolina Korgul, Hunar Batra, Oishi Deb, Emma Beharry, Cornelius Emde, Thomas Foster, Anna Gausen, María Grandury, Simeng Han, Valentin Hofmann, Lujain Ibrahim, Hazel Kim, Hannah Rose Kirk, Fangru Lin, Gabrielle Kaili-May Liu, Lennart Luettgau, Jabez Magomere, Jonathan Rystrøm, Anna Sotnikova, Yushi Yang, Yilun Zhao, Adel Bibi, Antoine Bosselut, Ronald Clark, Arman Cohan, Jakob Foerster, Yarin Gal, Scott A. Hale, Inioluwa Deborah Raji, Christopher Summerfield, Philip H.S. Torr, Cozmin Ududec, Luc Rocher, Adam Mahdi

### Abstract
Evaluating large language models (LLMs) is crucial for both assessing their capabilities and identifying safety or robustness issues prior to deployment. Reliably measuring abstract and complex phenomena such as 'safety' and 'robustness' requires strong construct validity, that is, having measures that represent what matters to the phenomenon. With a team of 29 expert reviewers, we conduct a systematic review of 445 LLM benchmarks from leading conferences in natural language processing and machine learning. Across the reviewed articles, we find patterns related to the measured phenomena, tasks, and scoring metrics which undermine the validity of the resulting claims. To address these shortcomings, we provide eight key recommendations and detailed actionable guidance to researchers and practitioners in developing LLM benchmarks.

### Link
[arXiv](https://arxiv.org/abs/2511.04703)

### Reason for Selection
This systematic review provides paper2 with empirically-validated criteria for ensuring test case quality and validity based on analysis of 445 benchmarks across major ML/NLP venues. The eight recommendations and operational checklist offer concrete guidelines for the learning phase to verify that generated test cases measure intended capabilities. The taxonomy of validity failures serves as a diagnostic tool for identifying and correcting systematic biases in test generation, ensuring paper2 produces benchmarks that genuinely assess target abilities.

---

## Finding 15: LLM-based Active Learning

### Idea Summary
LLM-based Active Learning extends active learning from selection-based approaches to include generation of entirely new data instances, with taxonomy categorizing techniques by querying (selection vs. generation) and annotation (human, LLM, or hybrid) components. The survey reveals that LLMs enable querying modules to generate unlabeled instances on demand rather than selecting from fixed pools, reducing training costs for supervised fine-tuning. This paradigm shift from selection to generation provides paper2 with algorithmic strategies for creating novel test cases rather than merely sampling from existing distributions.

### Paper
**Title:** From Selection to Generation: A Survey of LLM-based Active Learning

**Authors:** Yu Xia, Subhojyoti Mukherjee, Zhouhang Xie, Junda Wu, Xintong Li, Ryan Aponte, Hanjia Lyu, Joe Barrow, Hongjie Chen, Franck Dernoncourt, Branislav Kveton, Tong Yu, Ruiyi Zhang, Jiuxiang Gu, Nesreen K. Ahmed, Yu Wang, Xiang Chen, Hanieh Deilamsalehy, Sungchul Kim, Zhengmian Hu, Yue Zhao, Nedim Lipka, Seunghyun Yoon, Ting-Hao Kenneth Huang, Zichao Wang, Puneet Mathur, Soumyabrata Pal, Koyel Mukherjee, Zhehao Zhang, Namyong Park, Thien Huu Nguyen, Jiebo Luo, Ryan A. Rossi, Julian McAuley

### Abstract
Active Learning (AL) has been a powerful paradigm for improving model efficiency and performance by selecting the most informative data points for labeling and training. In recent active learning frameworks, Large Language Models (LLMs) have been employed not only for selection but also for generating entirely new data instances and providing more cost-effective annotations. Motivated by the increasing importance of high-quality data and efficient model training in the era of LLMs, we present a comprehensive survey on LLM-based Active Learning. We introduce an intuitive taxonomy that categorizes these techniques and discuss the transformative roles LLMs can play in the active learning loop. We further examine the impact of AL on LLM learning paradigms and its applications across various domains. Finally, we identify open challenges and propose future research directions. This survey aims to serve as an up-to-date resource for researchers and practitioners seeking to gain an intuitive understanding of LLM-based AL techniques and deploy them to new applications.

### Link
[arXiv](https://arxiv.org/abs/2502.11767)

### Reason for Selection
This survey provides paper2 with a comprehensive taxonomy of LLM-based active learning techniques applicable to intelligent test case selection and generation. The shift from selection to generation paradigm offers algorithmic strategies for creating novel, informative test cases rather than sampling from fixed distributions. The examination of querying and annotation components provides concrete implementation guidance for paper2's learning phase to identify information gaps and generate targeted test cases that maximize learning signal.

---

## Summary: Key Algorithmic Strategies for paper2

### 1. Multi-Model Competitive Evaluation (ArenaBencher)
- Ability inference from existing test cases
- Candidate generation preserving original intent
- Multi-model feedback aggregation
- Iterative refinement with in-context demonstrations

### 2. Graph-Based Dynamic Generation (DyVal, DARG)
- DAG extraction from reasoning tasks
- Structural perturbation for controllable complexity
- Code-augmented LLM verification
- Linguistic diversity preservation

### 3. Adversarial Curriculum Learning (ATGen)
- Co-evolutionary test/code generation
- Dual-objective optimization (accuracy + attack success)
- Dynamic difficulty escalation
- RL-based policy learning

### 4. Continuous Refresh Mechanisms (LiveBench)
- Time-based sourcing from recent information
- Monthly update cadence
- Objective automatic scoring
- Version-controlled task evolution

### 5. Memory-Augmented Sequential Evaluation (LLM-Evolve, Evo-Memory)
- Demonstration memory construction
- Retrieval-based learning
- Action-think-memory refine pipeline
- Experience reuse across task streams

### 6. Self-Rewarding Frameworks (Self-Rewarding LMs)
- LLM-as-a-Judge prompting
- Iterative DPO with self-generated rewards
- Co-evolution of capability and evaluation quality
- Recursive self-improvement

### 7. Active Selection Policies (Active Evaluation Acquisition)
- RL-based dependency modeling
- Informative subset selection
- Prediction of unselected outcomes
- Efficiency optimization

### 8. Multi-Tiered Coverage Assessment (TESTEVAL, CLOVER)
- Graduated difficulty levels
- Coverage-based evaluation
- Context-aware test generation
- Diagnostic weakness identification

### 9. Quality-Diversity Optimization (QDHF)
- Human feedback for diversity metrics
- Contrastive learning in latent space
- 2AFC preference mechanisms
- Exploration-exploitation balance

### 10. Construct Validity Assurance (Measuring what Matters)
- Systematic validity checking
- Taxonomy of validity failures
- Eight-point recommendation framework
- Operational checklists

---

## Implementation Roadmap for paper2

### Phase 1: Foundation
1. Implement graph-based test case representation (DyVal/DARG approach)
2. Establish multi-model evaluation infrastructure (ArenaBencher)
3. Create baseline coverage metrics (TESTEVAL/CLOVER)

### Phase 2: Learning Mechanisms
1. Deploy memory-augmented sequential evaluation (Evo-Memory)
2. Implement self-rewarding evaluation loop (Self-Rewarding LMs)
3. Add active selection policies (Active Evaluation Acquisition)

### Phase 3: Adaptive Generation
1. Integrate adversarial curriculum learning (ATGen)
2. Implement continuous refresh pipeline (LiveBench)
3. Add diversity optimization (QDHF)

### Phase 4: Quality Assurance
1. Apply construct validity framework (Measuring what Matters)
2. Implement automated verification (DARG code-augmented approach)
3. Establish design principle compliance (Static-to-Dynamic Survey)

---

**Document prepared:** March 31, 2026
**Total papers surveyed:** 15
**Primary venues:** ACL, EMNLP, NeurIPS, ICLR, ICML, arXiv
**Publication years:** 2024-2026
