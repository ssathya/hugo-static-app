+++
date = '2025-03-23T16:55:18-04:00'
draft = true
title = 'Semantic Kernel with RAG'
+++

# Semantic Kernel with RAG revisited!

## What is RAG, and why do we need it?

Semantic Kernel finds data in a model, and these models have the underlying data that is presented to the user. But how about your own data? For example, you might have a database of your eating habits. You get on your weigh scale and see that there has been a significant swing in your weight during the last week. A possible interaction could be if you could ask your chat application, which is integrated with the Semantic Kernel and RAG, what I had during the past week. Wouldn't it be wonderful if it would give you the answer? And the answer is yes, it can, and we'll work on an example that does something similar. 

We will build an application that uses Semantic Kernel. It loads your own data (e.g., a PDF, text, or other file) and performs natural language Q&A over it using these tools.

Before we roll up our sleeves, let us first answer the question: What is RAG? RAG stands for Retriever Augmented Generation. RAG does the following:

* **Retriever:** A system/component to fetch relevant documents or data from an extensive collection (e.g., expense records).
* **A Language Model:** A large language model (LLM) that can generate natural language responses based on prompts and retrieved context.

When you ask a question, RAG retrieves relevant snippets or documents from your dataset and uses them to group the LLM's response. This helps the AI give more accurate context-aware answers by focusing on the information you provide. 

## Use Cases for RAG
* Customer Support: RAG can improve chatbots by retrieving relevant information from knowledge bases, FAQs, or customer records to provide accurate and contextually relevant responses.
* Healthcare Applications: In medical chatbots, RAG can retrieve information from electronic health records and medical databases to assist in clinical decision-making.
* E-commerce Support: RAG-powered chatbots can retrieve data from store inventories and order histories to help customers with product recommendations, order tracking, and troubleshooting.
* News
* Personal Knowledge Management: By pulling user-specific data, RAG can generate tailored responses, making interactions more relevant and engaging.

### NEWS
This is my personal interest, so I am pointing out some interesting facts.

Retrieval-augmented generation (RAG) can fetch the latest news. By integrating RAG with reliable news APIs or databases, a chat application can retrieve up-to-date information and generate contextually relevant summaries or responses.

Some news sources do offer free access to their content through APIs, though there are often limitations on usage. Here are a few options for free news APIs:
1. **Newsdata.io:** This API provides access to news articles from over 80,000 sources worldwide. It allows filtering by keywords, countries, languages, and categories. The free tier offers limited API calls.
1. **NewsAPI.org:** While its free tier has been deprecated, it previously allowed access to top headlines and articles from various sources. You might still find limited free access depending on your use case.
1. **GNews API:** Offers free access to global news articles with filtering options for language, country, and topic. It’s a good choice for lightweight applications.
1. **The Guardian API:** Provides free access to its content, categorized by tags and sections. It’s a great option for developers looking for high-quality journalism.
1. **Currents API:** This API aggregates news from various sources, blogs, and forums. It offers a free tier with limited API calls.

Look at [Public APIs developer's portal](https://publicapis.dev/category/news) or [this GitHub page](https://github.com/free-news-api/news-api).
