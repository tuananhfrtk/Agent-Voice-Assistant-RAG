# 🏦 Banking Voice Assistant

An intelligent voice-enabled banking assistant that provides natural, conversational responses to customer banking queries. This AI-powered assistant learns from your bank's documentation to deliver accurate, professional, and trustworthy responses through both text and voice interfaces.

## 🌟 Features

- **Natural Voice Interaction**: Professional voice responses using advanced text-to-speech technology
- **Banking-Specific Knowledge**: Learns from your bank's documentation to provide accurate information
- **Multi-Modal Responses**: Delivers both text and voice responses for better accessibility
- **Professional Tone**: Maintains a trustworthy, banking-appropriate communication style
- **Source Attribution**: Always cites sources from official banking documentation
- **Customizable Voice**: Choose from professional voice options suitable for banking
- **Secure Setup**: Protected API key management and secure data handling

## 🎯 Use Cases

- Customer service automation
- Banking product information
- Account type explanations
- Banking procedure guidance
- Policy and regulation clarification
- Interest rate and fee information
- Banking process walkthroughs

## 🚀 Quick Start

1. Clone the repository:
```bash
git clone https://github.com/your-org/banking-voice-assistant.git
cd banking-voice-assistant
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Create a `.env` file in the root directory with your API keys:
```env
OPENAI_API_KEY=your_openai_api_key
QDRANT_URL=your_qdrant_url
QDRANT_API_KEY=your_qdrant_api_key
FIRECRAWL_API_KEY=your_firecrawl_api_key
```

4. Run the application:
```bash
streamlit run banking_voice_assistant.py
```

## ⚙️ Configuration

The Banking Voice Assistant requires the following API keys and configurations:

1. **OpenAI API Key**: For natural language processing and voice generation
2. **Qdrant Vector Database**: For storing and retrieving banking documentation embeddings
3. **Firecrawl API Key**: For crawling and processing banking documentation
4. **Banking Documentation URL**: The URL containing your bank's documentation

### Voice Options

The assistant offers several professional voice options suitable for banking:
- Nova (Default): Professional, clear, and trustworthy
- Onyx: Deep, authoritative, and confident
- Echo: Warm, friendly, and approachable
- Fable: Clear, articulate, and professional
- Shimmer: Bright, engaging, and professional

## 💻 Usage

1. **Initial Setup**:
   - Launch the application
   - Navigate to the sidebar
   - Enter all required API keys
   - Provide your banking documentation URL
   - Select your preferred voice
   - Click "Initialize Banking Assistant"

2. **Asking Questions**:
   - Type your banking-related question in the input field
   - Wait for the assistant to process your query
   - Receive both text and voice responses
   - Download voice responses if needed
   - View source documentation for verification

3. **Example Queries**:
   - "How do I open a savings account?"
   - "What are the current interest rates for home loans?"
   - "Can you explain the wire transfer process?"
   - "What are the fees for international transactions?"
   - "How do I set up online banking?"

## 🔒 Security Considerations

- All API keys are stored securely in the session state
- Documentation processing is done locally
- No sensitive banking data is stored
- All communications are encrypted
- Regular security audits recommended

## 🛠️ Technical Requirements

- Python 3.8+
- Streamlit
- OpenAI API access
- Qdrant Vector Database
- Firecrawl API access
- Internet connection for API calls



## 🤝 Contributing

We welcome contributions to improve the Banking Voice Assistant! Please follow these steps:

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request





