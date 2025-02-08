# lex-chatbot
Building a self-service digital assistant using Amazon Lex and Amazon Bedrock Knowledge Bases

#Chatbot: A chatbot is an AI-powered software application designed to simulate human-like conversations through text or voice interactions. Chatbots can be used for customer support, information retrieval, automation, and other interactive experiences.

#RAG: Retrieval-Augmented Generation (RAG) is an AI architecture that enhances the capabilities of Large Language Models (LLMs) by combining them with external knowledge sources.
Instead of relying solely on their pre-trained data, RAG systems allow LLMs to access and incorporate relevant information from external sources like databases, documents, or APIs during the generation process.

#Vector Database: A database that stores data as vectors, which are numerical representations of data points. These vectors capture the meaning and relationships within the data, enabling similarity searches and AI-powered applications.

#Vectorization: The process of converting data (text, images, etc.) into vectors. This involves using machine learning models to extract key features and represent them as a list of numbers, capturing the semantic meaning of the data. The cat sat on the mat": [0.8, 0.2, 0.1]

Amazon Lex is an AWS service for building conversational interfaces into applications using voice and text. Amazon Lex provides advanced conversational interfaces using voice and text channels. It features natural language understanding capabilities to recognize more accurate identification of user intent and fulfills the user intent faster. 

