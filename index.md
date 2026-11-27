---
layout: default
description: "Portfolio of Ishan Srivastava, software engineer building production agentic AI systems and data pipelines. Experience, education and selected projects."
---

# Portfolio

Experience, education and selected projects. The project write-ups follow one structure: the **problem and motivation**, **how it was solved**, and a short **summary** of the result and stack.

| Section | Contents |
| --- | --- |
| [Experience](#experience) | Suki.ai · Intellithink Industrial IoT Labs |
| [Education](#education) | Durham University: MSc and MSci |
| [Skills](#skills) | Languages, databases, cloud, AI and agents |
| [Applied AI and Software Engineering](#applied-ai-and-software-engineering) | [AI CFO](#ai-cfo-a-private-local-first-personal-finance-ledger-with-an-ai-assistant) |
| [Machine Learning](#machine-learning) | [Predictive maintenance](#predictive-maintenance-using-machine-learning-based-anomaly-detection) |
| [Computer Vision and Generative Models](#computer-vision-and-generative-models) | [Diffusion SDE](#diffusion-sde-a-score-based-generative-modelling-package) · [Paired translation](#paired-image-to-image-translation-using-diffusion-models) · [CycleGAN](#cyclegans-for-medical-image-translation) · [Unpaired translation](#unpaired-image-to-image-translation-masters-project) |
| [Publications](#publications) | Journal of High Energy Physics |

---

## Experience

### Suki.ai: Software Engineer II
*Bengaluru · Aug 2025 to present*

- Built Suki.ai's first production agentic service for patient-chart and general medical knowledge queries (Google ADK, PostgreSQL, Redis, Langfuse), taking it from prototype to 10K requests per day with P50 latency under 20 seconds through targeted caching.
- Reworked retrieval for temporal queries (up to 50% faster) and added handling for negation and other query types the system previously could not answer, lifting accuracy on those cases by up to 40%.
- Integrated PubMed's open APIs through tool calling to tighten citation integrity on medical responses.
- Set up the agent's evaluation foundation from scratch: gold datasets, evaluation pipelines and Langfuse-based observability, so regressions are caught in the data rather than by a customer.
- Designed a patient-document ingestion pipeline moving terabyte-scale clinical data from EMR systems into GCP FHIR Stores, generating embeddings stored in PostgreSQL with pgvector to power RAG-based semantic and lexical retrieval.
- Built an internal LLM Manager/Router package used across engineering pods for standardized model routing, token-aware load balancing and dynamic provider fallback, letting SREs reroute traffic during GCP Vertex AI outages without touching Kubernetes pods or backend code.
- Introduced a spec-driven development workflow adopted by other pods; teams using it delivered features close to 2x faster.

### Intellithink Industrial IoT Labs: Backend Software Engineer
*Bengaluru · Sep 2022 to Jul 2025*

- Redesigned IoT data ingestion around AWS Kinesis and Lambda batching, moving toward a more serverless setup and cutting cloud spend by 25%.
- Added Parquet compression and S3 lifecycle policies that reduced storage costs by 20% without affecting query performance.
- Reworked the DynamoDB schema with local and global secondary indexes so dashboard queries resolve in a single call, bringing data-heavy dashboard page loads under 5 seconds.
- Added a Redis caching layer in front of high-frequency API calls, reducing database load by 20% and improving response times by 10%.
- Owned deployment of a real-time ML inference pipeline on SageMaker, wired into Lambda, SQS and DynamoDB; moving it to Dockerized serverless execution saved 25% on cost.
- Tuned request handling with FastAPI, Nginx and Gunicorn with Uvicorn to process streaming IoT data concurrently, making inference roughly 2x faster.

---

## Education

- **MSc, Scientific Computing and Data Analysis**, University of Durham (2021 to 2022). First Class Honours.
- **MSci, Theoretical Physics**, University of Durham (2015 to 2019). First Class Honours.

---

## Skills

| | |
| --- | --- |
| **Languages** | Python, Go, SQL |
| **Databases** | PostgreSQL (pgvector), DynamoDB, MySQL, Redis |
| **Cloud and infrastructure** | GCP (Vertex AI, Kubernetes, Helm, Grafana, FHIR Store), AWS (Lambda, S3, Kinesis, SageMaker), Docker, Git, Linux |
| **AI and agents** | Google ADK, LLM tool calling, RAG, evaluation and observability (Langfuse), agent routing and fallback design |

---

## Applied AI and Software Engineering

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

## Computer Vision and Generative Models

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
[![Thesis report](https://img.shields.io/badge/Thesis-Read_the_report_(PDF)-informational)](pdf/ProjectReport.pdf)

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
[![Thesis report](https://img.shields.io/badge/Thesis-Read_the_report_(PDF)-informational)](pdf/ProjectReport.pdf)

**Problem and motivation.**
Paired training data is often unavailable, so unpaired translation, where no image in one domain has a matching partner in the other, is the more general and harder problem. CycleGAN is the standard approach. My Master's thesis (supervised by Dr Chris G Willcocks, Durham University) asked whether score-based generative models could be adapted to the unpaired setting.

**How it was solved.**

1. I extended the score-based SDE framework from the paired project to unpaired translation between two image domains.
2. The model was trained on the horse2zebra benchmark and compared directly with CycleGAN outputs on the same inputs.
3. The full thesis report, covering both the paired and unpaired experiments, is linked above and the code is in the repository.

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
