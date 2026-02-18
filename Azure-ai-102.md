# Azure AI 102

- The Azure Computer Vision `Read API` is a powerful tool for extracting text from images and documents, including PDFs and TIFFs. It supports both printed and handwritten text in multiple languages and provides asynchronous processing for efficient text recognition.
- Speech SDK: `SpeechREcognitionLanguage` for source and `AddTargetLanguage()` for target. `AudioStreamFormat` to define format being streamed.
- Azure Cognitive Service for Language
  - `chit-chat` adds predefined conversational response for casual dialogue.
  - `create entity` helps extracting structured data from user input, but does not imporve the model's ability to match different phraseing.
  - Adding `alternative prhasing` to existing q/a helps the model recogonize different ways user may ask the same question, which improves coverage.
- `Azure Spatial Analysis` in the Azure AI Vision suite enables real-time understanding of `people`’s movements in physical spaces using video feeds. It is designed for scenarios like people counting, zone occupancy, and movement tracking, without requiring prior machine learning expertise.
- azure ai document intelligence
  - file size limit: s0 -> 500MB, 200 pages.
- Azure AI video indexer
  - [file size: 2GB, length: 6 hours](https://learn.microsoft.com/en-us/azure/azure-video-indexer/avi-support-matrix).
- Azure open ai
  - fine tune data: `jsonl` format
- Azure AI language service, custom qusetion answering -> [import file type](https://learn.microsoft.com/en-us/azure/ai-services/language-service/question-answering/reference/document-format-guidelines): `.tsv, .xls, .txt`.
- `Azure AI Language` provides powerful tools for natural language processing (NLP) tasks such as text analysis, conversational AI, and document processing. Python developers can leverage Azure's SDKs and APIs to integrate these capabilities into their applications. `pip install azure-ai-language-conversations`
- Enabling a `service endpoint` for Account in VNet allows secure communication over the Azure backbone network, eliminating the need for public internet access.
