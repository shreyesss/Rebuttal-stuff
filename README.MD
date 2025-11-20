# Reviewer-1

## Weaknesses

- **W1.** As this is a quickly evolving and crowded space, it's better to compare with more recent baselines, and more importantly, to compare with more recent related research on data synthesis and recent upgrades to **GRPO** (such as **GSPO** which is now widely used in this space).

- **W2.** Relatedly, the components in this work, including the data synthesis and minor improvements on the training part, by far, have been largely explored, which undermines the contribution of this work. The **RAPO** relative to **GRPO** is still minor update per se, and the improvements in Table 3 is not very substantial.

- **W3.** To thoroughly evaluate the "deep research" capabilities of the agent, the paper would be benefited from incorporating commonly used **BrowseComp** benchmark. Some RAG benchmarks such as **SimpleQA** is already relatively too old and too short to be informative in this space.

---

## Questions

- **Q1.** A typo in the Table 2: systems such **Kimi-Researcher** should be closed source agents?

- **Q2.** Will all the models and datasets be open sourced?

- **Q3.** The authors might consider summarizing how **Fathom** differs from other recent research agents (in dataset construction, training paradigm, and report synthesis) to contextualize its contributions within the current surge of DeepResearch systems.

- **Q4.** The same for RL algorithms, such as **GSPO** which is now widely used by search agents.

---

## Responses

### Q1

We thank the reviewer for pointing it out, we apologize for the oversight we will fix it in the manuscript placing Kimi-K2 in the closed-source section.

### Q2

Yes, all the models and datasets and dataset generation code will be open sourced.

---

### W1 & Q4

We appreciate the reviewers' concern about RAPO's comparison with GSPO, and present the follwing explanation. 

**GSPO** is concurrent work: the paper (`arXiv:2507.18071`) was released on 25th July, after our literature review cutoff in July, and thus it was not included in our experiments. We will be happy to mark this clearly in the paper and add **GSPO** to the related-work section.

**GSPO** enhances **GRPO** by replacing token-level importance weighting with sequence-level ratios and clipping, yielding improved stability for multi-step tool-calling, particularly in MoE architectures. **RAPO**, by contrast, targets a distinct failure mode: group-variance collapse (σ→0) that emerges in multi-turn, tool-augmented reasoning. While **RAPO** itself does not induce long-horizon tool-use behavior, its combination with our step-wise steerable reward provides a principled mechanism for expanding and regulating multi-step tool-calling trajectories through cognitively controlled reward shaping. Crucially, **RAPO** is designed to be architecturally lightweight and complementary to all **GRPO** variants, including **GSPO**; however, due to compute constraints, we will be unable to train a full **GSPO+RAPO** system during the rebuttal period.

---

### W2 & Q3

We appreciate the reviewers' concern about the distinguishing of our work in the recent surge of DeepResearch systems. To this regard we put forth the following arguments.

#### 1. Dataset construction

Unlike recent DeepSearch dataset pipelines such as **WebSailor** (Li et al.) and **ASearcher** (Gao et al.), **Fathom** adopts a fundamentally different data-generation design. As shown in Sec. 2.1 and Fig. 4, **DUETQA** enforces (i) true live-web search dependency by ensuring that at least one reasoning hop requires post-2024 evidence, which is beyond the knowledge cutoffs of most frontier models, (ii) deliberate heterogeneous source diversification across non-Wikipedia domains, this design minimizes pre-training overlap and prevents models from short-circuiting multi-hop reasoning.

Further, **DUETQA** is operationally simple: each question is produced using only three model calls (one for generation and two for verification). This contrasts sharply with **ASearcher** and **WebSailor**, which rely on multi-step page traversal and multi-turn retrieval chains to synthesize each question. Notably **WebSailor** has not released its full dataset and data generation code while we intend to fully open source our work.

Finally, **DUETQA** provides explicit thematic via mixture-of-themes mode through a manually curated and editable taxonomy of 200+ themes and question framing style control via seeded question mode, enabling fine-grained steering of question domains and styles. Such steerability is absent in other approaches. 

