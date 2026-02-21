# Azure AI 102

- The Azure Computer Vision `Read API` is a powerful tool for extracting text from images and documents, including PDFs and TIFFs. It supports both printed and handwritten text in multiple languages and provides asynchronous processing for efficient text recognition.

## speech

- Speech SDK: `SpeechREcognitionLanguage` for source and `AddTargetLanguage()` for target. `AudioStreamFormat` to define format being streamed.
- To enable `speech capabilities` in a chatbot, you must create a Speech service to process audio input and output, register the Direct Line Speech channel (which connects the bot to speech clients), and enable WebSockets (required for real-time audio communication).

## Azure Cognitive Service for Language

- `chit-chat` adds predefined conversational response for casual dialogue.
- `create entity` helps extracting structured data from user input, but does not imporve the model's ability to match different phraseing.
- Adding `alternative prhasing` to existing q/a helps the model recogonize different ways user may ask the same question, which improves coverage.
- `Azure Spatial Analysis` in the Azure AI Vision suite enables real-time understanding of `people`’s movements in physical spaces using video feeds. It is designed for scenarios like people counting, zone occupancy, and movement tracking, without requiring prior machine learning expertise.

## azure ai document intelligence

- file size limit: s0 -> 500MB, 200 pages. resolution: 50*50 - 10K*10K.
- Azure AI video indexer
  - [file size: 2GB, length: 6 hours](https://learn.microsoft.com/en-us/azure/azure-video-indexer/avi-support-matrix).

## Azure open ai

- fine tune data: `jsonl` format

- Azure AI language service, custom qusetion answering -> [import file type](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/reference/document-format-guidelines): `.tsv, .xls, .txt`.
- `Azure AI Language` provides powerful tools for natural language processing (NLP) tasks such as text analysis, conversational AI, and document processing. Python developers can leverage Azure's SDKs and APIs to integrate these capabilities into their applications. `pip install azure-ai-language-conversations`
- Enabling a `service endpoint` for Account in VNet allows secure communication over the Azure backbone network, eliminating the need for public internet access.

## Content Safety Studio

- A metaprompt is a higher-level instruction given to an AI model that guides it on how to generate, refine, or interpret other prompts, effectively acting as a blueprint for creating tailored AI responses. Safety metaprompt feature is to guid the behavior of generative models by encouraging safer outputs.
- Full Scale (0–7) – Used by the text and multimodal (image + text) models. 0–1 → 0 (Safe) 2–3 → 2 (Low) 4–5 → 4 (Medium) 6–7 → 6 (High)

- Protected material detection is specially desinged to detect copyrighted or sensiteve material.

- Azure AI `Metrics Advisor` is an AI-powered service designed to monitor metrics and diagnose issues in near-real time. It is built on the AI Anomaly Detector and is part of Azure AI Services. This service helps organizations stay ahead of incidents by embedding AI-powered monitoring features without requiring machine learning expertise.

## Content moderator

### text

- Category1 refers to the sexually explicit or adult in certain situations.
- Category2 refers to the sexually suggestive or mature in certain situations.
- Category3 refers to the offensive in certain situations.

## face detection API

- To validate real person in live video, checking for changes in the `HeadPose` attribute over time helps confirm natural movement, indicating a live persence ratherthan a static image. Thsi is a common method for liveness detectin.

- Azure AI's `groundedness detection` supports two primary tasks: Summarization and QnA. Users can select between Reasoning Mode for detailed explanations or Non-Reasoning Mode for faster detection. Additionally, the API includes a correction feature that automatically aligns ungrounded outputs with the provided sources