![image](https://github.com/user-attachments/assets/04ea7edd-cbd8-421f-9881-9a5c955bc67d)

Amazon Bedrock simplifies the process of developing and scaling generative AI applications powered by large language models (LLMs) and other foundation models (FMs). It offers access to a diverse range of FMs from leading providers such as Anthropic Claude, AI21 Labs, Cohere, and Stability AI, as well as Amazon’s proprietary Amazon Titan models. Additionally, Amazon Bedrock Knowledge Bases empowers you to develop applications that harness the power of Retrieval Augmented Generation (RAG), an approach where retrieving relevant information from data sources enhances the model’s ability to generate contextually appropriate and informed responses.

![image](https://github.com/user-attachments/assets/6489864c-a1e3-40f3-92e8-14517c8a268e)


The generative AI capability of QnAIntent in Amazon Lex lets you securely connect FMs to company data for RAG. QnAIntent provides an interface to use enterprise data and FMs on Amazon Bedrock to generate relevant, accurate, and contextual responses. You can use QnAIntent with new or existing Amazon Lex bots to automate FAQs through text and voice channels, such as Amazon Connect.

With this capability, you no longer need to create variations of intents, sample utterances, slots, and prompts to predict and handle a wide range of FAQs. You can simply connect QnAIntent to company knowledge sources and the bot can immediately handle questions using the allowed content.

In this, we demonstrate how to build chatbots with QnAIntent that connects to a knowledge base in Amazon Bedrock (powered by Amazon OpenSearch Serverless as a vector database) and build rich, self-service, conversational experiences for your customers.

Solution overview
The solution uses Amazon Lex, Amazon Simple Storage Service (Amazon S3), and Amazon Bedrock in the following steps:

1. Users interact with the chatbot through a prebuilt Amazon Lex web UI.
2. Each user request is processed by Amazon Lex to determine user intent through a process called intent recognition.
3. Amazon Lex provides the built-in generative AI feature QnAIntent, which can be directly attached to a knowledge base to fulfill user requests.
4. Amazon Bedrock Knowledge Bases uses the Amazon Titan embeddings model to convert the user query to a vector and queries the knowledge base to find the chunks that are semantically similar to the user query. The user prompt is augmented along with the results returned from the knowledge base as an additional context and sent to the LLM to generate a response.
5. The generated response is returned through QnAIntent and sent back to the user in the chat application through Amazon Lex.
The following diagram illustrates the solution architecture and workflow.

![chatbot](https://github.com/user-attachments/assets/2ecf5122-cd1e-4b29-bec9-e4d38afd4c1b)

In the following sections, we look at the key components of the solution in more detail and the high-level steps to implement the solution:

1. Create a knowledge base in Amazon Bedrock for OpenSearch Serverless.
2. Create an Amazon Lex bot.
3. Create new generative AI-powered intent in Amazon Lex using the built-in QnAIntent and point the knowledge base.

Prerequisites
To implement this solution, you need the following:

An AWS account with privileges to create AWS Identity and Access Management (IAM) roles and policies. For more information, see Overview of access management: Permissions and policies.
Familiarity with AWS services such as Amazon S3, Amazon Lex, Amazon OpenSearch Service, and Amazon Bedrock.
Access enabled for the Amazon Titan Embeddings G1 – Text model and Anthropic Claude 3 Haiku on Amazon Bedrock. For instructions, see Model access.

![image](https://github.com/user-attachments/assets/92811e7d-f5e5-47b0-9d7b-370b12e38714)

A data source in Amazon S3. For this, I've used docker pdf.

![image](https://github.com/user-attachments/assets/454e1c27-c33d-4ace-b1da-a76678986ff2)

Create a knowledge base
To create a new knowledge base in Amazon Bedrock, complete the following steps. For more information, refer to Create a knowledge base.

1. On the Amazon Bedrock console, choose Knowledge bases in the navigation pane.
2. Choose Create knowledge base.
3. On the Provide knowledge base details page, enter a knowledge base name, IAM permissions, and tags.
4. Choose Next.

![image](https://github.com/user-attachments/assets/4f597760-236a-4495-a446-f39975a1ea90)

5. For Data source name, Amazon Bedrock prepopulates the auto-generated data source name; however, you can change it to your requirements.
6. Keep the data source location as the same AWS account and choose Browse S3.

![image](https://github.com/user-attachments/assets/16e73bea-8815-4640-85de-bd43a6b1aff2)

7. Select the S3 bucket where you uploaded the Amazon shareholder documents and choose Choose. This will populate the S3 URI, as shown in the following screenshot.
8. Choose Next.
9. Select the embedding model to vectorize the documents. For this post, we select Titan embedding G1 – Text v1.2.
10. Select Quick create a new vector store to create a default vector store with OpenSearch Serverless.
11. Choose Next.

![image](https://github.com/user-attachments/assets/6e99de1e-0e06-4edf-a60a-9fec5b8ece12)

12. Review the configurations and create your knowledge base. After the knowledge base is successfully created, you should see a knowledge base ID, which you need when creating the Amazon Lex bot.
13. Choose Sync to index the documents.

Create an Amazon Lex bot
Complete the following steps to create your bot:

On the Amazon Lex console, choose Bots in the navigation pane.
Choose Create bot.

For Creation method, select Create a blank bot.
For Bot name, enter a name (for example, FAQBot).

For Runtime role, select Create a new IAM role with basic Amazon Lex permissions to access other services on your behalf.
Configure the remaining settings based on your requirements and choose Next.
On the Add language to bot page, you can choose from different languages supported.
For this post, we choose English (US).
Choose Done.

After the bot is successfully created, you’re redirected to create a new intent.

Add utterances for the new intent and choose Save intent.

Add QnAIntent to your intent
Complete the following steps to add QnAIntent:

On the Amazon Lex console, navigate to the intent you created.
On the Add intent dropdown menu, choose Use built-in intent.

For Built-in intent, choose AMAZON.QnAIntent – GenAI feature.
For Intent name, enter a name (for example, QnABotIntent).
Choose Add.

After you add the QnAIntent, you’re redirected to configure the knowledge base.

For Select model, choose Anthropic and Claude V2.
For Choose a knowledge store, select Knowledge base for Amazon Bedrock and enter your knowledge base ID.
Choose Save intent.

![image](https://github.com/user-attachments/assets/14199655-c7bd-4816-b4f4-00f40bc6a861)

After you save the intent, choose Build to build the bot.
You should see a Successfully built message when the build is complete.

![image](https://github.com/user-attachments/assets/b5082b97-4508-4315-bb26-497c445fb1d2)

You can now test the bot on the Amazon Lex console.
Choose Test to launch a draft version of your bot in a chat window within the console.

Enter questions to get responses.

![image](https://github.com/user-attachments/assets/b3584da5-0449-44bb-acca-7d1bf9349265)

![image](https://github.com/user-attachments/assets/cfcd151f-ab48-42d9-ae87-afb6ecb29859)

![image](https://github.com/user-attachments/assets/a8d53331-fff7-420b-bb2d-e6b05c3265de)

Reference: https://aws.amazon.com/blogs/machine-learning/build-a-self-service-digital-assistant-using-amazon-lex-and-amazon-bedrock-knowledge-bases/

https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html

https://docs.aws.amazon.com/lex/latest/dg/what-is.html