| Work       | Live-web-search* | Dataset Steerability    | Dataset open source | Code open source |
| ---------- | ---------------- | ----------------------- | ------------------- | ---------------- |
| WebSailor  | No               | No                      | No                  | No               |
| ASearcher  | No               | No                      | Yes                 | Yes              |
| SFR-DR     | No               | No                      | No                  | No               |
| DUETQA     | Yes              | Yes                     | Yes                 | Yes              |

#### 2. Report synthesis

As detailed in Sec. 3.1, **Fathom** introduces, to our knowledge, the first 4B-scale DeepResearch system capable of producing **structured**, **citation-grounded** research reports to **open-ended questions**. Very few open-source models like **SFR-DR** (Nguyen et. eal), **Tongyi-DeepResearch** (Li et. al.) possess open-ended research synthesis capability, and none do so at this small (4B) scale and scope.

Our **DEEPRESEARCH-SFT** dataset is likewise the first of its kind, providing supervision along three orthogonal axes: (i) question decomposition into ordered, semantically coherent sections; (ii) evidence-to-section alignment to ensure citation accuracy; and (iii) insight-planning to guide the synthesis of higher-level claims.

Building on this dataset, the **Fathom-Synthesizer** model can transform any DeepSearch trace into a citation-dense, multi-section report. These design choices yield substantial performance gains over contemporary strong DeepResearch agents such as **Kimi-Researcher** and **Grok-DeeperSearch**.

#### 3. Training paradigm

As mentioned in Sec. 1.2, **Fathom** introduces a two-stage ** RL-Zero ** framework that eliminates the cold-start phase used in all prior DeepResearch agents. Cold-start narrows the solution space of the base model and imprints strong bias in the cognitive behaviors instilled in the student model, causing downstream model behavior to depend heavily on the quality of that cold-start data. In contrast, our RL-Zero pipeline learns long-horizon tool-use directly, without inheriting any behavioral bias from a handcrafted or imperfect cold-start set. The stepwise steerable reward provides explicit, fine-grained control over the model’s evolving reasoning trajectory, allowing control over the acxquired cognitive behaviors such as exploration verififcation from first principles rather than from pre-imposed patterns.

---

### W3

We appreciate the reviewer’s suggestion to evaluate **Fathom** on the most challenging DeepSearch benchmark **BrowseComp**. We originally did not include **BrowseComp** because it cannot be meaningfully evaluated under our current training and context-budget constraints. **BrowseComp** queries typically require **50–100+ tool calls** and **128K–256K** context windows, whereas our model is trained on **Qwen3-4B** with a **40,960-token** window (no RoPE/YARN extension), limiting feasible trajectories to **<30 calls**. Under these conditions, the benchmark becomes a test of context limits rather than the underlying method.

As requested, we now provide a comparison of **Fathom-Search-4B** on **BrowseComp** against models of similar size and scope (see attached figure). **Fathom-Search-4B** **outperforms strong open-source baselines** such as **II-Search-4B**, **WebSailor-3B**, and **Jan-Nano-128K** on **Pass@1**, and its **Pass@5** performance is already **competitive with GPT-oss-20B (high)** despite the significant model-size and context-budget gap. This indicates that our method scales favorably even in a setting for which it was not explicitly optimized.

