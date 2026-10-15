# Machine Learning 101

## Amazon Comprehend

- NLP service, to uncover insights and connections in text

## Amazon Kendra

- intelligent search service, human-like interaction, can ask questions about a data source (e.g. S3, Confluence, Google Workspace, RDS, etc)

## Amazon Lex

- build and deploy AI chatbots (text/voice) — powers the Alexa service

## Amazon Polly

- text-to-speech (lifelike)
- SSML (Speech Synthesis Markup Language)

## Amazon Rekognition

- deep-learning based image and video analysis
- identify/analyse objects, faces, etc
- e.g. flow:
    - S3 bucket with images → S3 event triggers Lambda → Rekognition identifies animals → metadata stored in DynamoDB

## Amazon Textract

- extract text, handwriting, and data from scanned documents
- also understands and analyses the data (incl forms, tables, etc)

## Amazon Transcribe

- speech recognition, audio-to-text

## Amazon Translate

- text translation service, from a native language to multiple other languages
- encoder (reads source, does semantic representation) → decoder (reads meaning, writes target language)
- auto-detects source language

## Amazon Forecast 101

- forecasting time-series data
- e.g. retail demand, supply chain, staffing, web traffic, etc

## Amazon Fraud Detector

- detection of potential fraud online, e.g. unauthorized transactions, creation of fake accounts, account takeover/phishing/social engineering

## Amazon SageMaker

- fully managed ML service, for data scientists, developers etc
- build, train, deploy ML models
