# 🤖 AI Chat & Collaboration Solution Project Plan (Enhanced Version)

**Vert.x MSA + React + React Native + AI Integrated Platform**

## ⚠️ Important Notice

**This project strictly prohibits commercial use, redistribution, and reuse.**

* 🚫 **Commercial Use Prohibited**: Cannot be used for any form of commercial purposes
* 🚫 **Redistribution Prohibited**: Cannot be distributed or shared with third parties
* 🚫 **Reuse Prohibited**: Code or design cannot be utilized in other projects
* ✅ **Usage Conditions**: Can only be used after obtaining prior approval from the creator (richardaiagent) and paying legitimate fees

**Contact**: richardaiagent@github.com

## 📄 License

**Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License**

**R.C-MSA-VrR-Tutorial © 2015 by richardaiagent** is licensed under Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License.

**Attribution Information:**
* Title of work: R.C-MSA-VrR-Tutorial
* Creator: richardaiagent
* Source: https://github.com/richardaiagent/R.C-MSA-VrR-Tutorial
* License: https://creativecommons.org/licenses/by-nc-sa/4.0/

**You are free to:**
* Share — copy and redistribute the material in any medium or format
* Adapt — remix, transform, and build upon the material

**Under the following terms:**
* **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made. You may do so in any reasonable manner, but not in any way that suggests the licensor endorses you or your use.
* **NonCommercial** — You may not use the material for commercial purposes
* **ShareAlike** — If you remix, transform, or build upon the material, you must distribute your contributions under the same license as the original

**No additional restrictions** — You may not apply legal terms or technological measures that legally restrict others from doing anything the license permits.

**Full Legal Text**: https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode

## 📋 Project Overview

### 🎯 Objectives

* Development and service launch of next-generation AI-based chat & collaboration solution on-premises
* Windows 11 + Docker Desktop + VS Code environment
* Vert.x-based MSA + React Web + React Native Mobile
* Building in-house AI models with Ollama for unlimited token usage
* **Primary Goal**: Support simple HR tasks and unit screen development coding tasks
* Real-time collaboration and personalized AI assistant services

### 🎯 Phase 1 MVP Goals (Detailed)

**HR Task AI Support**
* Employee information inquiry and management
* Automatic creation of leave/business trip applications
* Work schedule optimization suggestions
* Meeting room reservation and scheduling

**Development Coding Task Support**
* React/React Native component code generation
* Automatic unit screen template generation
* Code review and optimization suggestions
* API integration code snippet provision

## 🏗️ Technology Stack

| Category | Technology | Version | Purpose |
|----------|------------|---------|---------|
| Backend | Java + Eclipse Vert.x | 11 + 4.x | MSA Backend |
| Web Frontend | React + TypeScript | 18 + 5.x | Web Client |
| PC Desktop | Electron + React | Latest + 18 | Windows/Mac/Linux |
| Mobile | React Native + TypeScript | 0.72+ | iOS/Android App |
| Database | MySQL + Oracle + Redis | 8.0 + 12c + 7.x | Multi-DB Support |
| AI Integration | Ollama + Custom Models | Llama2/CodeLlama/Mistral | On-Premises AI |
| Vector DB | Redis Vector Search | 7.2+ | Embedding Storage |
| Container | Docker + Docker Compose | Latest | Containerization |
| Proxy | Nginx | Latest | Reverse Proxy |
| Monitoring | Prometheus + Grafana | Latest | Monitoring |

## 🤖 AI Model Detailed Specifications

### Ollama AI Service Configuration

| Model Name | Size | Memory Usage | Main Purpose | Performance Target |
|------------|------|--------------|--------------|-------------------|
| Llama2-7B-Chat | 7B | ~4GB RAM | General conversation, HR task Q&A | Response time < 3s |
| CodeLlama-7B | 7B | ~4GB RAM | Code generation, review, debugging | Code generation < 5s |
| Mistral-7B-Instruct | 7B | ~4GB RAM | Business document writing, summarization | Text generation < 4s |

### AI Service Detailed Features

**HR Task AI Support:**
* Natural language-based employee information search
* Automatic completion of leave application templates
* Meeting schedule conflict detection and alternative suggestions
* Work priority analysis and recommendations

**Development Coding AI Support:**
* React/TypeScript component scaffolding
* Automatic REST API client code generation
* Screen layout CSS/Tailwind suggestions
* Code quality analysis and improvement suggestions

### Redis Vector Search Utilization

```yaml
Vector DB Configuration:
  - Model: sentence-transformers/all-MiniLM-L6-v2
  - Dimensions: 384
  - Similarity: Cosine Similarity
  - Index: HNSW (Hierarchical Navigable Small World)
  
Stored Data:
  - Company policy document embeddings
  - Code snippet library
  - FAQ and work manuals
  - Development guidelines
```