# Portfolio

Selected projects in applied AI, data engineering and machine learning. Each entry follows the same structure: the **problem and motivation**, **how it was solved**, and a short **summary** of the result and the stack.

| Area | Projects |
| --- | --- |
| Software Engineering and AI Applications | [AI CFO: private personal finance ledger with an AI assistant](#ai-cfo-a-private-local-first-personal-finance-ledger-with-an-ai-assistant) |
| Generative AI and LLMs | [Food Label Reader](#food-label-reader-app) · [Medical Chatbot (Llama 2 + RAG)](#medical-chatbot-using-large-language-models-llama-2) |
| Data Engineering | [Premier League Table Pipeline](#premier-league-table-scraper-with-airflow-and-google-bigquery) |
| Machine Learning | [Predictive Maintenance](#predictive-maintenance-using-machine-learning-based-anomaly-detection) |
| Computer Vision | [Diffusion SDE package](#diffusion-sde-a-score-based-generative-modelling-package) · [Paired image translation](#paired-image-to-image-translation-using-diffusion-models) · [CycleGAN for retinal images](#cyclegans-for-medical-image-translation) · [Unpaired translation (Master's thesis)](#unpaired-image-to-image-translation-masters-project) |
| Research | [Publications](#publications) |

---

## Software Engineering and AI Applications

### AI CFO: a private, local-first personal finance ledger with an AI assistant

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/home-finance-assistant)
[![Documentation](https://img.shields.io/badge/Docs-Architecture_and_User_Guide-informational)](https://github.com/Ishan-phys/home-finance-assistant/tree/main/docs)

<br>
<center><img src="images/aicfo-overview.png?raw=true" alt="AI CFO overview page showing net worth, money in and out for the month, safe to spend, and the inbox" width="90%"/></center>
<center><sub>The Overview page. All names and amounts in these screenshots come from the app's built-in synthetic sample data.</sub></center>
<br>

**Problem and motivation.**
Most personal finance apps either ask for your bank login or ask you to trust a language model with your numbers. Neither is acceptable for sensitive financial data, and an AI-generated total cannot be audited. A household with several Indian bank accounts, credit cards and investment accounts also needs the figures to reconcile: net worth, spending and savings should agree with each other on every page. I built AI CFO to keep the data on my own machine, produce numbers that always add up, and still let me ask questions about my money in plain English.

**How it was solved.**

1. **A double-entry ledger as the source of truth.** Every transaction moves money between two accounts and must sum to zero; PostgreSQL itself rejects anything that does not. Credit card bill payments and moves between my own accounts are paired as transfers, so they never count as spending or income. Money is stored as integer paise, with no floating point anywhere.
2. **A statement ingestion pipeline.** An uploaded file is unlocked, read, redacted, parsed, categorized and journaled in one pass, entirely in memory. Per-bank adapters handle HDFC (account and credit card), SBI, Axis credit card and Groww (mutual funds and stocks) in XLS, XLSX, CSV and PDF, so adding a bank is a small, bounded change.
3. **Privacy by design.** Names, phone numbers, account numbers and UPI handles are replaced with tokens before anything is stored or sent to a model, and a failed redaction rejects the upload. Original narrations are encrypted at rest, the app listens on `127.0.0.1` only, and access requires a single-owner sign-in.
4. **Categorization the owner controls.** Transactions are categorized by the owner's own rules first, then by choices made previously for the same description, and only then by a language model. Anything the model cannot place stays visibly *Uncategorized*.
5. **Figures computed by code, explained by a model.** Net worth, income and expenses, savings and "safe to spend" come from pure functions over one set of postings, and each month carries a check line proving that the change in net worth equals what was saved. The chat assistant (Google ADK over LiteLLM, with Claude or OpenAI) answers by calling read-only tools that return those computed figures, so the model explains the numbers but never calculates them. Account names are masked before they reach the model, and replies are stripped of links and images to block data exfiltration through hidden instructions in a statement.
6. **Engineering discipline.** The backend is a `uv` workspace of small single-purpose Python packages whose allowed imports are enforced in CI. The project has pytest suites (using throwaway PostgreSQL containers), Playwright end-to-end tests, Ruff and strict mypy, plus CodeQL, dependency review and secret scanning on every change.

**Summary.**
AI CFO turns downloaded bank and card statements into a reconciled set of books and an assistant that answers questions such as "how much can I safely spend this month?" using figures it looked up rather than invented. It is a complete, documented product (user guide, architecture and security notes, HTTP API reference) that runs locally with a single `docker compose up`.

**Stack:** Python 3.13, FastAPI, PostgreSQL 16, SQLAlchemy 2, Alembic, Next.js 16, React 19, Tailwind CSS, Google ADK, LiteLLM, Docker Compose, pytest, Playwright, GitHub Actions.

<br>
<center><img src="images/aicfo-spending.png?raw=true" alt="AI CFO spending page showing net in, net out and saved with a category breakdown" width="90%"/></center>
<center><sub>Spending: where the month's money came from and where it went.</sub></center>
<br>
<center><img src="images/aicfo-assistant.png?raw=true" alt="AI CFO assistant answering which credit card had the most spending in a month" width="90%"/></center>
<center><sub>The assistant answers from looked-up figures and shows which lookups it used.</sub></center>
<br>

---

## Generative AI and LLMs

### Food Label Reader App

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/food-labels-gemini)

**Problem and motivation.**
Nutrition labels are dense, vary in layout, and say little about whether a product suits a particular person. Reading ingredients, allergens and nutrient values and comparing them with recommended intakes is slow to do by hand. This project explores whether a multimodal LLM can read a photographed label and put it in context for the person buying the product.

**How it was solved.**

1. A user uploads a photo of a label and selects an age group and gender.
2. Google Gemini's vision model extracts the ingredients, nutritional content and allergens from the image into structured text.
3. Reference documents on recommended nutritional intakes were embedded with Google's embedding model and stored in a Chroma vector database.
4. A Retrieval-Augmented Generation (RAG) step retrieves the nutritional goals for the selected demographic and prompts Gemini to compare the label against them, so the answer is grounded in the reference documents.
5. The pipeline is served through a FastAPI backend with a simple web front end.

**Summary.**
The app turns a photo of a food label into an interpretation tailored to the user's age group and gender, with the retrieval step keeping answers tied to reference documents rather than the model's memory.

**Stack:** Python, Google Gemini (text and vision), LangChain, Chroma, FastAPI.

<br>
<center><img src="images/uploaded_file.jpg?raw=true" alt="Food Label Reader app interface"/></center>
<br>

---

### Medical Chatbot using Large Language Models (Llama 2)

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/llms-project.git)

**Problem and motivation.**
General-purpose chatbots answer medical questions fluently but without grounding in a trusted source, which is a poor fit for a domain where accuracy matters. I wanted to build a question-answering assistant that answers from a specific medical encyclopedia and runs its language model locally, without sending queries to a third-party model API.

**How it was solved.**

1. The medical encyclopedia (PDF) was loaded and split into overlapping chunks of about 400 characters.
2. Each chunk was converted into an embedding with a Sentence-Transformers model (`all-MiniLM-L6-v2`) and indexed in the Pinecone vector database.
3. At query time, the most relevant chunks are retrieved and passed, together with the question, to a quantized Llama-2-7B-chat model running locally through LangChain and Hugging Face tooling.
4. A Flask web application provides the chat interface.

**Summary.**
The result is an end-to-end retrieval-augmented medical assistant: the answers draw on the indexed encyclopedia, and the language model runs on local hardware.

**Stack:** Python, Llama-2-7B (quantized), LangChain, Hugging Face, Pinecone, Flask.

<br>
<center><img src="images/chatbot.png?raw=true" alt="Medical chatbot web interface"/></center>
<br>

---

## Data Engineering

### Premier League Table Scraper with Airflow and Google BigQuery

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/league-table-scraper.git)

**Problem and motivation.**
A league table is only a snapshot. To analyse how a season develops, each week's standings have to be captured and stored in a queryable form. This project automates that collection as a small but complete data pipeline, and it also served as a practical exercise in orchestration and infrastructure as code.

**How it was solved.**

1. An Apache Airflow DAG scrapes the Premier League table on a weekly schedule.
2. The scraped standings are loaded into Google BigQuery for storage and analysis.
3. The BigQuery resources on Google Cloud Platform are provisioned with Terraform, so the infrastructure is reproducible.
4. The Airflow tasks run inside a Docker container started from a customised `docker-compose` file, which makes the setup portable.

**Summary.**
A scheduled, containerised pipeline that builds a historical record of league standings in BigQuery, with infrastructure defined in code.

**Stack:** Python, Apache Airflow, Google BigQuery, Terraform, Docker.

<br>
<center><img src="images/bpl.png?raw=true" alt="Premier League table pipeline"/></center>
<br>

---

## Machine Learning

### Predictive Maintenance using Machine Learning based Anomaly Detection

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/predictive-maintenance-ml)
[![View Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://pm-app-mjx2m72y9ajpuzsdeexmnm.streamlit.app/)

**Problem and motivation.**
Unplanned failures of rotating machinery such as bearings are costly, and fixed maintenance schedules either replace healthy parts or miss developing faults. Run-to-failure sensor data rarely comes with labelled faults, so detection has to work without labels. This project builds a complete predictive maintenance system, from raw vibration data to a monitoring dashboard.

**How it was solved.**

1. **Data and features.** The system uses the IMS Bearing dataset from the Center for Intelligent Maintenance Systems. Vibration signals are converted into condition indicators combining time-domain statistics (such as kurtosis and skewness) with frequency-domain features from windowed FFT spectra.
2. **Model.** An Isolation Forest, with tuned hyperparameters, learns normal bearing behaviour and flags departures from it as anomalies, so no fault labels are required.
3. **Pipeline.** Ingestion, validation, transformation, training and prediction are separate pipeline components, and the prediction service is containerised with Docker.
4. **Storage and interface.** Predictions are stored in MongoDB and shown in a Streamlit application with the time waveform, FFT spectrum and anomaly status for each bearing. The app is deployed to the cloud.

**Summary.**
A three-tier application (Streamlit front end, ML service, MongoDB) that detects anomalies in bearing vibration data and presents them for monitoring.

**Stack:** Python, scikit-learn, SciPy, MongoDB, Streamlit, Plotly, Docker.

<br>
<center><img src="images/streamlit.png?raw=true" alt="Predictive maintenance Streamlit dashboard"/></center>
<br>

---

## Computer Vision

### Diffusion SDE: a score-based generative modelling package

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/Diffusion-SDE)

**Problem and motivation.**
Score-based generative models built on stochastic differential equations (SDEs) can synthesise high-quality images and avoid the training instability of GANs, but using them usually means adapting research code. I wanted a small, reusable Python package that lets someone train such a model on their own image dataset and generate new samples without rewriting the training loop.

**How it was solved.**

1. The package wraps dataset loading, a score network, SDE-based noising, training and sampling behind a small API: a dataset class, a `diffSDE` class with `.train()` and `.generate_samples()`.
2. Training teaches the network to estimate the score of progressively noised data; sampling then reverses the noising SDE step by step, turning random noise into an image. About 1,500 to 2,000 sampling steps give high-quality samples.
3. The package is published on PyPI and supports Python 3.7 onwards. A Google Colab notebook demonstrates generation from a pretrained model.

**Summary.**
Installing it with `pip install diffusion_sde` and pointing it at an image folder is enough to train a diffusion model and generate new images.

**Stack:** Python, PyTorch, PyPI packaging.

<br>
<center><img src="images/header.png?raw=true" alt="Diffusion SDE generation from noise"/></center>
<br>

---

### Paired Image to Image Translation using Diffusion Models

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/paired-via-sde)

**Problem and motivation.**
Many imaging tasks need to convert one kind of image into another, such as satellite photographs into map-style renderings. GAN-based methods such as pix2pix dominate this task, so I wanted to test whether a score-based diffusion model could do it with high fidelity to the ground truth.

**How it was solved.**

1. A score-based generative model was trained in PyTorch on the Maps dataset, which pairs satellite images (domain A) with map images (domain B).
2. At inference, a conditional sampler generates an image in one domain given the paired image from the other, in both directions.
3. Quality was measured with the Structural Similarity Index Measure (SSIM) against the real target images, evaluated as a function of the number of sampling steps.

**Summary.**
With 1,500 sampling steps the generated images reached an SSIM of 0.977 against the ground truth, showing that conditional diffusion sampling reproduces the target domain closely.

**Stack:** Python, PyTorch, score-based SDE models.

<br>
<center><img src="images/paired.png?raw=true" alt="Satellite to map translation results"/></center>
<br>

---

### CycleGANs for Medical Image Translation

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/CycleGANs)

**Problem and motivation.**
Diabetic retinopathy screening produces large numbers of retinal images, and the vessel annotations used to analyse them are drawn by highly trained specialists, which is slow and expensive. Automatically producing a vasculature annotation from a retinal image could speed up that work.

**How it was solved.**

1. A CycleGAN was implemented for unpaired image-to-image translation between domain A (retinal images) and domain B (vasculature annotations).
2. Cycle-consistency loss lets the model learn the mapping without requiring pixel-aligned image pairs.
3. The model was trained on the Digital Retinal Images for Vessel Extraction (DRIVE) dataset, with configuration in a single file and a notebook version for experimentation.

**Summary.**
The trained model generates vessel-annotation images directly from retinal photographs, demonstrating automated annotation as an alternative to manual labelling.

**Stack:** Python, PyTorch, CycleGAN.

<br>
<center><img src="images/cyc_gan.png?raw=true" alt="CycleGAN retinal vessel generation results"/></center>
<br>

---

### Unpaired Image-to-Image Translation (Master's Project)

[![View on GitHub](https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub)](https://github.com/Ishan-phys/Unpaired-via-SDE)

**Problem and motivation.**
Paired training data is often unavailable, so unpaired translation, where no image in one domain has a matching partner in the other, is the more general and harder problem. CycleGAN is the standard approach. My Master's thesis asked whether score-based generative models could be adapted to the unpaired setting.

**How it was solved.**

1. I extended the score-based SDE framework from the paired project to unpaired translation between two image domains.
2. The model was trained on the horse2zebra benchmark and compared directly with CycleGAN outputs on the same inputs.
3. The thesis report and code are in the repository.

**Summary.**
In the comparison, the diffusion-based approach did not yet match CycleGAN on unpaired translation. The repository documents this and identifies improving it as future work.

**Stack:** Python, PyTorch, score-based SDE models.

<br>
<center><img src="images/thesis.png?raw=true" alt="Unpaired translation results compared with CycleGAN"/></center>
<br>

---

## Publications

- [Constraining the weights of Stokes polytopes using BCFW recursions for phi^4](https://link.springer.com/article/10.1007/JHEP04(2021)064), *Journal of High Energy Physics*, 2021.

<br>
<center><img src="images/paper.png?raw=true" alt="Publication figure"/></center>
<br>

---

<center><sub>Built with Jekyll and the Minimal theme.</sub></center>
