# 🛡️ ContractGuardAI

> AI-powered contract protection system that automatically monitors Box storage, analyzes contracts using AWS Bedrock & Box AI, and generates protected mirror contracts with comparison documents and negotiation guides.

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![AWS](https://img.shields.io/badge/AWS-Bedrock%20%7C%20SNS-orange.svg)](https://aws.amazon.com/)
[![Box](https://img.shields.io/badge/Box-API-blue.svg)](https://developer.box.com/)

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [AWS Deployment](#-aws-deployment)
- [How It Works](#-how-it-works)
- [Examples](#-examples)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

## 🎯 Overview

ContractGuardAI is an intelligent contract analysis system that helps you protect your interests in contracts. Simply upload contracts to Box, and the system automatically:

- **Monitors** your Box storage for new contracts
- **Analyzes** contracts against your personal interests and preferences
- **Generates** protected "mirror" contracts with your interests in mind
- **Creates** comparison documents showing all changes
- **Provides** negotiation guides with talking points
- **Detects** time-sensitive action items and sends notifications

Perfect for employment contracts, service agreements, lease agreements, and more!

## ✨ Features

### 🤖 AI-Powered Analysis
- Uses **AWS Bedrock** (Claude) and **Box AI** for intelligent contract analysis
- Understands natural language preferences
- Context-aware contract rewriting

### 📁 Automated Monitoring
- Continuous monitoring of Box folders
- Automatic detection of new contracts
- Background processing with configurable intervals

### 📄 Multi-Format Output
For each contract, generates three comprehensive documents:
1. **Mirror Contract** - Protected version with your interests
2. **Redline Comparison** - Visual PDF showing all changes
3. **Negotiation Guide** - Step-by-step guide with talking points

### 🔔 Action Item Detection
- Identifies time-sensitive deadlines
- Sends email notifications via AWS SNS
- Tracks important dates and obligations

### 🎯 Flexible Configuration
- **General Interests File** - Set preferences once, apply to all contracts
- **Per-Contract Instructions** - Override preferences for specific contracts
- **Non-Negotiables** - Protect critical terms that must stay unchanged

### ☁️ Cloud-Native
- Fully integrated with Box storage
- AWS infrastructure (Bedrock, SNS, Secrets Manager)
- Scalable EC2 deployment option

## 🏗️ Architecture

```
┌─────────────┐
│   Box API   │ ← Contract Storage & AI Analysis
└──────┬──────┘
       │
       ▼
┌──────────────────────┐
│  Contract Processor  │ ← Monitors & Processes Contracts
│  ┌────────────────┐ │
│  │ Box Service    │ │
│  │ Bedrock AI     │ │
│  │ Action Detector│ │
│  └────────────────┘ │
└──────────┬───────────┘
           │
           ▼
┌─────────────┐
│   AWS SNS   │ ← Email Notifications
└─────────────┘
```

## 🛠️ Tech Stack

- **Python 3.8+** - Core language
- **Box SDK** - Box API integration
- **AWS Bedrock** - AI contract analysis (Claude)
- **AWS SNS** - Email notifications
- **Box AI** - Document analysis and generation
- **python-docx** - Word document generation
- **reportlab** - PDF generation
- **boto3** - AWS services integration

## 📦 Installation

### Prerequisites

- Python 3.8 or higher
- Box Developer Account ([Get one here](https://developer.box.com))
- AWS Account (for Bedrock and SNS)
- Git

### Step 1: Clone the Repository

```bash
git clone https://github.com/Sameersah/ContractGuardAI.git
cd ContractGuardAI
```

### Step 2: Install Dependencies

```bash
# Using pip
pip install -r requirements.txt

# Or using uv (recommended)
uv pip install -r requirements.txt
```

### Step 3: Set Up Box MCP Server

The project includes the Box MCP server as a submodule. Set it up:

```bash
cd mcp-server-box
uv sync  # or: pip install -r requirements.txt
```

### Step 4: Configure Environment Variables

Create a `.env` file in the `mcp-server-box/` directory:

```bash
cd mcp-server-box
cp .env.example .env
nano .env  # or use your preferred editor
```

Add your credentials:

```ini
# Box OAuth Credentials
BOX_CLIENT_ID=your_client_id_here
BOX_CLIENT_SECRET=your_client_secret_here
BOX_REDIRECT_URL=http://localhost:8000/callback

# AWS Credentials (for Bedrock and SNS)
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_DEFAULT_REGION=us-east-1
AWS_SNS_TOPIC_ARN=arn:aws:sns:region:account:topic-name
USER_EMAIL=your-email@example.com

# Optional: MCP Server Auth Token
BOX_MCP_SERVER_AUTH_TOKEN=your_secure_token_here
```

### Step 5: Authorize Box App

First-time authorization:

```bash
python -c "from box_ai_agents_toolkit import authorize_app; authorize_app()"
```

Follow the prompts to authorize the application in your browser.

## ⚙️ Configuration

### 1. Create Your Interests File

Create `MY_INTERESTS.txt` in Box folder `my_interests/`:

```
MY CONTRACT INTERESTS & PREFERENCES
====================================

GENERAL PRINCIPLES:
- I prefer shorter contract terms (1-2 years max) with renewal options
- I want clear termination clauses that favor me
- I need flexibility to exit contracts with reasonable notice

EMPLOYMENT CONTRACTS:
- Non-compete clauses: Max 6 months, specific to role
- IP Ownership: Company owns work on company time; I own personal projects
- Severance: At least 2 weeks per year of service
- PTO: 20+ days or unlimited

SERVICE AGREEMENTS:
- Payment terms: Net 15-30, not Net 60+
- Termination: Early exit with prorated refunds
- Liability: Reasonable caps, not unlimited

LEASE AGREEMENTS:
- Security deposit: Max 1 month's rent
- Early termination: 30 days notice option
- Pet policy: Reasonable, not blanket bans
```

### 2. Set Up Box Folder Structure

The system automatically creates these folders in your Box account:

```
Box Root/
├── Smart_Contracts/              # Upload contracts here
│   ├── Employment_Offer.pdf
│   └── Employment_Offer.pdf.instructions  (optional)
│
├── protect_your_interests/       # Generated outputs
│   └── Employment_Offer_mirror/
│       ├── 1_mirror_contract_protecting_YOUR_interests.docx
│       ├── 2_clean_redline_comparison.pdf
│       └── 3_negotiation_guide.docx
│
└── my_interests/                 # Your preferences
    └── MY_INTERESTS.txt
```

### 3. Per-Contract Instructions (Optional)

For specific contracts, create `{contract_name}.instructions` in the `Smart_Contracts/` folder:

```
CRITICAL ISSUES:
1. Non-compete clause (Section 8.3)
   - Current: "12 months, any industry, anywhere in the US"
   - Need: "6 months, software development only, within 50 miles"
   - Why: Too broad, limits career options unfairly

NON-NEGOTIABLES:
- Salary: $120,000 (must stay)
- Start date: January 15, 2025 (must stay)
```

## 🚀 Usage

### Run the Contract Processor

```bash
python contract_processor.py
```

The system will:
1. ✅ Create necessary folders in Box (if they don't exist)
2. ✅ Monitor the `Smart_Contracts/` folder for new files
3. ✅ Process each contract and generate 3 output files
4. ✅ Save results to `protect_your_interests/{contract_name}_mirror/`

### Workflow

1. **Upload Contract**: Drop a contract (PDF/DOCX) in `Smart_Contracts/` folder in Box
2. **System Detects**: Automatically detects new files (checks every 60 seconds)
3. **Loads Interests**: Reads `my_interests/MY_INTERESTS.txt` from Box
4. **Checks Instructions**: Looks for `{contract_name}.instructions` file (optional)
5. **Analyzes**: Uses Box AI and AWS Bedrock to analyze contract against your interests
6. **Generates**: Creates 3 output files using AI
7. **Saves**: Uploads results to `protect_your_interests/{contract_name}_mirror/`
8. **Notifies**: Sends email if action items are detected

### Output Files

For each processed contract, you'll receive:

1. **`1_mirror_contract_protecting_YOUR_interests.docx`**
   - Rewritten contract with your interests protected
   - Same structure as original
   - Key terms modified to favor you
   - Non-negotiables kept intact

2. **`2_clean_redline_comparison.pdf`**
   - Visual comparison showing all changes
   - Original text (strikethrough)
   - New text (highlighted)
   - Easy-to-read format

3. **`3_negotiation_guide.docx`**
   - Summary of all changes
   - Why each change protects your interests
   - Talking points for negotiations
   - Negotiation priority order

## ☁️ AWS Deployment

For production deployment on AWS EC2, see the [deployment guide](deploy/DEPLOYMENT_GUIDE.md).

### Quick Deploy

```bash
cd deploy/
cp secrets.json.example secrets.json
# Edit secrets.json with your credentials
./quick-deploy.sh
```

### Architecture

- **EC2 Instance** - Runs the contract processor
- **AWS Secrets Manager** - Stores credentials securely
- **AWS SNS** - Sends email notifications
- **IAM Roles** - Secure access without hardcoded keys

### Cost Estimate

- EC2 t3.micro: ~$7-10/month
- Secrets Manager: ~$0.40/month
- SNS: Free tier
- **Total: ~$8-11/month**

## 🔄 How It Works

### Processing Pipeline

```
1. Monitor Box Folder
   ↓
2. Detect New Contract
   ↓
3. Load User Interests
   ↓
4. Check Per-Contract Instructions (if exists)
   ↓
5. Analyze Contract with AI
   ├─→ Box AI: Document understanding
   └─→ AWS Bedrock: Contract analysis
   ↓
6. Generate Protected Contract
   ↓
7. Create Comparison Document
   ↓
8. Generate Negotiation Guide
   ↓
9. Detect Action Items
   ↓
10. Upload Results to Box
   ↓
11. Send Notifications (if action items found)
```

### AI Analysis Process

1. **Document Understanding**: Box AI extracts contract structure and content
2. **Interest Matching**: Compares contract terms against your preferences
3. **Term Modification**: Rewrites clauses to protect your interests
4. **Validation**: Ensures non-negotiables remain unchanged
5. **Documentation**: Creates comparison and negotiation materials

## 📝 Examples

### Example 1: Employment Contract

**Input**: `Employment_Offer_TechCorp.pdf`

**Your Interests** (from `MY_INTERESTS.txt`):
- Non-competes: Max 6 months
- IP: Personal projects remain mine
- Severance: 2 weeks per year

**Output**:
- Mirror contract with 6-month non-compete (instead of 12)
- IP clause protecting personal projects
- Improved severance terms
- Comparison PDF showing all changes
- Negotiation guide with talking points

### Example 2: Service Agreement

**Input**: `Service_Agreement_XYZ.pdf` + `Service_Agreement_XYZ.pdf.instructions`

**Per-Contract Instructions**:
```
Payment: Change from Net 60 to Net 30
Termination: Add 30-day cancellation option
```

**Output**:
- Modified payment terms
- Added termination clause
- All other terms adjusted per general interests

## 🐛 Troubleshooting

### Authentication Errors

```bash
# Re-authorize Box app
python -c "from box_ai_agents_toolkit import authorize_app; authorize_app()"
```

### File Not Found

- Ensure folders exist in Box: `Smart_Contracts/`, `protect_your_interests/`, `my_interests/`
- Check file names match exactly (case-sensitive)

### AI Errors

- Verify Box AI is enabled for your account
- Check AWS Bedrock access and region configuration
- Ensure sufficient API quotas

### Import Errors

```bash
# Reinstall dependencies
pip install -r requirements.txt
```

### AWS SNS Notifications Not Working

- Verify `AWS_SNS_TOPIC_ARN` is set correctly
- Check SNS topic exists and has email subscription
- Verify AWS credentials have SNS permissions

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🔗 Related Documentation

- [Contract Protection System Guide](CONTRACT_PROTECTION_SYSTEM.md)
- [Implementation Details](README_CONTRACT_SYSTEM.md)
- [AWS Deployment Guide](deploy/DEPLOYMENT_GUIDE.md)
- [Per-Contract Instructions](PER_CONTRACT_INSTRUCTIONS_GUIDE.md)

## 📧 Support

For issues, questions, or contributions:
- Open an [Issue](https://github.com/Sameersah/ContractGuardAI/issues)
- Check existing documentation in the `documents/` folder

## 🙏 Acknowledgments

- [Box AI Agents Toolkit](https://github.com/box-community/box-ai-agents-toolkit)
- [Box MCP Server](https://github.com/box-community/mcp-server-box)
- AWS Bedrock for AI capabilities

---

**Made with ❤️ for protecting your contract interests**