![(Figure 1): BrowseComp Accuracy comparison](https://github.com/shreyesss/Rebuttal-stuff/blob/main/browsecomp_passk_rebuttal.png)


# Reviewer-2

## Weaknesses

- **W1.** This work lacks a detailed analysis of whether the final trained model can truly work for long-horizon tool calls. When it succeeds and when it fails?  


---

## Questions

- **Q1.** How long does it take to train the model for different stages?  
- **Q2.** This work is claimed to build on top of RECALL (Chen et al., 20). Why not treat it as one of the baselines?

---

## Responses
### Q1

Single node with 8×H100 GPUs (3 days for stage-1, 2 days for stage-2). We plan on integrating modern asynchronous frameworks to speedup training and scale to multiple nodes. 

### Q2

RECALL is a RAG-style framework whose retriever operates over a static, pre-2024 Wikipedia index. Our experiments are explicitly designed to enforce live-web–search dependence and require post-2024 information to avoid contamination from model pretraining and to guarantee true search necessity. Because RECALL cannot retrieve beyond its fixed Wikipedia snapshot and therefore cannot satisfy the temporal or source-diversity constraints central to our evaluation, it is not an appropriate baseline for our setting.

---

### W1

  We thank the reviewer for raising this point. Our model does perform substantive long-horizon tool-based reasoning, and we provide both empirical evidence for the same and a principled task taxonomy to clarify when it succeeds and when it fails.

1. **Demonstrated long-horizon capability**

   Despite using a **40,960-token** context window (**Qwen3-4B**, without RoPE/YARN due to resource limits), our system can reliably executes **~30 sequential tool calls per trajectory**. This is already significantly stronger than the best open-source baselines for ≤4B model size, like **II-Search**, **Jan-Nano**, and others, despite their larger (**128K**) context lengths. Please refer to Fig. 1 in the manuscript for exact comparison. 

2. **Evidence of efficient long-horizon reasoning (Figs. 1–3)**

   - **Fig. 1 (GRPO vs RAPO):** Tool call and accuracy evolution on the validation set over stage-1 training. **GRPO** shows rising tool use without accuracy gains (inefficient exploration), whereas **RAPO** achieves higher accuracy with fewer calls, indicating productive long-horizon reasoning.
   ![(Figure 1): GRPO vs RAPO (Accuracy and Tool Call evolution)](https://github.com/shreyesss/Rebuttal-stuff/blob/main/stage_1_final.png)


   - **Fig. 2 (Steerable Reward vs Vanilla):** Tool call and accuracy evolution on the validation set over stage-2 training. With **Steerable reward**, accuracy increases monotonically with tool-call depth, indicating our Steerable reward leads to productive exploration, where increasing number of tool calls translate to higher accuracy, a signature of genuine long-horizon scaling absent in the vanilla reward.
    ![(Figure 2): Vanilla vs Steerable Reward (Accuracy and Tool Call evolution)](https://github.com/shreyesss/Rebuttal-stuff/blob/main/stage_2_final.png)

   - **Fig. 3 (Test-Time Scaling vs WebSailor-3B):** Test-time scaling plot for **Fathom-Search-4B’s** GRPO-trained checkpoint, RAPO-trained checkpoint, and **WebSailor-3B** (Li et. al.). It clearly demonstrates that **WebSailor-3B** and **GRPO** saturate early (~15 calls) as their performance does not increase with further tool calling , while our models continue improving as the call budget increases upto **30 tool calls**, demonstrating superior scaling under long-horizon budget.
    ![(Figure 3): Test-Time-Scaling](https://github.com/shreyesss/Rebuttal-stuff/blob/main/stage_2_final.png)

3. **Failure Modes**

   To contextualize when long-horizon tool use is required and when models fail, we follow the task-uncertainty taxonomy introduced by **WebSailor** (Li et. al.).

   - **Level-1 tasks** exhibit low uncertainty and can be solved using internal knowledge or 1–2 searches.

   - **Level-2 tasks** (e.g., multi-hop QA) begin with higher uncertainty but follow a deterministic, well-structured reasoning path in which uncertainty monotonically contracts as the model links successive entities; these questions can be solved with **≤25 searches**.

   - **Level-3 DeepSearch tasks:** the most challenging category combine high uncertainty at each step with no predefined reasoning path. Several intermediate hypotheses remain viable for many hops, and only cross-verification across distant, independent clues collapses the ambiguity to a correct answer. These questions require **up to 100 searches**.

   Our present stem not from the method but from the training regime and context budget. We train **Qwen3-4B** at a **40,960-token** context window (no RoPE/YARN extension) due to compute constraints, which caps feasible trajectories at **<30 tool calls**. **Level-3 DeepSearch tasks** (e.g., **BrowseComp**), however, routinely require **50–100+ calls** and **128K–256K** context, making them impossible to model faithfully under our current budget.

   Equally important, Level-3 problems require fuzzy, high-entropy training data, where multiple intermediate hypotheses remain viable for many hops and uncertainty is intentionally preserved rather than collapsing early. While **WebSailor** reports using such ambiguity-preserving supervision, their dataset and generation pipeline are not open-sourced, so it is unclear how these high-uncertainty questions are produced for training. In contrast, we open-source our entire data-generation pipeline and will extend it in the next iteration to explicitly produce Level-3–style, multi-hypothesis, long-horizon training data once we can train with a **128K-context window**.

# Reviewer-3

## Weaknesses

- **W1.** The paper points out that GRPO suffers from reward-hacking behaviors, where the model tends to overuse tools without improving reasoning quality. However, it does not clearly demonstrate that the proposed RAPO algorithm effectively mitigates this issue from the perspective of actual tool-call frequency. In Figure 4, only the response length is presented as a proxy for the number of tool calls, which is an indirect measure. A more concrete quantitative analysis or visualization of tool-call usage would strengthen the claim that RAPO truly alleviates reward-hacking behavior.  

- **W2.** More challenging benchmarks, such as BrowseComp, are expected to be included to better demonstrate the model’s deep research capabilities and generalization to new, open-ended evaluation settings.

---

## Questions

- **Q1.** What's the meaning of "SLMs"? The title mentions SLMs, but the paper does not define this abbreviation.

---

## Responses

### Q1

We apologize for not making it clear in the manuscript. **SLMs** stand for **Small Language Models** and is used to refer to models with parameter range in the low billions or millions. 

---

### W1

We thank the reviewer for raising this point. Here we provide 2 additional plots that compare the evolution of accuracy and tool-call depth over the course of stage-1 training. This serves as a direct measure of the effectiveness of our test-time scaling methods and alleviations from “reward-hacking” behavior. 

**Fig. 1 (GRPO vs RAPO):** Tool call and accuracy evolution on the validation set over stage-1 training, GRPO shows rising tool use without accuracy gains (inefficient exploration), whereas RAPO achieves higher accuracy with fewer calls, indicating productive long-horizon reasoning.

![Fig. 1 – GRPO vs RAPO: Tool Calls and Accuracy Evolution](https://github.com/shreyesss/Rebuttal-stuff/blob/main/stage_1_final.png)

**Fig. 2 (Steerable Reward vs Vanilla):** Tool call and accuracy evolution on the validation set over stage-2 training. With Steerable reward, accuracy increases monotonically with tool-call depth, indicating that our Steerable reward leads to productive exploration, where increasing number of tool calls translate to higher accuracy — a signature of genuine long-horizon scaling absent in the vanilla reward.

![Fig. 2 – Steerable Reward vs Vanilla: Tool Calls and Accuracy Evolution](https://github.com/shreyesss/Rebuttal-stuff/blob/main/stage_2_final.png)

---

### W2

We appreciate the reviewer’s suggestion to evaluate **Fathom** on the most challenging DeepSearch benchmark **BrowseComp**. We originally did not include **BrowseComp** because it cannot be meaningfully evaluated under our current training and context-budget constraints. **BrowseComp** queries typically require **50–100+ tool calls** and **128K–256K** context windows, whereas our model is trained on **Qwen3-4B** with a **40,960-token** window (no RoPE/YARN extension), limiting feasible trajectories to **<30 calls**. Under these conditions, the benchmark becomes a test of context limits rather than the underlying method.

As requested, we now provide a comparison of **Fathom-Search-4B** on **BrowseComp** against models of similar size and scope (see attached figure). **Fathom-Search-4B** **outperforms strong open-source baselines** such as **II-Search-4B**, **WebSailor-3B**, and **Jan-Nano-128K** on **Pass@1**, and its **Pass@5** performance is already **competitive with GPT-oss-20B (high)** despite the significant model-size and context-budget gap. This indicates that our method scales favorably even in a setting for which it was not explicitly optimized.

![(Figure 1): BrowseComp Accuracy comparison](https://github.com/shreyesss/Rebuttal-stuff/blob/main/browsecomp_passk_rebuttal.png)


