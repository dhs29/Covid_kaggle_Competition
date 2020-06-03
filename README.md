# CORD-19 submission: Closed Domain Question & Answer (CDQA) Search & Summarize (CDQASS) on COVID-19 Literature

By Team Die Corona

**Content:**

1. Approach
    * Introduction
    * Current Pipeline
2. Approach Implementation
    * CDQA
    * Pipeline Preparation
3. Task Results
    * Sub-Task Examples (Top Results, Summary & Wordcloud)
    * Discussion of Results
    * Future Work
4. References
5. Try it out!


# 1. Approach

## Introduction: Closed Domain Question & Answer (CDQA) Search & Summarize (CDQASS) on COVID-19 Literature


The CDQA based COVID-19 Search & Summarize implemented here is based on a retriever-reader dual algorithmic approach developed by André Macedo Farias et al [1][2]. The CDQA model is inspired by the DrQA Open Domain Question Answer model, developed by Chen et al [3][4]. The main challenge with Question and Answering as a Natural Language Understanding task is that QA models often fail to perform when asked to produce an answer for a question from a large input text. To address this challenge the ODQA model was broken into two steps: 1) first, narrow down the input text to the top articles where the answer might be present (The Retriever) using search (e.g., tf-idf, BM25), and 2) out of the narrowed down input text, find the best potential answer (the Reader) using a Q&A model. For this reason, ODQA and its CDQA cousin can be considered as an approach to providing "Machine Reading at Scale." In addition to CDQA, we also summarized the top answers via an abstractive summarizer and a WordCloud visualization to aid the user with a first glance of the results.


In the retriever-reader dual model approach from CDQA (Figure 1, block 1.a), the user starts with a question similar to how one would ask a question in a web search engine. The retriever model then applies a TF-IDF type search, whereby the question string is compared to all of the article abstracts in our closed domain. For our closed domain, we included the 40,000+ article abstracts as our corpus. The top articles are identified based on a cosine similarity between the question string and abstract text.

As noted above, we limited the retriever data search to the abstracts as opposed to full articles. We chose this approach for two main reasons: 1) from a technical standpoint, feeding a large corpus for the closed domain resulted in memory crashes in our kaggle kernel which prevented us from using more than just the abstracts; 2) from a content and problem standpoint, we argue that abstracts will contain the most immediately relevant information a medical professional, researcher or policy maker will require as a starting point in their inquiry. We accepted this as a limitation to our corpus definition and recognize that we might leave out in some instances some very relevant information only found in the full article text; we will consider this as a future avenue of improvement.

Once we have identified the top articles that best address the question using a TF-IDF like ranking from the retriever model, we then proceed to the reader model (Figure 1, block 1.b). In this step, we again provide the input question as well as the abstract corpus. The pre-trained QA model (a DistilBERT implementation available from the Hugginface NLP library [5]) is then reading each selected abstract selected from the retriever in the first step, and providing an answer to the same question for each paragraph. Although it is possible to finetune the reader model by providing an annoted corpus, we selected the pretrained model for our implementation. Our proposal for future work would be to consider our tool being used by the wider user community, and implement a seamless annotation feature (available from the CDQA library) which would allow us to finetune our Reader and develop higher quality results.

Once the retriever and reader models are used, our solution presents the top k answers based on a weighted score between the retriever score (based on TF-IDF cosine similarity) and reader score (based on DistilBERT QA Q-A pair probability) (Figure 1, block 1.c). The standard CDQA implementation suggests a retriever score weight of 0.35 based on testing on the SQUAD v1.1 dataset, however we found that a higher weight of 0.5 resulted in answers containing text more relevant to the initial question.

In a set of downstream tasks, we've taken the output answers and related abstracts and further processed them into a final summary and related visual Wordcloud of the results. To aid the summarizer task, in a first step we have taken all of the abstracts and extracted the relevant sentences where predicted answers from the top-k results are contained (Figure 1, block 2.a). Afterwards, the relevant sentences are fed to the summarizer as a paragraph, out of which an abstractive summary is generated (a BART implementation from Lewis et al[6] based on the Hugginface NLP library [5]) (Figure 1, block 2.b). The motivation behind the summarizer pipeline is to aid the user in further sorting through the answers to get a high level view of the results and what could potentially be the key takeaways of the CDQA results. At the same time, we recognize the limitations of abstractive summarizers: even state of the art Natural Language summarizers are still evaluated on metrics (e.g., ROUGE-X, which measures n-gram overlap against annoted summaries) where there is no universal acceptance--especially as summary annotation is understandibly one of the most difficult human tasks in this space. In other words, if two human beings read the same text and are asked to produce a summary of the text, would they produce exactly the same summary? Although summarization (whether abstractive or extractive) faces inherent limitations, we still  believe it is a useful tool in providing the user with a first glance of the provided answers.
